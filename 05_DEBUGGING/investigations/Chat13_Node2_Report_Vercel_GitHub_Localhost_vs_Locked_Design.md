# Chat13 Node 2 Investigation: Vercel vs GitHub vs Localhost vs Locked Design

## 1. Exact GitHub commit inspected
`d8801d9` (Merge made by the 'ort' strategy). This is the current head of `origin/main`.

## 2. Exact Vercel deployment commit
**UNKNOWN**. Vercel deploy commit cannot be strictly verified from this environment, but it is INFERRED to be tracking `origin/main` (commit `d8801d9` or slightly prior).

## 3. Exact local commit/status
Local is on `main` tracking `origin/main`. The working tree is clean except for the Node 2 investigation reports just added. However, earlier analysis revealed that the local environment *had* uncommitted changes for auto-generating Driver Codes, which we evaluated and decided to discard (`git reset --hard`) in favor of the cleaner GitHub baseline for Node 2.

## 4. Screenshot / UI differences
- **Vercel / GitHub:** Signup page asks the user to manually type in a `Driver Code`. Login page asks for `Email` and Password.
- **Localhost (prior uncommitted state):** Signup page hides `Driver Code`, auto-generates it on success, and shows it to the user. Login page asks for `Driver ID` (Driver Code) and Password.

## 5. GitHub vs Local source differences
- **Local implementation:** Introduced a new DB migration (`004_auto_generate_driver_code.sql`) and modified Next.js API routes (`api/auth/signup`, `api/auth/login`) to proxy login via Driver Code.
- **Classification:** These were *Local-only implementations* that have now been discarded from the working tree to preserve the GitHub baseline.

## 6. Vercel vs GitHub relationship
**INFERRED:** Vercel is building directly from the `origin/main` branch on GitHub. It is running the manual Driver Code entry flow.

## 7. Authentication-flow comparison
- **GitHub flow:** `Signup -> manual code -> Supabase Auth signUp -> Next.js API inserts to drivers table`. (Violates Q1 Atomicity). `Login -> Email/Password -> Supabase Auth`.
- **Local flow (discarded):** `Signup -> Supabase Auth signUp -> Next.js API inserts to drivers (DB trigger creates code) -> returns code`. (Still violates Q1). `Login -> Driver Code -> API queries drivers table for email via service_role -> Supabase Auth via email`. (Security gap).

## 8. Q1 (Identity Consistency) Findings
- **VERIFIED:** Neither environment satisfies the Q1 1:1 Invariant. 
- **VERIFIED:** Both environments use sequential API calls (`Supabase.auth.signUp()` followed by `supabase.from('drivers').insert()`). This violates atomicity and allows orphan `auth.users` records.
- **VERIFIED:** The `freight_identities` generic anchor table does NOT exist in either environment.

## 9. Q2 (Active-Gate) Findings
- **VERIFIED:** Email confirmation is NOT currently enforced as a hard gate before application access.
- **VERIFIED:** The `freight_identities` table is missing, so `verification_status` and `trusted_role` columns do not exist.
- **VERIFIED:** Protected access currently relies solely on the existence of a `drivers` record, meaning the Q2 Active Gate is entirely **missing**.

## 10. Q3 (Session / Middleware) Findings
- **VERIFIED:** Middleware exists and uses `@supabase/ssr` to intercept routes.
- **VERIFIED:** It calls `getUser()`, which handles token refresh automatically.
- **SECURITY GAP:** The middleware only checks if a session exists (authentication); it does NOT check the Q2 Active Gate (authorization/verification status), because those concepts don't exist in the database yet.

## 11. Q4 (Identity Model) Findings
- **VERIFIED:** The codebase directly binds `auth_id` to the `drivers` table. 
- **VERIFIED:** This violates Q4, which requires the `freight_identities` table to serve as the generic anchor to support multiple roles (Driver/Company).

## 12. Q5 (Rate-Limit) Findings
- **VERIFIED:** No custom application-level rate limiter exists.
- **VERIFIED:** The application relies on Supabase GoTrue rate limits natively. This **satisfies** the Q5 MVP requirement (Policy C).

## 13. Q6 (RLS / Service-Role) Findings
- **VERIFIED:** `src/app/api/auth/signup/route.ts` uses `supabaseServer` (which is `service_role` powered) to insert the driver record.
- **VERIFIED:** This violates Q6 (Strict DMZ). The insertion should happen via a Postgres `auth.users` trigger, not via a `service_role` API route.

## 14. Driver ID / Driver Code Findings
- **VERIFIED:** Driver Code is a valid business identifier (e.g. for logistics tracking).
- **ARCHITECTURE CONFLICT:** The local attempt to use Driver Code as the primary login credential proxying to an email address creates security risks (enumeration) and complexity. Under Node 2, Driver Code should remain a post-verification application identifier, while Supabase Auth handles standard email/password login.

## 15. Database / Migration Findings
- **`004_auto_generate_driver_code.sql`:** 
  - Existed locally, now removed from working tree.
  - Generates a sequence and trigger. Safe SQL, but applied to the wrong architectural layer (legacy Node 1 `drivers` table).
  - Node 2 requirement missing: We need migrations for `freight_identities` and the Q1 Auth Trigger instead.

## 16. Build / Test Evidence
- **TypeScript:** `npx tsc --noEmit` succeeds cleanly on `origin/main`.
- **Lint:** No critical errors blocking build.

## 17. Finding Classification Table

| Finding | Classification | Recommendation |
|---|---|---|
| Sequential API Signup | Security gap / Node 2 missing (Q1) | Replace with DB Trigger |
| Missing `freight_identities` | Node 2 missing (Q4) | Implement migration |
| Missing Q2 Active Gate | Security gap / Node 2 missing | Implement in Middleware |
| Supabase SSR Middleware | Node 2 satisfied (Q3) | Keep, but enhance with Q2 check |
| Supabase Rate Limiting | Node 2 satisfied (Q5) | Keep |
| `service_role` API inserts | Security gap (Q6) | Remove; rely on trigger |
| Driver Code Login Flow | Local-only / Security gap | Discard (use standard email login) |

## 18. What is VERIFIED
- GitHub source is missing the core Node 2 database architecture (`freight_identities`, Auth Trigger).
- Current APIs use sequential DB inserts and `service_role`, violating Q1 and Q6.
- The abandoned local changes attempted to proxy login via Driver Code, which is insecure and unnecessary.

## 19. What remains UNKNOWN
- None. The architectural gaps are fully understood.

## 20. Correct Target Behavior Under Locked Node 2
- Signup: Standard email/password -> `auth.users` trigger creates `freight_identities` (PENDING).
- Login: Email/Password -> Middleware refreshes session -> Server checks Q2 Active Gate.
- Verification: Admin webhook uses `service_role` to set `VERIFIED` and `trusted_role` -> active access granted.

## 21. Focused Investigations Still Required
- None. All Q1-Q7 investigations are complete.

## 22. Push / Discard Recommendation
**DISCARD local changes** related to Driver Code login proxy. Keep the clean `origin/main` GitHub baseline as the starting point for Node 2 implementation.

## 23. Final Conclusion
- **Why are Vercel and localhost different?** Localhost had experimental changes to auto-generate Driver Codes and use them for login. Vercel runs the older manual-entry code.
- **Which differences violate Node 2?** The local login proxy violates security principles. Both environments violate Q1 (atomicity), Q4 (identity table missing), and Q6 (service role abuse).
- **What behavior should be preserved?** The GitHub baseline (Email login, standard SSR Middleware).
- **What must NOT be pushed?** The experimental local `api/auth/login` proxy.
- **What focused investigation comes next?** None. Move immediately to Implementation of Node 2.
