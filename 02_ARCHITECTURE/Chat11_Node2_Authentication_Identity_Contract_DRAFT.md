# Chat12 — Node 2 Authentication + Identity Contract — DRAFT

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Status:** DRAFT / NOT LOCKED / REQUIRES AYUSH REVIEW  
**Chat:** Chat12  

## 1. Purpose

Define the authentication, application-identity, role-establishment, and verification contract required before authentication implementation resumes.

This draft is derived from the locked Node 1 product/authorization model and the Chat11/Chat12 investigation records.

This document is a design draft. It does not authorize implementation.

## 2. Authoritative Node 1 dependency

Node 1 locks:

```text
1 Auth User ↔ exactly 1 application identity
1 Auth User ↔ exactly 1 application role
Role = Company OR Driver
```

Node 2 must provide a trustworthy authenticated identity context that Node 1 authorization can consume.

Node 2 must not redefine Node 1 authorization rules.

## 3. Current verified baseline

The Chat11/Chat12 investigations established:

- Supabase Auth is used.
- Driver authentication currently exists.
- Driver records map to `auth.users` through `drivers.auth_id`.
- `drivers.auth_id` is UNIQUE.
- Current authentication assumes every authenticated user is a Driver.
- No Company identity currently exists.
- No generic application identity model currently exists.
- No explicit role model currently exists.
- Protected UI/API access performs server-side authentication checks.
- The server currently obtains Supabase Auth user identity and then manually looks up Driver identity.
- Authentication-related local source changes reported by Antigravity are not committed/pushed to the source repository.
- Driver codes are sequential identifiers such as `DRV010`; they are database-unique but enumerable.
- No application-level authentication rate limiter currently exists; current implementation relies on native Supabase Auth limits.
- Server-side authentication code uses a service-role client for selected identity operations; service-role credentials remain server-only.
- No automated authentication/identity tests were found in the source repository.
- Current signup creates Auth User and application Driver identity through separate operations and is therefore not atomic.
- Current sequential signup can leave an orphaned Auth User if Driver identity creation fails.
- Server-side compensation is technically available but is unsafe as the primary consistency mechanism for unknown-outcome failures because the current `ON DELETE SET NULL` relationship can leave an orphaned Driver record.
- A PostgreSQL `auth.users` trigger is technically available and can create an application identity within the Auth User insertion transaction.
- The pending-identity investigation found Model C (generic pending Freight Identity → verification → trusted role) to be the strongest fit for the Node 1 invariant and the trigger consistency model.

These are evidence findings, except where explicitly marked as a proposed contract decision below.

## 4. Contract goals

Node 2 must establish:

1. A single trustworthy application identity for each authenticated user.
2. Exactly one application role for each authenticated user after verification.
3. Explicit Company vs Driver identity semantics.
4. A controlled verification process for establishing the trusted role.
5. Server-derived identity context for protected requests.
6. No authorization trust in client-supplied identity or trusted-role fields.
7. Authentication behavior that can safely feed Node 1 authorization.
8. A defined signup/onboarding consistency model.
9. A defined email-confirmation and account-activation model.
10. A defined session lifecycle.
11. A defined authentication abuse/rate-limiting boundary.
12. Testable acceptance criteria.

## 5. Proposed identity model — MODEL C / PROPOSED

The leading model is a generic Freight Identity created atomically with the Auth User:

```text
Supabase Auth User
        │
        │ exactly one
        ▼
Generic Freight Identity
        │
        ├── requested_role = Company / Driver
        ├── verification_status = PENDING / VERIFIED / REJECTED
        └── trusted_role = NULL until verification
```

After successful verification:

```text
Generic Freight Identity
        ↓
verification_status = VERIFIED
        ↓
trusted_role = Company OR Driver
        ↓
Role-specific application identity/access
```

The application identity must be derived from the authenticated Supabase Auth user, not from a client-provided user/driver/company ID.

The exact database schema and role-specific mapping remain subject to independent review and final contract lock.

## 6. One-user / one-identity invariant — **DECIDED / LOCKED FOR NODE 2**

The system must enforce:

```text
ONE auth.users.id
        ↓
ONE Freight application identity
```

The **database must enforce uniqueness** of the Auth User reference in the Freight Identity table. Conceptually:

```text
freight_identities.auth_user_id = UNIQUE
```

This database-level uniqueness is the authoritative enforcement mechanism for the one-user/one-identity invariant. The PostgreSQL Auth trigger is the atomic creation mechanism, but the trigger alone is not considered sufficient enforcement.

The user must not simultaneously become both Company and Driver.

The requested role is not the trusted role.

The trusted role is established only by the server-controlled verification process.

### Q4 Independent review

Claude reviewed Q4 and returned **CONCERN**, specifically because the previous draft did not explicitly specify a DB-level uniqueness guarantee. The decision is now resolved by explicitly requiring a unique database constraint on the Auth User reference before implementation/lock. No Q4 blocker was identified. The independent review is recorded in:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md`

## 7. Requested role vs trusted role — PROPOSED

At signup, the user may request:

```text
requested_role = Company
```

or:

```text
requested_role = Driver
```

This is **user-provided intent, not authorization**.

The system must never treat a client-controlled requested role, user metadata role, or client-supplied role field as proof of authorization.

The server-controlled fields are conceptually:

```text
verification_status = PENDING | VERIFIED | REJECTED
trusted_role = NULL | Company | Driver
```

Only an authorized verification action may transition a pending identity to a trusted role.

## 8. Verification model — PROPOSED / HACKATHON MVP

For the hackathon MVP, verification is a controlled human-review workflow.

```text
User signup
    ↓
Select requested role
    ↓
Provide official evidence
    ↓
verification_status = PENDING
    ↓
Ayush reviews evidence
    ↓
APPROVE or REJECT
    ↓
If approved:
verification_status = VERIFIED
trusted_role = requested_role
```

Evidence currently selected for the MVP:

```text
Driver  → Driving Licence evidence
Company → GST evidence/details
```

The user cannot approve their own verification and cannot directly modify `verification_status` or `trusted_role`.

The exact reviewer interface and audit-log implementation remain to be finalized before implementation.

## 9. Pending identity security boundary — PROPOSED

A PENDING identity may exist for consistency and verification workflow purposes but is not an authorized Driver or Company.

A PENDING identity must not:

- perform protected Driver-only operations;
- perform protected Company-only operations;
- appear as an eligible Driver in marketplace queries;
- create/own protected business resources requiring a trusted role;
- bypass verification through client-supplied role fields.

Role-specific business tables should be structurally or policy-wise isolated from PENDING identities where practical.

## 10. Signup / onboarding consistency — **DECIDED / LOCKED FOR NODE 2**

The current sequential signup flow is not an acceptable final consistency boundary because it can leave:

```text
Auth User exists
BUT
required application identity does not exist
```

The selected architecture direction is **PostgreSQL-trigger-based atomic creation of the generic Freight Identity**.

Conceptually:

```text
Supabase Auth User creation
        ↓
PostgreSQL transaction
        ↓
auth.users INSERT trigger
        ↓
Generic Freight Identity (PENDING)
        ↓
Both succeed → durable consistent state

OR

Identity creation fails
        ↓
Transaction rolls back
        ↓
No durable Auth User without Freight Identity
```

The trigger is the selected atomic identity-creation mechanism. It is not yet implementation authorization by itself.

Server-side compensation is not selected as the primary consistency mechanism because unknown-outcome timeouts can cause Auth deletion while leaving a Driver record through the current `ON DELETE SET NULL` relationship.

### Q1 Independent review

Claude reviewed Q1 and returned **APPROVE**. Claude agreed that trigger-based atomic creation correctly closes the verified Auth User/application identity orphan gap. The remaining exact trigger implementation/error handling is implementation detail and must still be validated before implementation is considered verified.

## 11. Email confirmation and account activation — OPEN POLICY DECISION

Identity existence must not be treated as equivalent to an active or authorized account.

The contract distinguishes:

```text
Identity exists
    ≠
Email confirmed
    ≠
Verification completed
    ≠
Authorized
    ≠
Active/usable account
```

The final contract must explicitly define:

- whether email confirmation is required;
- unconfirmed login behavior;
- whether verification may proceed before email confirmation;
- when a verified identity becomes active/usable;
- failure behavior for each state.

This remains OPEN until independently reviewed and approved.

## 12. Driver Code authentication — DRAFT

Driver Code is an identifier, not a secret.

The current sequential format (`DRV###`) is enumerable and therefore must not be treated as sufficient authentication protection by itself.

A pending identity must not receive a trusted Driver code if that code would expose it as an active Driver.

The authentication contract therefore requires:

- password/authentication credential remains the secret factor;
- Driver Code must not be treated as a secret;
- brute-force protection must be explicitly defined;
- generic authentication failures should avoid revealing whether a Driver Code exists.

The final rate-limiting design remains an OPEN decision in this draft.

## 13. Authentication rate limiting — DRAFT

Authentication abuse protection must exist at the application/security boundary appropriate to the chosen implementation.

The final contract must define at least:

- per-IP controls;
- per-account/credential identifier controls;
- shared state where required by deployment architecture;
- behavior when limiter state is unavailable;
- ordering relative to identity lookup;
- failure response behavior.

The current absence of application-level rate limiting is a verified gap, not an approved final architecture.

## 14. Authenticated request context — DRAFT

A protected server request must derive identity from the verified authentication session.

Conceptually:

```text
Request
  ↓
Verified Supabase Auth session/user
  ↓
Freight Identity lookup
  ↓
Verification status + trusted role
  ↓
Node 1 authorization
  ↓
Business operation
```

Client-supplied `driver_id`, `company_id`, `role`, `trusted_role`, or `verification_status` must never be accepted as proof of identity or authorization scope.

## 15. Verification authority boundary — PROPOSED

The hackathon MVP uses a single controlled human verifier: Ayush.

The verifier action must occur through a server-authorized path rather than direct client/database manipulation.

Conceptually:

```text
Ayush / authorized verifier
        ↓
Server-side verification action
        ↓
Validate pending identity + evidence
        ↓
PENDING → VERIFIED or REJECTED
        ↓
Set trusted_role only on approval
```

The verifier permission mechanism, audit evidence, and exact server endpoint/interface remain to be finalized before implementation lock.

## 16. Session lifecycle — DRAFT

The authentication contract must define:

- session establishment;
- server-side verification;
- refresh behavior;
- logout behavior;
- expired/invalid session behavior;
- cookie/token handling;
- protected-route behavior during refresh failure.

The current investigation identified a possible session-refresh limitation in the absence of middleware. This must be resolved/verified during implementation planning before lock.

## 17. Service-role boundary — DRAFT

Supabase `service_role` credentials are server-only.

They must never be exposed to browser/client bundles.

Any elevated operation used for identity/verification/authentication must remain behind a server-side trust boundary.

The final implementation must minimize elevated operations and must not use service-role access as a substitute for application authorization.

Node 1 authorization remains authoritative for business-resource authorization.

## 18. Authentication failure behavior — DRAFT

Authentication failures should use generic externally visible responses where revealing whether a credential/account/identity exists would create enumeration risk.

Unauthenticated protected requests must be rejected before protected business operations execute.

Wrong-role requests must not fall through to the other role's business logic.

Exact HTTP status/error contracts remain to be finalized before implementation lock.

## 19. Role-specific access — DRAFT

The system must distinguish:

```text
Company user → Company-authorized operations
Driver user  → Driver-authorized operations
```

A Driver must not obtain Company authorization merely by supplying Company identifiers.

A Company must not obtain Driver authorization merely by supplying Driver identifiers.

A PENDING or REJECTED identity must not obtain either trusted role through client-controlled fields.

The authenticated role/identity context must be server-derived.

Detailed business-resource authorization remains Node 1's responsibility.

## 20. Testing contract — DRAFT

Before Node 2 can be accepted, tests/evidence must demonstrate at minimum:

### Identity invariants

- one Auth User cannot create two Freight identities;
- one Auth User cannot hold both Company and Driver trusted roles;
- every Auth User has exactly one Freight identity after successful signup transaction;
- trusted role is server-controlled;
- DB-level uniqueness on `freight_identities.auth_user_id` rejects duplicate identity rows for one Auth User.

### Verification

- requested role alone cannot authorize the user;
- PENDING cannot perform protected Company/Driver operations;
- user cannot modify verification status;
- user cannot modify trusted role;
- only authorized verifier action can change PENDING → VERIFIED/REJECTED;
- approved Driver evidence produces trusted Driver role;
- approved Company GST evidence produces trusted Company role;
- rejected verification cannot gain trusted role access.

### Signup / onboarding consistency

- Auth User and generic Freight Identity are created atomically under the selected mechanism;
- identity creation failure does not leave a durable Auth User without Freight Identity;
- duplicate/concurrent identity creation cannot create two identities for one Auth User;
- verification state cannot bypass email/authorization requirements.

### Authentication

- valid authentication succeeds;
- invalid credentials fail generically;
- unauthenticated protected requests fail;
- expired/invalid sessions fail safely;
- logout invalidates the expected session behavior.

### Role enforcement

- Company cannot access Driver-only operations;
- Driver cannot access Company-only operations;
- client-supplied role cannot escalate privileges;
- PENDING/REJECTED cannot access trusted role operations.

### Identity handoff

- authenticated request resolves the correct Freight Identity;
- client-supplied identity cannot replace authenticated identity;
- Node 1 authorization receives trusted identity context.

## 21. Open decisions before contract lock

The following remain open and must be resolved before the complete Node 2 contract can become LOCKED:

1. Exact Freight Identity database representation beyond the now-required unique Auth User reference.
2. Exact Company and Driver role-specific mapping after verification.
3. Exact trigger implementation/security/error handling, while the atomic-trigger architecture and DB uniqueness requirement are decided.
4. Exact email-confirmation policy and state ordering.
5. Exact verifier authorization mechanism and audit trail.
6. Exact evidence validation checklist for Driver licence and Company GST evidence.
7. Exact session refresh mechanism.
8. Exact application-level authentication rate-limiting policy.
9. Exact authentication failure status/error contract.
10. Exact server-side identity-context helper/interface.
11. Exact Node 2 automated acceptance-test set.
12. Exact document/evidence retention and access policy.

## 22. Non-goals

Node 2 does not redefine:

- trip ownership;
- Driver marketplace behavior;
- atomic claim behavior;
- delivery lifecycle;
- evidence timeline;
- AI evidence-grounded summary;
- Node 1 resource authorization rules.

Those remain governed by the existing roadmap and locked records.

## 23. Status

```text
Q1 Signup consistency                        = DECIDED / LOCKED FOR NODE 2
Q4 One-user → one-identity enforcement       = DECIDED / LOCKED FOR NODE 2
Q4 DB uniqueness requirement                 = DECIDED
Pending identity vs verification investigation = COMPLETE
Leading identity model                       = MODEL C / PROPOSED
Verification model                           = HACKATHON MANUAL REVIEW / PROPOSED
Contract design                              = DRAFT
Independent Q1/Q4 review                     = COMPLETE
Ayush approval for Q1/Q4                     = APPROVED
Remaining Node 2 blocking questions          = Q2, Q3, Q5, Q6, Q7
Complete Node 2 contract lock                = NOT YET
Implementation                                = PAUSED
```

This document must not be treated as full Node 2 implementation authorization until the remaining blocking decisions are resolved and the complete contract is explicitly locked/approved.
