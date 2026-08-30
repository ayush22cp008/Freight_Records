# Chat17 — Day 10 — Node 2 Subnode Investigation

## Title
Reviewer Login + Password Recovery

## Parent Node
Node 2 — Authentication + Identity

## Day
Day 10

## Status
INVESTIGATION OPEN

## Parent Node State
Node 2 remains COMPLETE / ACCEPTED. This Subnode does not reopen or invalidate the Node 2 completion checkpoint.

## Related Current State
Node 3 — Company Trip Creation + Publishing is ON HOLD while this Subnode is investigated and resolved.

## Reason for Subnode
The accepted Node 2 scope established a minimum reviewer workflow and a common Freight Login entry point for Company and Driver authentication. A follow-up product need has now been identified for a dedicated Reviewer Login experience and password recovery capability.

This work is treated as significant unexpected follow-up work rather than silently changing the already-accepted Node 2 scope.

## Scope Under Investigation

### A. Reviewer Login

Determine the correct authentication and routing behavior for a Reviewer account, including:

- Whether Reviewer is represented by the existing application identity model or by a separate reviewer authorization mechanism.
- How a reviewer reaches the Reviewer Login entry point.
- How successful reviewer authentication is distinguished from Company/Driver authentication.
- How the authenticated reviewer is routed to the Reviewer Queue.
- How unauthorized Company/Driver users are prevented from obtaining reviewer access.
- Whether the existing reviewer_authorizations design remains the authoritative authorization mechanism.

### B. Password Recovery

Determine the correct password-recovery behavior for all supported authentication users:

- Company
- Driver
- Reviewer

The intended capability is a secure account-owner password recovery flow, not an administrative mechanism for arbitrarily changing another user's password.

The investigation must establish:

- Existing authentication provider capabilities and current implementation.
- Whether a forgot-password entry point already exists.
- Current email/recovery-link behavior.
- Recovery-session handling.
- Password update behavior after recovery.
- Role/identity behavior after a successful password reset.
- Failure behavior for invalid, expired, reused, or otherwise invalid recovery links/tokens.
- Whether recovery can safely support Reviewer accounts under the existing reviewer authorization model.
- Any rate-limiting or abuse-control requirements already established for authentication.

## Investigation Boundary

Do not implement changes in this investigation.

Do not create a new implementation prompt until the investigation has established:

1. Current source behavior.
2. Current database/authentication model.
3. Existing reviewer authorization behavior.
4. Existing password-reset/recovery behavior.
5. Any mismatch between current behavior and the required product behavior.
6. Root cause of each confirmed gap.
7. The minimal architecture/implementation decision needed to close the gap without violating locked Node 1/Node 2 identity rules.

## Required Investigation Sequence

```text
OBSERVATION
→ INVESTIGATION
→ EVIDENCE
→ ROOT CAUSE
→ DECISION
→ FIX
→ BUILD/TEST
→ AYUSH MANUAL VERIFICATION
```

At this stage only the first five stages are in scope. Fix/implementation remains blocked until the decision is recorded.

## Locked Constraints

The investigation must preserve the following established constraints:

```text
1 Auth User ↔ exactly 1 application identity
1 Auth User ↔ exactly 1 application role
Role = Company OR Driver

Driver Code / Driver ID is NOT an authentication credential.

Valid session ≠ active Freight account ≠ authorization for every operation.

Reviewer authorization must not be inferred merely from Company/Driver trusted_role.
```

The accepted Node 2 reviewer workflow uses a dedicated reviewer authorization mechanism and role-aware access control. Do not replace that with an unsafe role shortcut without explicit architecture evidence.

## Security Questions

The investigation must explicitly check:

- Can an unauthenticated user reach reviewer functionality?
- Can a Company or Driver session access Reviewer Queue without reviewer authorization?
- Can a user manipulate a role or identity identifier to obtain reviewer access?
- Can password recovery reveal whether another account exists through unsafe responses?
- Can a recovery token be reused after password change?
- Can an expired/invalid recovery token change a password?
- Can password reset be used to target another account without possession of the account's recovery channel?
- Are recovery endpoints protected against abuse according to the existing authentication rate-limiting decision?

## Evidence Classification

Use only:

- VERIFIED — directly supported by current source/test/record evidence.
- INFERRED — supported by evidence but not directly demonstrated.
- UNKNOWN — not established yet.

Do not mark a behavior VERIFIED merely because the implementation appears to support it.

## Expected Deliverable

Produce a complete investigation result containing:

```text
Current behavior
Evidence inspected
Confirmed gaps
Root cause
Security implications
Architecture decision required / not required
Recommended minimal fix scope
Verification requirements
```

The result must explicitly state whether this Subnode is:

- a UI-only gap,
- an authentication capability gap,
- an authorization gap,
- a password-recovery implementation gap,
- or a combination of these.

## Completion Gate for This Investigation

The investigation is complete only when the root cause and required decision are clear enough to create a separate implementation plan/prompt without guessing.

No source-code implementation is authorized by this record.

## Record Routing

Investigation:
`05_DEBUGGING/investigations/`

Architecture decision, if required:
`02_ARCHITECTURE/`

Implementation plan/prompt, only after investigation/decision:
`03_IMPLEMENTATION/plans/`
`03_IMPLEMENTATION/prompts/`

Implementation report:
`03_IMPLEMENTATION/implementation_reports/`
