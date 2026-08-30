# Chat17 — Day 10 — Node 2 Subnode Implementation Plan

## Title
Reviewer Login + Password Recovery

## Parent
Node 2 — Authentication + Identity

## Subnode Status
IMPLEMENTATION PLAN — READY FOR APPROVAL

## Parent / Execution State
- Node 2 remains COMPLETE / ACCEPTED.
- This is a controlled follow-up Subnode and does not reopen Node 2.
- Node 3 is ON HOLD until this Subnode is resolved and released.

## Investigation Basis
Authoritative investigation:
`05_DEBUGGING/investigations/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Investigation_Report.md`

The investigation confirmed two implementation gaps:

1. Reviewer authentication succeeds through the shared login page but the authenticated layout requires a `freight_identities` record, preventing reviewers authorized through `reviewer_authorizations` from reaching `/reviewer/queue`.
2. No password recovery entry point, recovery flow, or password-update page exists.

## Locked Architecture Constraints

Preserve the existing identity and authorization model:

```text
1 Auth User ↔ exactly 1 application identity
1 Auth User ↔ exactly 1 application role
Role = Company OR Driver

Reviewer authorization → reviewer_authorizations
Company / Driver identity → freight_identities
```

Do not create a fake `freight_identities` record for a Reviewer solely to satisfy the authenticated layout.

Do not convert Reviewer into a Company or Driver role.

Do not use Driver Code / Driver ID as authentication credentials.

Do not weaken reviewer authorization by treating authentication alone as reviewer authorization.

## Implementation Objective

Make the smallest safe changes necessary to:

```text
Reviewer
→ shared login
→ authenticated session
→ reviewer authorization check
→ Reviewer Queue
```

and:

```text
Company / Driver / Reviewer
→ Forgot Password
→ recovery email
→ secure recovery session
→ choose new password
→ return to normal authenticated flow
```

## Workstream A — Reviewer Login / Routing

### A1. Preserve the shared login page

Keep `src/app/login/page.tsx` as the common authentication entry point. Do not create unnecessary duplicate credential authentication logic.

Add or preserve role-neutral login behavior.

### A2. Correct authenticated-layout handling

Update `src/app/(authenticated)/layout.tsx` so that absence of `freight_identities` does not automatically block a legitimately authorized Reviewer.

The layout must distinguish:

```text
Valid Company/Driver identity
OR
Valid Reviewer authorization
```

before deciding that an authenticated user has no valid application access.

### A3. Reviewer authorization

Use the existing `reviewer_authorizations` mechanism as the authoritative reviewer-access check.

A valid reviewer session must not be granted access merely because the user is authenticated.

The reviewer authorization check must be server-side / authoritative for protected reviewer functionality.

### A4. Reviewer routing

After successful login, a user authorized as Reviewer must be able to reach:

`/reviewer/queue`

Do not break existing Company/Driver routing.

### A5. Unauthorized access

A Company or Driver user without reviewer authorization must remain unable to access `/reviewer/queue`.

An authenticated user with neither a valid `freight_identities` record nor valid reviewer authorization must remain blocked.

## Workstream B — Password Recovery

### B1. Login entry point

Add a clearly visible `Forgot Password?` entry point to `src/app/login/page.tsx`.

The wording should be role-neutral because Company, Driver, and Reviewer users use the same recovery mechanism.

### B2. Recovery request page

Create a dedicated route such as:

`/forgot-password`

The page should accept an email address and initiate the existing Supabase password-recovery capability through `resetPasswordForEmail`.

The implementation should use the configured application recovery URL rather than exposing internal implementation details to the user.

### B3. Account-enumeration resistance

Do not expose whether an email belongs to a registered account through distinct success/failure UI or response behavior where the existing auth design can avoid it.

Use a neutral recovery-request result.

### B4. Recovery callback/session handling

Create a dedicated password-update route such as:

`/update-password`

Handle the Supabase recovery session correctly, including the existing project authentication/session conventions.

Do not treat an arbitrary query parameter or email address as proof of authorization to change a password.

### B5. New password

Allow the authenticated recovery session to set a new password using the supported Supabase password-update operation.

Require appropriate password validation consistent with the application's existing authentication rules.

### B6. Invalid / expired / reused recovery state

The UI must fail safely when the recovery session/link is invalid, expired, or otherwise unusable.

A stale or invalid recovery artifact must not permit a password change.

### B7. Post-reset behavior

After successful password update:

- establish/retain the valid authentication state according to Supabase behavior;
- route the user into the appropriate existing application flow;
- do not bypass verification or reviewer authorization gates.

A password reset must not change:

- application role,
- trusted role,
- verification status,
- reviewer authorization,
- identity ownership.

## Security Requirements

The implementation must explicitly preserve:

1. Reviewer authorization is separate from ordinary Company/Driver identity.
2. Authentication alone is insufficient for reviewer access.
3. Password recovery does not authorize access to another account.
4. Recovery artifacts cannot be reused after invalidation/consumption according to provider behavior.
5. Password recovery does not bypass existing verified-account or role-aware application gates.
6. Existing authentication rate-limiting decisions must not be weakened.
7. No secrets or service-role credentials are exposed to the client.

## Files Expected to Be Evaluated

At minimum inspect and modify only where required:

```text
src/app/login/page.tsx
src/app/(authenticated)/layout.tsx
src/app/(authenticated)/reviewer/queue/page.tsx
src/lib/auth.ts
```

plus the minimum new recovery routes/components required by the existing application structure.

Do not perform unrelated refactors.

## Verification Matrix

### Reviewer

```text
1. Reviewer credentials authenticate successfully.
2. Reviewer without freight_identities but with reviewer_authorizations reaches /reviewer/queue.
3. Reviewer Queue remains protected.
4. Company user cannot access Reviewer Queue.
5. Driver user cannot access Reviewer Queue.
6. Authenticated user without either valid application identity or reviewer authorization remains blocked.
7. Company routing remains correct.
8. Driver routing remains correct.
```

### Password recovery

```text
9. Login displays Forgot Password.
10. Recovery request accepts an email.
11. Recovery request produces neutral user-facing behavior.
12. Valid recovery email/link opens the password-update flow.
13. New password can be set through a valid recovery session.
14. Invalid/expired recovery state cannot change password.
15. Password reset does not change role/identity/verification/reviewer authorization.
16. Company can log in with the new password.
17. Driver can log in with the new password.
18. Reviewer can log in with the new password and still reaches reviewer flow.
```

## Build / Test Requirements

Before implementation can be declared complete:

- TypeScript/typecheck must pass.
- Lint must pass if configured.
- Production build must pass.
- Existing tests must pass.
- New targeted authentication/reviewer/recovery tests must pass where the project test structure supports them.
- Evidence must be recorded in the Antigravity implementation report.
- Ayush must manually verify the critical flows.

## Implementation Boundary

Antigravity must not:

- redesign the identity model;
- create Reviewer `freight_identities` merely to bypass the bug;
- make Reviewer a Company/Driver role;
- remove reviewer authorization checks;
- introduce Driver-ID login;
- implement an administrator password override;
- modify Node 3 functionality;
- perform unrelated cleanup/refactoring.

## Approval State

This plan is based on the completed Chat17 Day 10 investigation. It is ready for Ayush approval before an implementation prompt is created for Antigravity.

No source-code implementation is authorized by this plan until the plan is explicitly approved.
