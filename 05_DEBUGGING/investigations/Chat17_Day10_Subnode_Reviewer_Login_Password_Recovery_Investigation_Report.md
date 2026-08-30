# Chat17 — Day 10 — Reviewer Login & Password Recovery Investigation Result

## 1. Current Behavior
- **Reviewer Login**: Reviewers authenticate via the shared `src/app/login/page.tsx`. However, upon successful authentication, they are redirected to `/` which is within the `(authenticated)` layout. The `src/app/(authenticated)/layout.tsx` unconditionally requires a valid `freight_identities` record. Since reviewers are authorized via `reviewer_authorizations` (and are not standard Company/Driver users), they fail the `getFreightIdentity()` check and are shown an "Identity not found. Please contact support." block. They can never reach `/reviewer/queue`.
- **Password Recovery**: There is no "Forgot Password" entry point in the login UI, no API route to trigger Supabase password recovery, and no page to handle the password reset token/link (e.g., `/reset-password` or `#recovery` hash handling).

## 2. Evidence Inspected
- `src/app/login/page.tsx`: Shows standard Supabase email/password login with no password recovery links.
- `src/app/(authenticated)/layout.tsx`: Blocks any authenticated user without a `freight_identities` record.
- `src/app/(authenticated)/reviewer/queue/page.tsx`: Contains proper `reviewer_authorizations` check, but is unreachable due to the layout block.
- `src/lib/auth.ts`: `getFreightIdentity()` only queries `freight_identities`.

## 3. Confirmed Gaps
- **Gap 1**: Reviewers cannot access the application after logging in because they lack a `freight_identities` record.
- **Gap 2**: No UI or application logic exists for any user (Company, Driver, Reviewer) to recover or reset a forgotten password.

## 4. Root Cause
- **Root Cause 1**: `src/app/(authenticated)/layout.tsx` was designed under the assumption that all authenticated users are standard Freight users (Company/Driver). It does not account for administrative/Reviewer identities governed by `reviewer_authorizations`.
- **Root Cause 2**: Password recovery was not implemented in the MVP phase; standard Supabase `resetPasswordForEmail` capabilities were never exposed in the UI.

## 5. Security Implications
- **Reviewer Authorization**: The security of `reviewer_authorizations` remains intact. The current bug *over-restricts* reviewers (Denial of Service) rather than leaking access. Unauthorized users cannot reach the queue.
- **Password Recovery**: Without a built-in recovery flow, if a user loses their password, an admin must manually intervene. If we implement password recovery, we must ensure the token is handled securely (standard Supabase PKCE flow) and that it cannot be reused.

## 6. Architecture Decision Required / Not Required
- **Not Required**: No new architecture rules are needed. The existing distinction between `freight_identities` (app users) and `reviewer_authorizations` (admins) is correct according to Node 2 constraints. The layout simply needs to route them properly. Supabase's native auth handles the password recovery architecture.

## 7. Recommended Minimal Fix Scope
- **Fix 1 (Reviewer Routing)**: Update `src/app/(authenticated)/layout.tsx` to check `reviewer_authorizations` if `getFreightIdentity()` returns null. If authorized as a reviewer, allow layout rendering and redirect them to `/reviewer/queue` instead of blocking them.
- **Fix 2 (Password Recovery)**: 
  - Add a "Forgot Password?" link to `src/app/login/page.tsx`.
  - Create a new UI route (e.g., `/forgot-password`) to call Supabase `resetPasswordForEmail`.
  - Create a new UI route (e.g., `/update-password`) to handle the recovery session and allow the user to submit a new password securely.

## 8. Verification Requirements
- Verify a user *with* `reviewer_authorizations` but *without* `freight_identities` can log in and reach `/reviewer/queue`.
- Verify standard Company/Driver users are unaffected.
- Verify a user can request a password reset email and use the link to update their password.

## 9. Subnode Classification
This Subnode represents a combination of:
- **An authorization/routing gap** (Layout strictly requiring `freight_identities` blocking Reviewers).
- **A password-recovery implementation gap** (Missing UI and routes for standard Supabase recovery flow).
