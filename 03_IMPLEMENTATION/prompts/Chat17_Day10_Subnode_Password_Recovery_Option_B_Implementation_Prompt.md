# Chat17 — Day 10 — Password Recovery — Option B Implementation Prompt

## Execution Role
You are Antigravity, the implementation/execution agent.

Implement **only the password-recovery portion** of the approved Chat17 Day 10 Subnode. Reviewer authentication remains a separate workstream and must not be implemented by this prompt.

## Authoritative Records

Read these Records before changing code:

```text
02_ARCHITECTURE/Chat17_Day10_Password_Recovery_Option_B_Architecture_Decision.md
05_DEBUGGING/investigations/Chat17_Day10_Subnode_Password_Recovery_Fresh_Link_Failure_Investigation_Report.md
02_ARCHITECTURE/Chat17_Day10_Password_Recovery_Decisions.md
03_IMPLEMENTATION/plans/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Plan.md
```

The architecture decision is approved and locked as **Option B — token-hash-based recovery**.

## Problem Being Fixed

Current production evidence:

```text
Forgot Password request       → works
Recovery email delivery       → works
Fresh recovery link clicked once → fails
Result                        → /login?error=Invalid+or+expired+recovery+link
Supabase error                → access_denied / otp_expired
```

Current implementation mixes the application's `@supabase/ssr`/PKCE conventions with a `/api/auth/confirm` route that expects `token_hash` + `type`. The existing default Supabase `{{ .ConfirmationURL }}` link can also be consumed by email-security prefetchers before the user clicks it.

Do not treat the observed `otp_expired` as user delay; a fresh email was clicked once and failed.

## Locked Architecture

Implement this flow:

```text
Forgot Password
    ↓
resetPasswordForEmail()
    ↓
Supabase Reset Password email
    ↓
Custom Freight application recovery URL containing the recovery TokenHash
    ↓
Freight recovery page
    ↓
Explicit user action verifies token_hash as recovery
    ↓
Valid recovery session
    ↓
Set New Password
    ↓
Confirm New Password
    ↓
updateUser({ password })
    ↓
Success
    ↓
Login / normal application flow
```

The custom recovery application page must NOT automatically consume the single-use recovery credential merely because an email scanner or browser prefetcher loads the page. Verification/consumption must occur only as part of the intended user-driven recovery action.

## Production Application URL

Use the existing Vercel deployment domain:

```text
https://freighthackathon.vercel.app
```

No custom/purchased domain is required.

The exact recovery URL/path chosen by the implementation must be allowlisted in Supabase Authentication → URL Configuration.

## Supabase Dashboard Configuration — Required Manual Step

Custom SMTP has already been configured so Auth email templates are editable.

Do NOT create a new SMTP configuration and do NOT change the sender configuration unless a concrete implementation blocker requires it.

Update **Authentication → Emails → Reset password** only after the application recovery route is implemented and deployed/ready for testing.

Replace the default link that currently uses:

```text
{{ .ConfirmationURL }}
```

with a custom Freight recovery link that passes the recovery token hash using the supported Supabase template variable:

```text
{{ .TokenHash }}
```

The final link must target the production Freight recovery route selected in the source implementation. It must not directly point to a Supabase verification endpoint.

Do not claim this dashboard step is complete unless it is actually configured in Supabase.

## Source Implementation Requirements

### 1. Forgot Password entry point

Preserve the existing role-neutral `Forgot Password?` link on the shared login page.

It must continue to route to the dedicated password-recovery request page.

### 2. Recovery request

Use Supabase Auth's supported password-recovery request method (`resetPasswordForEmail`).

The request must use the production Freight recovery URL as the intended redirect/application destination, consistent with the locked architecture.

Preserve neutral account-existence behavior. Do not reveal whether an email is registered.

Preserve existing authentication/rate-limit decisions.

### 3. Recovery route

Create or update the dedicated password recovery route, preferably `/update-password` if consistent with the existing source structure.

The route/page must be capable of receiving the recovery token hash from the custom email link.

Do not automatically call the single-use token verification merely because the page rendered. The page may display a clear action such as:

```text
Continue to reset password
```

The token should be verified only through the intended explicit user action.

### 4. Recovery token verification

Use the supported Supabase recovery verification mechanism with:

```text
verifyOtp({
  token_hash,
  type: 'recovery'
})
```

Use the project's existing Supabase browser/server client conventions appropriately.

Do not treat an email address, arbitrary query parameter, or user-supplied identifier as proof of recovery authorization.

Do not expose service-role credentials.

### 5. Recovery session

After successful verification, ensure a valid Supabase recovery/auth session exists before allowing a password update.

Do not permit `updateUser({ password })` when recovery verification failed, is expired, or is otherwise unusable.

### 6. Password form

Provide:

```text
New Password
Confirm New Password
```

Validate that the two values match before submission.

Apply the existing Supabase/project password policy; do not invent a conflicting password policy.

Never display the password value in logs, URLs, error messages, or implementation reports.

### 7. Password update

Use the supported Supabase password-update operation (`updateUser({ password })`) only after a valid recovery session has been established.

A successful reset changes only the password.

It must NOT change:

```text
Application role
Application identity
Verification status
Reviewer authorization
Business data
```

### 8. Success / failure behavior

Successful reset:

```text
Password updated
    ↓
Clear success state
    ↓
Return user to Login / normal application flow
```

Invalid/expired/reused recovery credential:

```text
No password change
    ↓
Safe recovery error
    ↓
Offer to request a new reset email / return to Login
```

Do not leak internal Supabase errors unnecessarily.

### 9. Existing confirmation route

Do NOT delete `/api/auth/confirm` before confirming whether any other authentication flow uses it.

After implementation, determine whether it is unused by password recovery and document the result. Remove/change it only if evidence shows that it is safe and required by this implementation.

Do not perform unrelated authentication refactoring.

## Security Requirements

1. Recovery credential must be verified before password update.
2. Email prefetch must not consume the recovery credential simply by loading the application recovery page.
3. Single-use recovery behavior must remain enforced by Supabase.
4. Invalid/expired/reused recovery credentials cannot change passwords.
5. No account enumeration through distinct recovery-request responses.
6. No service-role secret exposed to the browser.
7. No role/identity/authorization changes during password recovery.
8. Existing auth rate limiting must remain intact.
9. No permanent application-managed reset-token store may be introduced.
10. No administrator password override may be introduced.

## Scope Boundary

Do NOT:

- implement Reviewer Authentication in this prompt;
- modify Node 3;
- redesign the identity model;
- create Reviewer `freight_identities` records;
- make Reviewer a Company/Driver role;
- restore Driver-ID/Driver-Code login;
- implement a separate Change Password feature;
- perform unrelated refactoring;
- weaken authorization checks.

## Testing Requirements

Run the project's appropriate:

```text
TypeScript/typecheck
Lint (if configured)
Production build
Existing test suite
Targeted auth/recovery tests where supported
```

At minimum verify:

```text
1. Login shows Forgot Password.
2. Forgot Password accepts an email.
3. Recovery request uses neutral response behavior.
4. Fresh recovery email is received through configured SMTP.
5. Fresh recovery link loads the Freight recovery page.
6. Loading the recovery page alone does not consume the recovery credential.
7. Explicit Continue/verification succeeds for a fresh valid token.
8. New Password + Confirm Password are required and must match.
9. Valid recovery session allows password update.
10. Successful reset returns to Login/normal flow.
11. New password can authenticate the same account.
12. Old password no longer authenticates, subject to normal Supabase session/password behavior.
13. Expired/reused/invalid recovery credential cannot change the password.
14. Password reset does not change role, identity, verification status, or reviewer authorization.
15. Company, Driver, and Reviewer accounts can use the same recovery mechanism.
```

## Manual Verification Required From Ayush

Antigravity must NOT mark the flow VERIFIED based only on automated tests.

After deployment/configuration, provide Ayush with the exact manual test sequence:

```text
1. Open Freight Login.
2. Click Forgot Password.
3. Enter the test account email.
4. Submit once.
5. Open the newest Supabase/Resend email.
6. Click Reset password once.
7. Confirm the Freight recovery page opens.
8. Click the explicit recovery/Continue action.
9. Enter New Password.
10. Enter Confirm New Password.
11. Submit.
12. Confirm success.
13. Log in with the new password.
14. Confirm the user's existing role/authorization remains unchanged.
```

Ayush must report the observed UI result before the implementation can be called manually verified.

## Implementation Report Requirements

Create/update the Antigravity implementation report under:

```text
03_IMPLEMENTATION/implementation_reports/
```

The report must distinguish:

```text
IMPLEMENTED
BUILD/TESTED
CONFIGURED IN SUPABASE
MANUALLY VERIFIED BY AYUSH
UNKNOWN / NOT VERIFIED
```

Record exact files changed, tests executed and their results, Supabase configuration changes actually performed, deployment evidence if available, and any remaining issue.

Do not claim password recovery is complete until the fresh-link manual test succeeds.

## Completion Gate

The implementation is complete only when:

```text
Option B architecture implemented
        ↓
Supabase Reset Password template configured
        ↓
Fresh recovery email generated
        ↓
Fresh link clicked once
        ↓
Freight recovery page reached
        ↓
Token verified successfully
        ↓
New Password + Confirm Password accepted
        ↓
Password updated
        ↓
Login with new password succeeds
        ↓
Role/identity/authorization preserved
        ↓
Build/tests pass
        ↓
Ayush manual verification recorded
```

If any stage fails, report the evidence and stop rather than masking the failure or inventing a workaround.
