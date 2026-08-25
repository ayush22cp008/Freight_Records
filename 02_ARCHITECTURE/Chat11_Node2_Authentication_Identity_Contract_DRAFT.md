# Chat12 — Node 2 Authentication + Identity Contract — DRAFT

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Status:** DRAFT / NOT LOCKED / REQUIRES AYUSH REVIEW  
**Chat:** Chat12  

## 1. Purpose

Define the authentication and application-identity contract required before authentication implementation resumes.

This draft is derived from the locked Node 1 product/authorization model and the Chat11/Chat12 Node 2 investigation records.

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
- A PostgreSQL `auth.users` trigger is technically available and can create application identity within the Auth User insertion transaction.

These are evidence findings, except where explicitly marked as a proposed contract decision below.

## 4. Contract goals

Node 2 must establish:

1. A single trustworthy application identity for each authenticated user.
2. Exactly one application role for each authenticated user.
3. Explicit Company vs Driver identity semantics.
4. Server-derived identity context for protected requests.
5. No authorization trust in client-supplied identity or role fields.
6. Authentication behavior that can safely feed Node 1 authorization.
7. A defined signup/onboarding consistency model.
8. A defined email-confirmation and account-activation model.
9. A defined session lifecycle.
10. A defined authentication abuse/rate-limiting boundary.
11. Testable acceptance criteria.

## 5. Proposed identity model — DRAFT

The conceptual model should be:

```text
Supabase Auth User
        │
        │ exactly one
        ▼
Application Identity
        │
        ├── role = Company
        │       └── Company identity
        │
        └── role = Driver
                └── Driver identity
```

The application identity must be derived from the authenticated Supabase Auth user, not from a client-provided user/driver/company ID.

The exact database representation (single identity table, role-specific mapping, or another design) remains a decision to be finalized before lock. This draft intentionally does not prescribe an unreviewed schema.

## 6. One-user / one-identity invariant — DRAFT

The system must enforce, at the application/database boundary as appropriate:

```text
ONE auth.users.id
        ↓
ONE application identity
        ↓
ONE role
```

A single Auth User must not simultaneously become both a Company and Driver identity.

A single Auth User must not own multiple application identities.

The enforcement mechanism must be explicit and testable.

## 7. Role model — DRAFT

Allowed application roles:

```text
Company
Driver
```

Role must be server-trusted application state.

The client must not be able to select an arbitrary role on an authenticated request and thereby gain that role's authorization.

Role enforcement must occur before protected role-specific business operations.

## 8. Signup / onboarding consistency — PROPOSED DECISION, NOT YET LOCKED

The current sequential signup flow is not an acceptable final consistency boundary because it can leave:

```text
Auth User exists
BUT
required application identity does not exist
```

The preferred architecture decision from Chat12 investigation is **PostgreSQL-trigger-based atomic application-identity creation**.

Conceptually:

```text
Supabase Auth User creation
        ↓
PostgreSQL transaction
        ↓
auth.users INSERT trigger
        ↓
Application Identity creation
        ↓
Both succeed → durable consistent state

OR

Identity creation fails
        ↓
Transaction rolls back
        ↓
No durable Auth User without identity
```

The trigger is a proposed identity-consistency mechanism. It is not yet implementation authorization.

Server-side compensation is not selected as the primary consistency mechanism because unknown-outcome timeouts can cause Auth deletion while leaving a Driver record through the current `ON DELETE SET NULL` relationship.

The final implementation must preserve the Node 1 invariant:

```text
1 Auth User ↔ exactly 1 application identity
```

## 9. Email confirmation and account activation — DRAFT / OPEN POLICY DECISION

Identity existence must not be treated as equivalent to an active or authorized account.

The contract distinguishes:

```text
Identity exists
    ≠
Email confirmed
    ≠
Authorized
    ≠
Active/usable account
```

The proposed state model is:

```text
Auth User + Application Identity
        ↓
Email confirmation state
        ↓
Identity/role validity
        ↓
Protected application access only when required gates pass
```

The final contract must explicitly define:

- whether email confirmation is required;
- unconfirmed login behavior;
- whether identity is created before or after confirmation;
- when the account becomes active/usable;
- failure behavior for each state.

This remains OPEN until independently reviewed and approved.

## 10. Driver Code authentication — DRAFT

Driver Code is an identifier, not a secret.

The current sequential format (`DRV###`) is enumerable and therefore must not be treated as sufficient authentication protection by itself.

The authentication contract therefore requires:

- password/authentication credential remains the secret factor;
- Driver Code must not be treated as a secret;
- brute-force protection must be explicitly defined;
- generic authentication failures should avoid revealing whether a Driver Code exists.

The final rate-limiting design remains an OPEN decision in this draft.

## 11. Authentication rate limiting — DRAFT

Authentication abuse protection must exist at the application/security boundary appropriate to the chosen implementation.

The final contract must define at least:

- per-IP controls;
- per-account/credential identifier controls;
- shared state where required by deployment architecture;
- behavior when limiter state is unavailable;
- ordering relative to identity lookup;
- failure response behavior.

The current absence of application-level rate limiting is a verified gap, not an approved final architecture.

## 12. Authenticated request context — DRAFT

A protected server request must derive identity from the verified authentication session.

Conceptually:

```text
Request
  ↓
Verified Supabase Auth session/user
  ↓
Application identity lookup
  ↓
Role + identity context
  ↓
Node 1 authorization
  ↓
Business operation
```

Client-supplied `driver_id`, `company_id`, or `role` may be input data but must never be accepted as proof of the caller's identity or authorization scope.

The exact helper/context API remains an implementation detail to be defined after this contract is approved.

## 13. Session lifecycle — DRAFT

The authentication contract must define:

- session establishment;
- server-side verification;
- refresh behavior;
- logout behavior;
- expired/invalid session behavior;
- cookie/token handling;
- protected-route behavior during refresh failure.

The current investigation identified a possible session-refresh limitation in the absence of middleware. This must be resolved/verified during implementation planning before lock.

## 14. Service-role boundary — DRAFT

Supabase `service_role` credentials are server-only.

They must never be exposed to browser/client bundles.

Any elevated operation used for identity/authentication must remain behind a server-side trust boundary.

The final implementation must minimize elevated operations and must not use service-role access as a substitute for application authorization.

Node 1 authorization remains authoritative for business-resource authorization.

## 15. Authentication failure behavior — DRAFT

Authentication failures should use generic externally visible responses where revealing whether a credential/account/identity exists would create enumeration risk.

Unauthenticated protected requests must be rejected before protected business operations execute.

Wrong-role requests must not fall through to the other role's business logic.

Exact HTTP status/error contracts remain to be finalized before implementation lock.

## 16. Role-specific access — DRAFT

The system must distinguish:

```text
Company user → Company-authorized operations
Driver user  → Driver-authorized operations
```

A Driver must not obtain Company authorization merely by supplying Company identifiers.

A Company must not obtain Driver authorization merely by supplying Driver identifiers.

The authenticated role/identity context must be server-derived.

Detailed business-resource authorization remains Node 1's responsibility.

## 17. Testing contract — DRAFT

Before Node 2 can be accepted, tests/evidence must demonstrate at minimum:

### Identity invariants

- one Auth User cannot create two application identities;
- one Auth User cannot hold both Company and Driver roles;
- every active application identity maps to exactly one Auth User;
- role is server-trusted.

### Signup / onboarding consistency

- Auth User and required application identity are created atomically under the selected mechanism;
- application identity creation failure does not leave a durable Auth User without identity;
- duplicate/concurrent identity creation cannot create two identities for one Auth User;
- email-confirmation state does not bypass required authorization gates.

### Authentication

- valid authentication succeeds;
- invalid credentials fail generically;
- unauthenticated protected requests fail;
- expired/invalid sessions fail safely;
- logout invalidates the expected session behavior.

### Role enforcement

- Company cannot access Driver-only operations;
- Driver cannot access Company-only operations;
- client-supplied role cannot escalate privileges.

### Identity handoff

- authenticated request resolves the correct application identity;
- client-supplied identity cannot replace authenticated identity;
- Node 1 authorization receives trusted identity context.

### Abuse protection

- authentication rate limiting behaves according to the final selected policy;
- enumeration-sensitive responses remain generic.

### Security boundary

- service-role credentials are server-only;
- protected APIs do not trust client identity fields as authorization proof.

## 18. Open decisions before contract lock

The following must be explicitly decided/reviewed before this document can become LOCKED:

1. Exact application identity database representation.
2. Exact Company and Driver identity mapping.
3. Final approval of PostgreSQL-trigger-based atomic signup identity creation.
4. Exact email-confirmation policy and activation-state semantics.
5. Exact session refresh mechanism.
6. Exact application-level authentication rate-limiting policy.
7. Exact authentication failure status/error contract.
8. Exact server-side identity-context helper/interface.
9. Exact role enforcement boundary.
10. Exact Node 2 automated acceptance-test set.

## 19. Non-goals

Node 2 does not redefine:

- trip ownership;
- Driver marketplace behavior;
- atomic claim behavior;
- delivery lifecycle;
- evidence timeline;
- AI evidence-grounded summary;
- Node 1 resource authorization rules.

Those remain governed by the existing roadmap and locked records.

## 20. Status

```text
Signup consistency investigation = COMPLETE
Signup architecture decision      = PROPOSED
Contract design                   = DRAFT
Contract lock                     = NOT YET
Ayush approval                    = REQUIRED
Implementation                    = PAUSED
```

This document must not be treated as an implementation authorization until explicitly locked/approved.
