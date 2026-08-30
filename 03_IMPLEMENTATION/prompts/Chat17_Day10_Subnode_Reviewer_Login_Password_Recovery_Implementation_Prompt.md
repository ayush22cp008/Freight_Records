# Chat17 — Day 10 — Node 2 Subnode — Antigravity Implementation Prompt

## Execution Authority

Implement the approved Chat17 Day 10 Node 2 Subnode plan in the application source repository.

Source repository:
`ayush22cp008/freight_hackathon`

Records repository:
`ayush22cp008/Freight_Records`

Approved plan:
`03_IMPLEMENTATION/plans/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Plan.md`

Investigation:
`05_DEBUGGING/investigations/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Investigation_Report.md`

## Important Execution Boundary

This prompt is the implementation stage only. Do not redo the investigation unless the source has materially changed and the plan is contradicted by current evidence.

Do not modify Node 3 functionality. Node 3 is currently ON HOLD.

Do not change the locked Node 1/Node 2 identity architecture.

## Objective

Implement the approved minimal fix for:

1. Reviewer Login / routing so an authorized Reviewer can authenticate through the shared login and reach `/reviewer/queue`.
2. Secure password recovery for Company, Driver, and Reviewer users through the existing Supabase authentication model.

## Reviewer Implementation

### 1. Shared login

Keep `src/app/login/page.tsx` as the shared email/password authentication entry point.

Do not introduce Driver-ID/Driver-Code authentication or duplicate credential logic.

### 2. Authenticated layout

Update `src/app/(authenticated)/layout.tsx` so that a missing `freight_identities` record does not automatically block a legitimately authorized Reviewer.

The access decision must support:

```text
valid freight identity
OR
valid reviewer authorization
```

Do not manufacture a `freight_identities` row for a Reviewer just to satisfy the layout.

### 3. Reviewer authorization

Use the existing `reviewer_authorizations` mechanism as the authoritative reviewer-access check.

Authentication alone must not grant Reviewer access.

### 4. Reviewer routing

An authenticated authorized Reviewer must be able to reach:

`/reviewer/queue`

Preserve existing Company and Driver routing behavior.

### 5. Unauthorized access

A Company or Driver user without valid reviewer authorization must remain unable to access reviewer functionality.

An authenticated user with neither a valid freight identity nor valid reviewer authorization must remain blocked.

## Password Recovery Implementation

### 6. Login UI

Add a clear `Forgot Password?` link to `src/app/login/page.tsx`.

Keep the wording role-neutral.

### 7. Forgot-password page

Create the minimum route required by the existing application structure, preferably:

`/forgot-password`

Use Supabase's `resetPasswordForEmail` capability.

Use the application's configured recovery redirect URL.

### 8. Enumeration resistance

Use neutral user-facing behavior for recovery requests. Do not disclose whether a submitted email exists as an account where the existing authentication provider supports a neutral response.

### 9. Password-update page

Create the minimum route required by the existing application structure, preferably:

`/update-password`

Handle the Supabase recovery session correctly. A query parameter or email address alone must never be treated as authorization to change a password.

### 10. Set new password

Allow the valid recovery session to set a new password through the supported Supabase password-update operation.

Apply the existing application's password validation requirements.

### 11. Invalid/expired recovery state

Invalid, expired, or otherwise unusable recovery state must fail safely and must not allow password changes.

### 12. Post-reset behavior

After successful reset, preserve the user's existing:

- application role
- identity
- verification status
- reviewer authorization

Password reset must not bypass existing role-aware or verification gates.

Route the user into the normal application flow appropriate to the authenticated account.

## Security Requirements

Explicitly preserve all of the following:

- Reviewer authorization is separate from Company/Driver identity.
- Authentication alone is insufficient for reviewer access.
- No administrator password override is being implemented.
- Password recovery cannot target another account without valid recovery authorization.
- Recovery artifacts cannot be used after they are invalidated/consumed according to provider behavior.
- No service-role credentials or secrets are exposed client-side.
- Existing authentication rate-limiting behavior is not weakened.
- Driver Code / Driver ID remains non-authentication data.

## Scope Control

Only modify files required for this subnode.

Expected areas to inspect:

```text
src/app/login/page.tsx
src/app/(authenticated)/layout.tsx
src/app/(authenticated)/reviewer/queue/page.tsx
src/lib/auth.ts
```

plus the minimum new password recovery route/component files required.

Do not perform unrelated refactors, dependency upgrades, styling redesigns, or Node 3 work.

## Verification Required Before Reporting Completion

Run the project's available checks and record exact commands/results.

### Reviewer tests

```text
1. Reviewer credentials authenticate.
2. Reviewer with reviewer_authorizations and without freight_identities reaches /reviewer/queue.
3. Reviewer Queue remains protected.
4. Company cannot access Reviewer Queue without reviewer authorization.
5. Driver cannot access Reviewer Queue without reviewer authorization.
6. User with neither valid identity nor reviewer authorization remains blocked.
7. Company routing remains correct.
8. Driver routing remains correct.
```

### Password recovery tests

```text
9. Login shows Forgot Password.
10. Forgot-password request works.
11. Recovery request is neutral regarding account existence.
12. Valid recovery link opens the password-update flow.
13. Valid recovery session can set a new password.
14. Invalid/expired recovery state cannot change password.
15. Password reset does not modify role/identity/verification/reviewer authorization.
16. Company can log in with the new password.
17. Driver can log in with the new password.
18. Reviewer can log in with the new password and reach the reviewer flow.
```

### Engineering checks

At minimum run the repository's applicable:

- TypeScript/typecheck
- lint
- production build
- existing automated tests
- targeted tests for reviewer authorization and password recovery where supported

Do not claim a check passed unless its actual command/result is available.

## Manual Verification Handoff

After implementation and automated checks, provide clear steps for Ayush to manually verify:

1. Reviewer login → Reviewer Queue.
2. Company/Driver remain correctly routed.
3. Forgot Password → email → reset password → login.
4. Password reset works for Company, Driver, and Reviewer.
5. Unauthorized reviewer access remains blocked.

## Implementation Report

After completion, create the implementation report in the Records repository under:

`03_IMPLEMENTATION/implementation_reports/`

Use a Chat17 / Day10 filename that clearly identifies this Subnode.

The report must include:

```text
Implementation status
Files changed
Source commit SHA
Commands executed
Build/lint/test results
Targeted security/behavior results
Known limitations
Ayush manual verification status
```

Do not mark the Subnode complete based only on code changes.

## Final Rule

Do not silently expand scope. If implementation reveals a major architecture conflict, stop and report it through the Records bridge instead of improvising a new architecture.
