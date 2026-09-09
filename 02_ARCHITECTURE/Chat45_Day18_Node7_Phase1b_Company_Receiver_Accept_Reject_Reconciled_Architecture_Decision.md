# Chat45 — Day 18 — Node 7 Phase 1b — Company Receiver Accept/Reject — Reconciled Architecture Decision

## 1. Record Status

**Status:** RECONCILED ARCHITECTURE — AYUSH FINAL APPROVAL REQUIRED

**Decision owner:** ChatGPT (architecture/reasoning)

**Final authority:** Ayush

**Execution agent:** Antigravity

**Implementation authorization:** NOT GRANTED

**Company Portal:** UNLOCKED

This record reconciles the following inputs:

1. Receiver Accept/Reject delivery investigation.
2. Future architecture package.
3. Claude peer architecture review.
4. Core state-machine/concurrency investigation.
5. Source and schema investigation.

The purpose is to produce one coherent architecture decision candidate before implementation.

---

## 2. Final Product Decision Being Proposed

For inter-company deliveries, the Receiving Company must explicitly decide:

```text
ACCEPT
or
REJECT
```

before the delivery becomes eligible for normal Driver marketplace execution.

The existing operational delivery lifecycle remains intact after the agreement gate is satisfied.

### Target product flow

```text
Sender creates Trip
        ↓
Receiver delivery request = PENDING
        ↓
Receiver decision
   ┌────┴────┐
   ↓         ↓
 ACCEPT    REJECT
   ↓         ↓
Eligible    Terminal non-claimable outcome
for normal  + sender-visible result
publication
   ↓
Driver marketplace
   ↓
Existing Driver claim
   ↓
Existing delivery lifecycle
   ↓
Completed
```

**Decision status:** PROPOSED — requires Ayush final approval.

---

## 3. Architecture Reconciliation: Request Layer vs Trip Status

The source investigation verified an important current fact:

```text
Current trips.status values:
active, draft, published, claimed, in_progress, completed
```

There are currently no `pending`, `accepted`, or `rejected` Trip status values.

The source investigation also observed that implementing the feature will require a blocking pre-publication condition and a terminal rejected outcome.

However, the earlier architecture package and Claude review recommend separating **agreement semantics** from the operational Trip lifecycle.

### Reconciled decision

Use a **persistent Receiver Delivery Request / handshake entity referencing the existing Trip**, while keeping the existing Trip status model authoritative for operational execution.

```text
Receiver Request
    ├── PENDING
    ├── ACCEPTED
    └── REJECTED

Existing Trip
    ├── draft
    ├── published
    ├── claimed
    ├── in_progress
    └── completed
```

The request state becomes an explicit marketplace eligibility prerequisite rather than replacing the operational Trip state machine.

### Why this resolves the apparent conflict

The current database does need a new persistent business rule, but it does **not necessarily require agreement semantics to be encoded directly as Trip operational statuses**.

The feature therefore changes the authoritative precondition for publication/claiming while preserving the existing Trip lifecycle after acceptance.

**Status:** RECOMMENDED ARCHITECTURE DECISION.

---

## 4. Canonical State Relationship

The architecture should be understood as two linked state machines.

### Receiver agreement state

```text
PENDING
   ├── ACCEPTED
   └── REJECTED
```

### Trip operational state

```text
DRAFT
   ↓
PUBLISHED
   ↓
CLAIMED
   ↓
IN_PROGRESS
   ↓
COMPLETED
```

### Required relationship

For an inter-company Trip:

```text
Receiver Request = PENDING
        ↓
Trip cannot enter Driver-claimable marketplace state
```

```text
Receiver Request = REJECTED
        ↓
Trip cannot enter Driver-claimable marketplace state through that request
```

```text
Receiver Request = ACCEPTED
        ↓
Trip may proceed through normal publication / marketplace rules
```

Acceptance therefore **unlocks** the existing operational path rather than creating a second operational lifecycle.

---

## 5. Accept / Publish Ordering

### Reconciled decision

**Receiver acceptance must occur before normal Driver marketplace eligibility.**

The sender may create the Trip and the Receiver request while the Trip remains non-marketplace-eligible.

Conceptually:

```text
Create Trip
    ↓
Create Receiver Request
    ↓
PENDING
    ↓
WAITING FOR RECEIVER
    ↓
Receiver ACCEPT
    ↓
Sender Publish permitted
    ↓
PUBLISHED / marketplace-eligible
```

For rejection:

```text
PENDING
    ↓
REJECTED
    ↓
No Driver marketplace eligibility
```

### Server invariant

The Publish API must enforce the Receiver agreement requirement server-side.

The Claim API must also preserve a server-side guard so a malicious or legacy path cannot bypass the agreement requirement.

### UI consequence

The exact location/enablement state of the Publish button is a UI decision, but the backend must remain authoritative.

**Status:** RECOMMENDED — requires Ayush approval.

---

## 6. Marketplace Eligibility Rule

The canonical rule is:

```text
Driver may claim Trip
ONLY IF
existing Trip claim preconditions are satisfied
AND
Receiver agreement requirement is satisfied.
```

For inter-company delivery:

| Receiver Request | Marketplace eligibility |
|---|---|
| PENDING | DENIED |
| REJECTED | DENIED |
| ACCEPTED | ALLOWED subject to existing publication/claim rules |

The implementation must enforce this in server/API paths, not by frontend filtering.

### Required regression protection

The existing atomic Driver claim condition remains authoritative after the Trip becomes eligible.

The new agreement gate is an additional precondition, not a replacement for the claim transaction.

**Status:** REQUIRED ARCHITECTURE DECISION.

---

## 7. Concurrency / Atomicity

### Reconciled decision

A Receiver Accept or Reject must be implemented as an atomic state transition from PENDING.

Conceptual operation:

```text
UPDATE receiver_request
SET state = <ACCEPTED or REJECTED>,
    decided_at = <timestamp>,
    decided_by = <authenticated company/user>
WHERE id = <request_id>
  AND state = 'PENDING'
```

Exactly one valid terminal decision may win.

```text
affected rows = 1
    → transition succeeded

affected rows = 0
    → request was already transitioned / invalid
```

A database transaction/row-lock strategy is acceptable if it provides the same invariant.

### Required invariant

```text
PENDING → one authoritative terminal decision
```

The UI button state must never be treated as the concurrency mechanism.

**Status:** RECOMMENDED / IMPLEMENTATION-READY PRINCIPLE.

---

## 8. Duplicate Request Protection

### Reconciled decision

At most one active PENDING Receiver request may exist for a Trip.

A database-level uniqueness mechanism should enforce this wherever supported by the actual schema/database platform.

Conceptually:

```text
Trip X
  └── one active PENDING request maximum
```

Terminal historical requests may remain stored.

Example:

```text
Request 1 → REJECTED
Request 2 → PENDING
```

This is preferred over reopening Request 1.

**Status:** RECOMMENDED — exact migration/index syntax remains implementation detail.

---

## 9. Resend / New Request Identity

### Reconciled decision

A resend after a terminal request outcome should normally create a **new request identity**.

```text
Request 1 → REJECTED
             ↓
        preserved audit record
             ↓
Request 2 → PENDING
```

The old request is not mutated back to PENDING.

### Default policy candidate

```text
PENDING   → duplicate request denied
REJECTED  → new request may be created, subject to sender policy
CANCELLED → new request may be created, subject to sender policy
ACCEPTED  → no duplicate agreement request for same execution
```

Exact sender permission to resend remains part of the API/UX contract.

**Status:** RECOMMENDED — Ayush approval required.

---

## 10. Sender Cancellation

Current source investigation verified that there is no sender Trip cancellation API today.

Therefore the new feature should not silently invent a broad cancellation system outside the approved scope.

### Reconciled v1 decision

Until a sender cancellation capability is explicitly designed and authorized:

```text
No new general sender-cancellation workflow is introduced
```

The architecture must nevertheless prevent a receiver from accepting a request that has been invalidated by another authoritative event.

If sender cancellation becomes required, it should create a distinct terminal request outcome rather than reopening or converting a request to ACCEPTED.

**Status:** DEFERRED / SCOPE CONTROL.

---

## 11. Trip Mutation While PENDING

A material change to the delivery requested from the Receiver can invalidate the Receiver's prior context.

### Reconciled decision

A material change while the request is PENDING must require a fresh Receiver decision.

At minimum:

- Receiving Company change.
- Trip cancellation/invalidation if such capability is later added.
- Material destination change.
- Material commercial change such as payout, once confirmed as part of the agreement contract.

The exact material-field allowlist requires implementation-stage source/schema review and must not be guessed.

### Required invariant

```text
Receiver accepts request A
        ↓
Sender materially changes request facts
        ↓
Original acceptance cannot silently remain valid for changed facts
```

**Status:** RECOMMENDED.

---

## 12. Rejection Reason

### Reconciled decision

Use an **optional rejection reason** for v1.

Reasoning from the architecture investigation:

- Receiver can reject without being blocked by form completion.
- Operational feedback remains possible.
- Data/API/UI can support a nullable reason.

The reason, when supplied, must be retained with the request decision.

**Status:** RECOMMENDED — Ayush approval required.

---

## 13. Expiration / Timeout

### Reconciled v1 decision

Do **not** introduce automatic request expiration in the initial implementation.

Therefore the v1 request remains:

```text
PENDING
until
ACCEPT / REJECT
or another explicitly authorized terminal invalidation event
```

This avoids introducing another concurrency actor, scheduler dependency, terminal state, and resend complexity for the first implementation.

Expiration can be designed later as a separate architecture change.

**Status:** RECOMMENDED DEFERMENT.

---

## 14. Same Company as Sender and Receiver

For a Trip where the authoritative Trip relationship is:

```text
sending_company_id == receiving_company_id
```

### Reconciled decision candidate

Do not require an external Company-to-Company handshake for the same Company.

The condition must be derived server-side from authoritative Trip data and authenticated Company identity.

The client must never be able to manufacture the bypass by submitting arbitrary company IDs.

**Status:** RECOMMENDED — Ayush approval required.

---

## 15. Migration / Existing Trips

The source investigation established that existing Trips already use the current operational statuses and that changing their eligibility unexpectedly could break active operations.

### Reconciled migration principle

Existing operational Trips must remain operational during rollout.

Therefore legacy Trips that were already operational before the new handshake feature should be treated as **legacy-exempt / implicitly agreement-satisfied for marketplace purposes**, rather than suddenly becoming blocked because no Receiver Request row exists.

This applies to the compatibility set identified by the source investigation:

```text
published
claimed
in_progress
completed
```

Whether `draft` trips need a synthetic request depends on their actual current semantics and must be decided during implementation migration planning.

### Important distinction

“Implicitly accepted” is a compatibility interpretation, not a statement that an actual historical Receiver acceptance occurred.

Audit/history must not fabricate a false Receiver action.

**Status:** RECOMMENDED migration strategy; exact SQL/backfill/exemption implementation remains to be designed.

---

## 16. Data Model Decision

### Preferred logical model

```text
receiver_delivery_requests
--------------------------------
request_id
trip_id
sending_company_id
receiving_company_id
state
created_at
decided_at
decided_by
rejection_reason (nullable)
```

Additional lifecycle/audit fields may be required after implementation schema inspection.

### Required integrity properties

- Request references one existing Trip.
- Sender/receiver identities are derived from the authoritative Trip relationship.
- State transitions are server-controlled.
- One active PENDING request maximum per Trip.
- Terminal requests remain auditable.
- Client-supplied company identity cannot override request relationships.

**Status:** LOGICAL ARCHITECTURE, not physical schema lock.

---

## 17. API / Server Boundary

The feature requires explicit server contracts for at least:

```text
Create Receiver Request
Read Receiver Pending Requests
Receiver Accept
Receiver Reject
Sender Request Status
```

Potential future contracts such as sender resend/cancel are controlled by the decisions above.

### Accept / Reject authorization

```text
Authenticated Company
        ↓
Trip.receiving_company_id
        ↓
Exact match required
```

Sending Company, Driver, and unrelated Companies must not be allowed to mutate the request.

### Claim authorization

The Driver Claim path must additionally verify that the Receiver agreement requirement is satisfied for Trips governed by the new handshake.

**Status:** REQUIRED ARCHITECTURE DECISION.

---

## 18. RLS / Security Decision

The source investigation verified that the current Trips table has RLS enabled but no client-side policies, with access currently enforced through server-side application logic using `supabaseServer` and service-role access.

The new request layer must preserve this server-authoritative security model unless a future security review explicitly changes it.

### Required security invariants

```text
Receiver → may Accept / Reject own request
Sender   → may read own request outcome, no Accept / Reject
Driver   → no request mutation
Other    → no access
```

The final physical RLS policy design must be reconciled with the project's current service-role architecture before implementation.

**Status:** VERIFIED current architecture + REQUIRED future security work.

---

## 19. Company UI Architecture

### Incoming Deliveries remains the receiver action surface.

Current Incoming Deliveries behavior was source-verified to show receiving trips in:

```text
active
claimed
in_progress
```

The new feature should introduce a visually clear **Pending Receiver Requests** state/section within the same receiving-company workflow unless UI investigation proves a better location.

Conceptually:

```text
Incoming Deliveries

Pending Requests
    Trip A   [Accept] [Reject]
    Trip B   [Accept] [Reject]

Active Deliveries
    Trip C   In Transit
    Trip D   Completion Required
```

After acceptance, the Trip moves into the existing operational receiving flow.

After rejection, the Receiver sees the durable rejected outcome according to the final history/status design.

**Status:** RECOMMENDED UI direction; detailed UI is not yet locked.

---

## 20. Driver Portal Impact

The Driver Portal must continue to expose only claimable marketplace Trips.

### Required behavior

```text
Pending Receiver Request
    ↓
Driver must NOT see as claimable

Rejected Receiver Request
    ↓
Driver must NOT see as claimable

Accepted Receiver Request
    ↓
Trip may appear through normal marketplace eligibility
```

No redesign of the Driver Portal should occur unless source investigation shows a necessary UI/API adaptation.

The essential change is the server-side marketplace gate.

**Status:** REQUIRED integration decision.

---

## 21. Existing Workflow Protection

The following remain protected:

- Driver atomic claim.
- Existing publication/marketplace behavior after eligibility is established.
- Receiver Check-in.
- Receiver Completion.
- Driver Completion.
- Completion ordering.
- Company Sender/Receiver History visibility.
- Unified Company Trip Detail.
- Existing authentication and identity rules.
- Existing relationship semantics.

Any deviation must be handled as a separately reviewed architecture change.

---

## 22. Final Acceptance Criteria for Implementation

Implementation cannot be considered complete until all are demonstrated.

### Receiver agreement

1. Correct Receiver sees pending request.
2. Unauthorized Company cannot Accept/Reject.
3. Accept creates one authoritative ACCEPTED outcome.
4. Reject creates one authoritative REJECTED outcome.
5. Concurrent conflicting decisions cannot produce multiple authoritative outcomes.

### Marketplace gate

6. PENDING cannot be Driver-claimed.
7. REJECTED cannot be Driver-claimed.
8. ACCEPTED can proceed through approved Publish/Marketplace flow.
9. Publish and Claim are protected server-side.

### Sender visibility

10. Sender can distinguish PENDING / ACCEPTED / REJECTED according to the final UI contract.
11. Rejection remains durable and auditable.
12. Optional rejection reason is preserved when supplied.

### Existing lifecycle

13. Accepted Trip can proceed through existing Driver claim.
14. Receiver Check-in still works.
15. Receiver Completion still works.
16. Driver Completion still works.
17. Company History remains correct.
18. Existing Sender/Receiver relationship visibility remains correct.

### Security / integrity

19. Cross-company request mutation is rejected.
20. Client-supplied identity cannot bypass authorization.
21. One active PENDING request per Trip is enforced.
22. Existing active/legacy Trips remain operational during migration.
23. No false historical Receiver acceptance event is fabricated for legacy data.

### Manual E2E

24. Ayush verifies sender creates request.
25. Ayush verifies Receiver Accept.
26. Ayush verifies Receiver Reject.
27. Ayush verifies pending/rejected marketplace blocking.
28. Ayush verifies accepted marketplace entry and Driver claim.
29. Ayush verifies rejection visibility.
30. Ayush verifies normal completion after acceptance.

---

## 23. Implementation Boundary

This is a **controlled architecture expansion**.

Expected implementation areas:

```text
Database / migration
Receiver request data model
Server/API contracts
Authorization / security
Publish gate
Driver Claim gate
Company UI
Driver impact if required
Automated tests
Manual E2E verification
```

ChatGPT must not implement source code directly.

Antigravity implementation is authorized only after Ayush approves this architecture record.

---

## 24. Reconciled Decision Matrix

| Area | Reconciled decision | Status |
|---|---|---|
| Product concept | Receiver Accept/Reject required before marketplace execution | PROPOSED |
| Request model | Separate persistent request referencing existing Trip | RECOMMENDED |
| Operational Trip lifecycle | Preserve existing lifecycle | LOCK-PRESERVATION |
| Accept before marketplace eligibility | Yes | RECOMMENDED |
| Server marketplace gate | Required | REQUIRED |
| Atomic decision | Conditional/transactional PENDING transition | RECOMMENDED |
| One active PENDING request | DB-level enforcement preferred | RECOMMENDED |
| Resend | New request identity | RECOMMENDED |
| Sender cancellation | Deferred; no new cancellation system in v1 | DEFERRED |
| Material Trip mutation | Requires fresh Receiver decision | RECOMMENDED |
| Rejection reason | Optional | RECOMMENDED |
| Expiration | Defer from v1 | RECOMMENDED |
| Same-company sender=receiver | No external handshake, server-derived | RECOMMENDED |
| Legacy active Trips | Preserve operational behavior via legacy exemption/implicit agreement satisfaction | RECOMMENDED |
| Driver Portal | Preserve UI; enforce marketplace gate server-side | REQUIRED |
| Company UI | Pending requests in Receiver action surface | RECOMMENDED |

---

## 25. Items Still Requiring Final Implementation-Stage Verification

The architecture is substantially reconciled, but the following physical implementation details must still be verified before the Antigravity handoff:

1. Exact database migration/index syntax.
2. Exact source locations for request creation and current Company Trip Publish UI/API.
3. All Driver marketplace query paths, including any non-obvious claim entry path.
4. Exact existing Trip RLS/service-role boundaries.
5. Exact current Trip fields whose mutation must invalidate PENDING.
6. Exact legacy Trip exemption/backfill mechanism.
7. Exact UI component placement and status copy.
8. Exact API response/error contract for conflicts and repeated decisions.
9. Exact audit representation and retention mechanism.

These are implementation details, not permission to begin coding.

---

## 26. Final Architecture Position

### Recommended final architecture

```text
             COMPANY / SENDER
                    │
                    ▼
              Existing Trip
                    │
                    ▼
        Receiver Delivery Request
                    │
                 PENDING
               ┌────┴────┐
               ▼         ▼
            ACCEPT     REJECT
               │         │
               ▼         ▼
          ACCEPTED    REJECTED
               │         │
               ▼         ▼
        Existing Publish   BLOCKED
        / Marketplace
               │
               ▼
         Driver Claim
               │
               ▼
       Existing Delivery
           Lifecycle
               │
               ▼
           COMPLETED
```

This preserves the existing operational Trip model while adding the missing business-consent gate required by the final Company product decision.

---

## 27. Governance Gate

Current state:

```text
PRODUCT REQUIREMENT: AGREED BY AYUSH
ARCHITECTURE: RECONCILED / PROPOSED
SOURCE INVESTIGATION: COMPLETE
CLAUDE PEER REVIEW: COMPLETE
IMPLEMENTATION: NOT YET AUTHORIZED
COMPANY: UNLOCKED
```

### Required next step

**Ayush final architecture approval.**

After explicit approval:

```text
Ayush approval
    ↓
Implementation handoff
    ↓
Antigravity preflight
    ↓
Implementation
    ↓
Build + automated tests
    ↓
Implementation report
    ↓
Ayush manual E2E verification
    ↓
Company ACCEPTED / LOCKED
```

No source implementation should begin before this approval gate is explicitly passed.
