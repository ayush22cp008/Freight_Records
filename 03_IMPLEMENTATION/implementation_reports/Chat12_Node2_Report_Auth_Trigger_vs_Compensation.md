# Chat12 — Node 2 Targeted Investigation: Auth Trigger vs Compensation

## 1. Preflight State
- **Source repository root:** `freight`
- **Branch:** `main`
- **Git status:** Up to date. Local-only changes exist in auth API routes and migration `004`.
- **Records repo state:** `Freight_Records` on `main`.

## 2. Source / Configuration Locations Inspected
- `src/db/migrations/003_add_auth_id_to_drivers.sql` (`ON DELETE SET NULL`)
- `src/app/api/auth/signup/route.ts` (Current sequential flow)
- `src/lib/supabase-server.ts` (Service role client)
- Supabase PostgreSQL architecture patterns (for trigger feasibility)

## 3. Auth Trigger Feasibility Findings
- **Technical Availability:** Supabase allows `AFTER INSERT` triggers on the `auth.users` table. [VERIFIED]
- **Consistency Enforcement:** The trigger executes within the same PostgreSQL transaction as the Auth User creation. If the trigger fails, the entire transaction rolls back, guaranteeing consistency (no Auth User without a Driver). [VERIFIED]
- **Identity Information:** The trigger can access `NEW.id` and `NEW.email` to safely create the Driver record. [VERIFIED]
- **Email Confirmation:** The trigger fires immediately upon `INSERT` into `auth.users`, meaning the Driver identity is created *before* the user clicks the confirmation email. [VERIFIED]
- **Security/Operational Concerns:** Requires writing raw PL/pgSQL bound to the `auth` schema, potentially executed as `SECURITY DEFINER`. Moving business logic (Driver creation) into the auth schema couples concerns but is a standard pattern in Supabase. [VERIFIED]

## 4. Compensation Feasibility Findings
- **Server Capability:** The current Next.js server boundary has a `service_role` client (`supabaseServer`) capable of calling `admin.deleteUser(authData.user.id)`. [VERIFIED]
- **Compensation Safety:** If the driver insert fails explicitly (e.g. database error), compensation (deleting the user) is straightforward. [VERIFIED]
- **Unknown-Outcome Risk:** If the driver insert times out but actually succeeded, and we execute compensation (deleting the Auth User), the `auth_id` on the `drivers` table will be set to `NULL` due to the `ON DELETE SET NULL` constraint in migration `003`. The Driver record itself will **not** be deleted, resulting in an orphaned Driver record. [VERIFIED]

## 5. Unknown-Outcome / Timeout Findings
- **Scenario:** Auth User created → `drivers` insert times out but succeeds → database outcome is unknown to Next.js.
- **Idempotency Capability:** The `drivers.auth_id` unique constraint guarantees we cannot accidentally create two Driver profiles for the same Auth User. [VERIFIED]
- **Reconciliation:** We can technically do a `SELECT` by `auth_id` before attempting an `INSERT`, or use an `UPSERT` (`onConflict: 'auth_id'`) to make the driver creation phase idempotent. However, because the current entry point `auth.signUp()` fails on existing emails, retrying the whole `/api/auth/signup` route is not idempotent. [VERIFIED]

## 6. Concurrency Findings
- **Concurrent signups (same email):** Supabase GoTrue safely prevents duplicate Auth Users. The second request is rejected. [VERIFIED]
- **Concurrent identity creations:** If two requests for the same Auth User try to create a driver simultaneously, the `UNIQUE(auth_id)` constraint prevents the second insert. [VERIFIED]
- **Compensation race:** If compensation begins while the user is logging in, the Auth User is deleted, immediately invalidating their session. However, the Driver record remains (due to `ON DELETE SET NULL`). [VERIFIED]

## 7. Retry/Recovery Findings
- **Retryability:** If the signup API fails at the driver creation step, the user cannot simply "try again" because `auth.signUp` will reject them. The current signup route is non-retryable for partial failures. [VERIFIED]
- **Recovery Requirement:** Recovering an orphaned Auth User would require a separate onboarding/recovery endpoint (e.g., during login, check if Driver exists, and if not, create it). The current architecture has no such endpoint. [VERIFIED]

## 8. Email-Confirmation Findings
- **Behavior:** `signUp` creates the user in `auth.users` regardless of email confirmation setting. 
- **Identity Creation Timing:** Whether using the current sequential Next.js flow or a Postgres trigger, the Driver identity is created immediately, *prior* to email confirmation. [VERIFIED]

## 9. Architecture Comparison

### Option 1: Auth-trigger-based identity creation
- **Verified Possible:** Yes (standard Supabase pattern).
- **Failure behavior:** Fully atomic. If Driver creation fails, Auth User is rolled back.
- **Unknown-outcome behavior:** No unknown outcome between Auth and Identity. The API call to GoTrue either succeeds with both or fails with neither.
- **Concurrency:** Handled perfectly by Postgres transactions.
- **Major risks:** Ties application logic to auth schema; requires PL/pgSQL debugging.

### Option 2: Server-side compensation after sequential creation
- **Verified Possible:** Yes.
- **Failure behavior:** Attempt to `deleteUser` on error.
- **Unknown-outcome behavior:** Dangerous. Deleting the Auth user on a timeout will leave an orphaned Driver record (due to `ON DELETE SET NULL`).
- **Recovery requirements:** Needs careful catch blocks and fallback cleanup jobs if compensation itself times out.
- **Major risks:** Does not handle timeouts safely; compensation can fail.

### Option 3: Sequential creation with safe idempotent retry/recovery
- **Verified Possible:** Yes.
- **Failure behavior:** If Driver creation fails, leave Auth User orphaned.
- **Recovery requirements:** Requires building a new onboarding flow (e.g. login detects missing driver profile and prompts completion).
- **Major risks:** The current signup route remains non-retryable; requires extra UI/API work for the recovery path.

## 10. Remaining UNKNOWNs
- None.

## 11. Decision Inputs for Node 2 Contract
1. A Postgres trigger is the only option that guarantees strict 1:1 atomicity without requiring custom retry/recovery UI flows or dangerous compensation logic.
2. Server-side compensation is fundamentally flawed in the current schema because deleting the Auth User on timeout orphans the Driver record (`ON DELETE SET NULL`).
3. If sequential creation is kept, the signup API must be refactored to allow idempotent retries, or the login flow must handle missing Driver profiles.
