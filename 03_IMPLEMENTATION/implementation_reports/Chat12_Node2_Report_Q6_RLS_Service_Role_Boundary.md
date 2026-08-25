# Chat12 Node 2 — Q6 Investigation: RLS / Service-Role Boundary

## 1. Executive Conclusion
The recommended policy for Q6 is a **Strict Demilitarized Zone (DMZ) Pattern**. 
1. **Direct Database Access:** All frontend Supabase clients MUST run under Row Level Security (RLS) bound to `auth.uid()`. 
2. **API Routes:** All user-facing API routes in Next.js MUST use a user-scoped Supabase client that propagates the user's session (and thus RLS), rather than escalating to `service_role`. 
3. **Service Role Use:** The `service_role` key MUST be restricted strictly to elevated administrative tasks (e.g., identity verification webhooks, cleanup jobs) that inherently require RLS bypass. It is never an authorization replacement.

## 2. Current Architecture / Evidence
- **Prior RLS Experiments:** Chat 8 confirmed that RLS based on `auth_id = auth.uid()` correctly filters events/trips to the logged-in driver.
- **Service Role Bleed:** Earlier APIs (e.g., `src/app/api/auth/signup/route.ts`) used `supabaseServer` initialized with the `SUPABASE_SERVICE_ROLE_KEY` to insert driver records, effectively bypassing RLS.
- **Next.js Client Creation:** `src/lib/supabase/server.ts` creates user-scoped clients (which respect RLS), while `src/lib/supabase-server.ts` exports a dangerous global service-role client.

## 3. Threat Model
- **IDOR (Insecure Direct Object Reference):** A malicious driver intercepts network requests and changes `driver_id` to steal/view another driver's trips.
- **Privilege Escalation:** A driver modifies their own `verification_status` to `VERIFIED`.
- **Service-Role Abuse:** An API route intended for users accidentally uses the `service_role` client, allowing users to bypass RLS by passing arbitrary IDs in the request body.
- **Missing RLS:** A new table is created without `ALTER TABLE ENABLE ROW LEVEL SECURITY`, defaulting to wide-open public access.

## 4. RLS Responsibility
RLS is the **foundational data boundary**. It guarantees that regardless of UI bugs, API flaws, or Node 1 gaps, a user cannot read, update, or delete rows belonging to another user.
- **Enforcement:** Every business table (`drivers`, `trips`, `events`, `freight_identities`) MUST have RLS enabled.
- **Identity mapping:** Policies must bind to `auth.uid()`, e.g., `WHERE auth_id = auth.uid()`.

## 5. Service-Role Responsibility
The `service_role` key operates as the PostgreSQL `postgres` superuser, bypassing RLS completely.
- **Valid Uses:** Administrative approvals, OCR webhook data ingestion, orphan account cleanup, background CRON jobs.
- **Invalid Uses:** Standard user API routes, Next.js Server Actions acting on behalf of a user.

## 6. Access Matrix

| Operation | User Client (Browser) | User Client (Next.js API) | Service Role Client |
|---|---|---|---|
| View own trips | ALLOWED (via RLS) | ALLOWED (via RLS) | DO NOT USE |
| View others' trips | BLOCKED (by RLS) | BLOCKED (by RLS) | BYPASSES RLS |
| Modify own verification state| BLOCKED (by RLS) | BLOCKED (by RLS) | ALLOWED |
| Create generic identity | BLOCKED (Auth Trigger) | BLOCKED (Auth Trigger)| N/A (Trigger handles) |
| Log an event (Node 1 allowed) | ALLOWED (via RLS) | ALLOWED (via RLS) | DO NOT USE |

## 7. Node 1 Authorization Interaction
RLS and Node 1 are complementary, not replacements for each other.
- **RLS:** Limits the query strictly to John's data. (John cannot fetch Jane's data).
- **Node 1 Authorization:** Examines John's state to enforce business rules. (John cannot start Trip B because John is already active on Trip A).
- **Enforcement:** RLS cannot easily evaluate complex cross-table state machines (Node 1) without severe performance penalties. Node 1 business logic lives in the application code, backed by RLS data isolation.

## 8. Identity / Verification Interaction
- **Verification Status:** A column like `verification_status` in `freight_identities` must have an RLS `UPDATE` policy that returns `false` (meaning no authenticated user can update it).
- **Trusted Elevation:** Only the `service_role` client (in a protected admin route or webhook) can update the `verification_status` or assign the `trusted_role`, because it bypasses the `false` RLS policy.

## 9. IDOR / Cross-User Attack Analysis
If an API route uses `service_role` and blindly trusts `req.body.driver_id`, an attacker can send someone else's ID and act on their behalf. 
By forcing all user-facing API routes to use a **user-scoped Supabase client**, the database automatically filters operations to `auth.uid()`. If the attacker sends a mismatched `driver_id`, the RLS policy evaluates `auth.uid() == row.auth_id` and rejects the operation natively.

## 10. Recommended MVP Policy
1. **Enable RLS Everywhere:** Run `ALTER TABLE x ENABLE ROW LEVEL SECURITY;` on all tables.
2. **User-Scoped Server Clients:** Next.js Server Components and API Routes MUST use the user-scoped `@supabase/ssr` client (which forwards the cookie/token to PostgREST).
3. **Quarantine Service Role:** Rename `supabaseServer` to `supabaseAdminClient` to explicitly flag its danger, and forbid its use in normal user API paths.
4. **Auth Trigger Overrides:** Identity creation runs in a database trigger, which natively runs with elevated privileges and circumvents the need for Next.js to use `service_role` for signups.

## 11. Rejected Alternatives
- **Application-Layer-Only Security:** (Disabling RLS and trusting Next.js to check IDs). Rejected. Prone to IDOR vulnerabilities when developers forget a check.
- **Service-Role APIs:** (Passing all API requests through `service_role`). Rejected. Subverts the Supabase architecture and recreates the exact IDOR risks RLS was built to solve.

## 12. Acceptance-Test Matrix
- **Test:** User A attempts to `SELECT` User B's trips via standard client. **Result:** Empty array (RLS enforced).
- **Test:** User A attempts to `UPDATE` their own `verification_status`. **Result:** Unauthorized / RLS violation.
- **Test:** Admin webhook (using service_role) updates User A's `verification_status`. **Result:** Success (RLS bypassed).
- **Test:** API Route fetches data using `service_role` for a user request. **Result:** CI/CD or Code Review Rejection (Policy Violation).

## 13. Remaining Unknowns
- None.

## 14. Proposed Q6 Decision
`Q6 = Strict RLS enforcement for all user operations; Service Role quarantined to admin/webhook operations only.`

---
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🔒 LOCKED
Q6 = 🟡 INVESTIGATED / NOT LOCKED
Implementation = ⏸️ PAUSED
