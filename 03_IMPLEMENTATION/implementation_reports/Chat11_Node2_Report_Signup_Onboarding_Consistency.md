# Chat11 — Node 2 Investigation: Signup Onboarding Consistency

## A. Preflight
- **Source repository root:** `freight`
- **Branch:** `main`
- **Git status:** Up to date with origin/main. 4 modified tracked files (`auth/login/route.ts`, `auth/signup/route.ts`, `login/page.tsx`, `signup/page.tsx`), 1 untracked file (`src/db/migrations/004_auto_generate_driver_code.sql`).
- **Relevant state:** All signup/authentication logic changes are currently **LOCAL ONLY** and have not been committed or pushed to the `freight` GitHub repository. The `Freight_Records` repository is on `main` and up to date.

## B. Current Signup Flow
The exact sequence for a signup request (`src/app/api/auth/signup/route.ts`):
1. **Entry:** `POST /api/auth/signup` receives `email` and `password`.
2. **Auth User Creation:** Calls `await createClient().auth.signUp({ email, password })`.
3. **Application Identity Creation:** If Auth succeeds, calls `await supabaseServer.from('drivers').insert({ auth_id: authData.user.id, name: email.split('@')[0] }).select('driver_code').single()`. (Note: `supabaseServer` uses the `service_role` key).
4. **Completion:** Returns `{ success: true, driver_code }` to the client.

## C. Transaction Boundaries
- **Not Atomic:** The creation of the Auth User (via Supabase Auth API) and the Application Identity (via Supabase PostgREST API) are two entirely separate network requests.
- **Transaction:** There is no database transaction wrapping these two operations. If the server crashes, network fails, or database insert fails between step 2 and 3, the operations are partially completed.

## D. Failure-State Matrix

| Scenario | Auth User | Application Identity | Evidence Status |
|---|---|---|---|
| Driver `insert` fails (e.g., timeout) | EXISTS | MISSING | VERIFIED (API returns 500 "Account created but failed to initialize driver profile" and leaves Auth User orphaned) |
| Auth User is deleted later by admin | MISSING | EXISTS | VERIFIED (Migration `003` defines `auth_id uuid UNIQUE REFERENCES auth.users(id) ON DELETE SET NULL`, leaving an orphaned Driver record if the Auth User is deleted) |

## E. Retry / Duplicate Behavior
- **Non-Idempotent Retries:** If a user gets orphaned (Auth User exists, Driver missing) and tries to sign up again, the `auth.signUp` call will fail (e.g., "User already registered"). The retry will never reach the `drivers` insert.
- **Resumption Impossible:** A partially-created account cannot be safely resumed through the current signup UI/API flow. 
- **Collisions:** Driver Code collisions are prevented by the database trigger generation and `UNIQUE` constraint, but `auth_id` uniqueness protects against one user creating multiple Drivers.

## F. Database Enforcement
- **`drivers.auth_id`:** `UNIQUE REFERENCES auth.users(id) ON DELETE SET NULL` (from committed migration `003_add_auth_id_to_drivers.sql`).
- **`driver_code`:** `UNIQUE NOT NULL` (from committed migration `001_create_core_tables.sql`).
- **Triggers:** A `BEFORE INSERT` trigger generates the `driver_code` (from local-only migration `004_auto_generate_driver_code.sql`). No triggers exist on `auth.users` for identity creation.

## G. Email Confirmation Interaction
- **Observed Behavior:** Identity (Driver record) creation occurs immediately after the `signUp` call returns, regardless of email confirmation status.
- **Unconfirmed Users:** The Driver record is created synchronously even if the Supabase project requires email confirmation. The code does not check the confirmation state before inserting into the `drivers` table.

## H. Orphan / Recovery Mechanisms
- **Current Mechanisms:** None.
- **Absence:** There is no background job, no retry queue, no administrative reconciliation panel, and no webhook/trigger to clean up orphaned Auth Users or retry failed Driver creations. 

## I. Architecture Constraints
- **API Route:** Signup is currently orchestrated sequentially in a Next.js Server API route.
- **Service Role:** The application already utilizes the `service_role` key in `supabaseServer` to perform admin-level database bypasses for Driver creation.
- **Missing Database Hooks:** There are currently no Postgres triggers hooked into the `auth.users` table.

## J. Decision Inputs
1. **Current failure mode:** The dual-API call approach demonstrably leaves orphaned Auth Users if the subsequent database insert fails.
2. **Current retry mode:** An orphaned user is permanently stuck; they cannot log in (no driver code) and they cannot sign up again (email taken).
3. **Current schema behavior:** Deleting an Auth User nullifies `auth_id` but leaves the Driver record orphaned in the database.
4. **Current architectural capabilities:** The Next.js API route has `service_role` access, but lacks transactional safety with Supabase Auth.

## K. Remaining UNKNOWNs
- All requested facts were successfully established from the current codebase and migrations.
