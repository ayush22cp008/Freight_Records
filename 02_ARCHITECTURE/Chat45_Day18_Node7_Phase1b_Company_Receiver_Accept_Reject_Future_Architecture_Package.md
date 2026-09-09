# Chat45 — Day 18 — Node 7 Phase 1b — Company Receiver Accept/Reject Future Architecture Package

## 1. Record Status

**Status:** PROPOSED FUTURE ARCHITECTURE — PEER REVIEW / AYUSH APPROVAL REQUIRED

**Decision owner:** ChatGPT (architecture/reasoning)

**Final authority:** Ayush

**Execution agent:** Antigravity

**Implementation authorization:** NOT GRANTED

This package converts the earlier Receiver Accept/Reject architecture review into a structured future-implementation package. It does **not** authorize source-code, database, API, RLS, authentication, lifecycle, or marketplace implementation.

---

## 2. Product Decision Being Evaluated

The intended Company behavior is:

```text
Sending Company
    ↓
Creates delivery for Receiving Company
    ↓
Receiving Company receives a delivery request
    ↓
Receiver chooses
    ACCEPT
    OR
    REJECT
```

Accepted request:

```text
Receiver ACCEPT
    ↓
Delivery becomes eligible for normal publication / marketplace execution
    ↓
Driver claims
    ↓
Existing delivery lifecycle
    ↓
Receiver completion + Driver completion
    ↓
Completed
```

Rejected request:

```text
Receiver REJECT
    ↓
Request becomes terminally rejected
    ↓
Delivery is not Driver-claimable
    ↓
Sending Company sees durable rejected outcome
    ↓
Outcome remains auditable
```

### Product recommendation

**RECOMMENDED:** Make an explicit Receiver Accept/Reject decision a required agreement gate for inter-company delivery requests before those deliveries enter normal Driver marketplace execution.

This recommendation requires Ayush approval before implementation planning becomes authorized.

---

## 3. Existing Architecture That Must Remain Intact

The existing Freight model already establishes:

```text
One Company
→ many trips
→ trip-specific Sender/Receiver relationship
→ shared delivery visibility
→ relationship/state-based actions
```

The existing operational lifecycle remains the authoritative execution model after a delivery is accepted and eligible for normal publication:

```text
DRAFT
  ↓
PUBLISHED / AVAILABLE
  ↓
CLAIMED
  ↓
IN_PROGRESS
  ↓
COMPLETED
```

The current Company surfaces remain:

- **My Created Trips** = sender/creator active monitoring.
- **Incoming Deliveries** = receiver action inbox.
- **History** = company-wide completed participation history.
- **Trip Detail** = unified exact-trip detail surface.

The new handshake should sit **before operational marketplace execution**, not replace the existing Company receiving-trip tracking model.

---

## 4. Core Architectural Direction

### Preferred model

Introduce a persistent **delivery-request / handshake layer around the existing Trip lifecycle**.

Conceptually:

```text
Sender creates delivery
        ↓
Persistent Receiver Delivery Request
        ↓
PENDING receiver decision
        │
        ├───────────────┐
        ↓               ↓
     ACCEPT            REJECT
        ↓               ↓
Request accepted      Request rejected
        ↓               ↓
Existing publication  Terminal non-claimable outcome
/ marketplace path   + sender-visible result
        ↓
Existing Trip lifecycle
```

### Architectural principle

The receiver decision is a **business agreement gate**, not merely a UI notification.

Therefore the decision requires a persistent server-authoritative source of truth and server-enforced marketplace gating.

---

## 5. Proposed Request Object

The exact physical data model is not yet locked, but the future architecture should represent at minimum the following logical facts:

```text
request identity
trip identity
sending company identity
receiving company identity
decision state
created/requested timestamp
decision timestamp
decision actor
optional rejection reason
```

### Required relationship

The request must be tied to the exact existing Trip rather than creating a duplicate physical delivery/trip record.

Conceptually:

```text
Delivery Request
      ↓ references
Existing Trip
```

The request layer adds agreement semantics; the existing Trip remains the operational freight entity.

---

## 6. Proposed Request State Model

For architecture purposes the initial conceptual request states are:

```text
PENDING
ACCEPTED
REJECTED
```

These are **proposed logical states, not locked database enums**.

### State semantics

**PENDING**

- Receiver has not made an authoritative decision.
- Delivery must not be Driver-claimable.
- Receiver may decide according to the final authorization contract.

**ACCEPTED**

- Receiver has given the required agreement.
- Delivery may proceed into the existing publication/marketplace path according to the final contract.
- Existing Driver claiming and delivery lifecycle semantics remain authoritative.

**REJECTED**

- Receiver has declined the request.
- Delivery must not become Driver-claimable through this request.
- Sender receives durable rejection visibility.
- State is terminal unless a future resend/new-request rule explicitly creates a new request.

### Important separation

Do **not** overload existing operational Trip states merely to represent a Company agreement decision unless a later architecture review explicitly determines that approach is superior.

The preferred direction is a separate persistent request state model around the existing Trip lifecycle.

---

## 7. Publication / Marketplace Gate

This is the most important backend/business-rule consequence.

The future system must enforce:

```text
Request = PENDING
        ↓
NOT claimable

Request = REJECTED
        ↓
NOT claimable

Request = ACCEPTED
        ↓
Eligible for normal publication / marketplace path
```

The exact ordering with the sender's Publish action must be explicitly defined before implementation.

The implementation must never rely only on frontend hiding/filtering.

The server-side eligibility rule must be authoritative so that:

- an unaccepted request cannot be claimed directly;
- a rejected request cannot be claimed directly;
- a malicious client cannot bypass the receiver decision.

---

## 8. Proposed Sender Flow

Future conceptual sender flow:

```text
Sender creates trip
        ↓
Identifies Receiving Company
        ↓
Creates delivery request
        ↓
Request = PENDING
        ↓
Sender sees waiting-for-receiver-decision state
        ↓
Receiver ACCEPTS
        ↓
Trip proceeds to normal publication / marketplace path
```

For rejection:

```text
Sender creates request
        ↓
PENDING
        ↓
Receiver REJECTS
        ↓
REJECTED
        ↓
Sender sees durable rejection result
```

### Sender cancellation

The exact interaction between sender cancellation and PENDING must be resolved before implementation.

At minimum the architecture must answer:

```text
Can sender cancel while PENDING?
What happens to the request record?
Can a cancelled request later be accepted?
Can the sender resend?
Does resend create a new request identity?
```

No implementation assumption is made here.

---

## 9. Proposed Receiver Flow

Future conceptual Receiver Action Inbox:

```text
Incoming Delivery Requests

From: Sending Company
Trip: [safe trip identity / route]
Status: PENDING

[ Accept ]   [ Reject ]
```

After Accept:

```text
PENDING → ACCEPTED
```

After Reject:

```text
PENDING → REJECTED
```

The exact visual treatment and whether pending requests share the existing Incoming Deliveries page or receive a distinct request section must be decided during UI architecture work.

The current Incoming Deliveries surface should remain the receiver action surface unless a future design explicitly replaces or partitions it.

---

## 10. Authorization Model

The receiver decision must be relationship-bound and server-derived.

```text
Authenticated Company
        ↓
Compare identity with Trip.receiving_company_id
        ↓
Only matching Receiver may Accept / Reject
```

Required authorization expectations:

| Actor | Accept | Reject |
|---|---:|---:|
| Receiving Company | ALLOW | ALLOW |
| Sending Company | DENY | DENY |
| Assigned Driver | DENY | DENY |
| Other Company | DENY | DENY |

The client must not establish authority by supplying an arbitrary company ID.

The server must derive the authenticated Company identity and validate the trip/request relationship.

---

## 11. Rejection Reason

A rejection reason is **strongly recommended**, because sender visibility and operational follow-up benefit from knowing why a request was rejected.

However:

**Requirement status: NOT YET LOCKED.**

The future product decision must determine one of the following patterns:

```text
Reject with optional reason
```

or

```text
Reject requires reason
```

The final choice should be made before implementation and reflected consistently in data model, UI, API contract, and acceptance criteria.

---

## 12. Sender Notification / Discovery

The sender should have a durable way to discover the outcome:

```text
Delivery Request
→ REJECTED
→ receiver decision
→ timestamp
→ optional reason
```

The exact notification transport remains:

**OPEN / UNKNOWN**

Potential delivery channels include in-app visibility or another approved notification mechanism, but this architecture package does not select one without evidence or product approval.

The durable request state itself must remain authoritative regardless of notification transport.

---

## 13. Concurrency and Idempotency

The future design must support a single authoritative receiver decision.

Conceptually:

```text
PENDING
   ├── Accept
   └── Reject

First valid atomic transition wins
        ↓
Final state
        ↓
Later conflicting action
→ conflict / idempotent response according to final contract
```

The final implementation must not depend on button disable state or local component state for concurrency control.

A server-side atomic condition should guarantee that two concurrent receiver decisions cannot both become authoritative.

The final architecture must also define interaction with concurrent sender cancellation, expiration, or resend.

---

## 14. Duplicate Requests / Resend

Before implementation, resolve:

```text
Can sender create more than one pending request for the same Trip?
Can sender resend after rejection?
Does resend reuse the same request or create a new request?
What prevents duplicate pending requests?
What does receiver see if duplicate requests exist?
```

Preferred principle:

**Avoid multiple simultaneous active requests for the same Trip unless a future business rule explicitly requires that behavior.**

The exact rule remains open for approval.

---

## 15. Expiration / Timeout

The architecture must decide whether a PENDING request can remain indefinitely or expires after a defined period.

Questions to resolve:

```text
Is expiration required?
What triggers it?
What is the terminal state?
Can the sender resend after expiration?
Can the receiver accept after expiration?
```

No expiration state is locked by this package.

---

## 16. Same Company as Sender and Receiver

Existing architecture treats Sender/Receiver as trip-specific relationships for the same Company identity.

Recommended compatibility behavior:

```text
Same Company = Sender + Receiver
        ↓
Do not require a duplicate external handshake
```

This is **INFERRED / RECOMMENDED**, not yet a locked business rule.

The final architecture review must explicitly approve or reject this behavior before implementation.

---

## 17. Audit / History Requirements

The request layer should preserve enough information for an auditable decision history.

At minimum:

```text
request created
receiver decision
decision timestamp
decision actor
final request state
optional rejection reason
```

The existing Trip Timeline should not be overloaded with request semantics unless the final architecture explicitly defines how Company-to-Company request events are represented.

The relationship between request history and existing Company History must also be defined.

---

## 18. Existing Receiving-Trip Tracking Must Remain

The handshake does **not** replace the existing receiving-company tracking.

Future conceptual flow:

```text
Delivery Request
       ↓
Receiver decision
       ↓
ACCEPTED
       ↓
Existing receiving-company relationship
       ↓
Incoming Deliveries
       ↓
Driver claims / delivery progression
       ↓
Receiver Check-in
       ↓
Receiver Completion
       ↓
Completed
       ↓
History / Trip Detail
```

The currently working Company receiving-trip tracking remains the foundation after acceptance.

---

## 19. Protected Existing Semantics

The following existing behaviors must not be changed casually while adding the handshake:

- Driver marketplace atomic claim behavior.
- Existing delivery lifecycle semantics after marketplace entry.
- Receiver Check-in.
- Receiver Completion.
- Driver Completion.
- Existing completion ordering.
- Company History participation visibility.
- Unified Company Trip Detail.
- Existing sender/receiver relationship semantics.
- RLS and server-side identity/authorization principles.

Any required change to these is an architecture-level delta and must be separately reviewed.

---

## 20. Required Data / API / Security Design Work

Before source implementation, the future architecture must lock:

### Data

- request table/object shape;
- relationship to Trip;
- request state representation;
- timestamps;
- decision actor;
- optional rejection reason;
- uniqueness constraints preventing unwanted duplicate active requests;
- indexes needed for receiver inbox and sender visibility;
- retention/audit expectations.

### API

- create request contract;
- fetch pending requests contract;
- Accept contract;
- Reject contract;
- sender status/visibility contract;
- cancellation/resend/expiration contracts if applicable;
- idempotency/conflict behavior;
- authorization failure semantics.

### RLS / Security

- receiver-only Accept/Reject policy;
- sender read visibility;
- prevention of cross-company request access;
- prevention of unauthorized request mutation;
- marketplace non-claimability enforcement;
- server-derived authenticated identity.

### Migration / Backfill

The architecture must determine whether existing Trips require backfilled request records.

No migration strategy is locked by this package.

---

## 21. Acceptance Criteria — Future Implementation

The future implementation should not be accepted until all of the following are demonstrated.

### Receiver

1. Correct Receiving Company sees a PENDING request.
2. Unauthorized Company cannot Accept or Reject.
3. Receiver Accept produces one authoritative ACCEPTED outcome.
4. Receiver Reject produces one authoritative REJECTED outcome.
5. Duplicate/conflicting receiver actions cannot produce two final outcomes.

### Marketplace

6. PENDING requests are not Driver-claimable.
7. REJECTED requests are not Driver-claimable.
8. ACCEPTED requests enter the approved publication/marketplace path.
9. Marketplace enforcement remains server-authoritative.

### Sender

10. Sender can see whether the request is pending, accepted, or rejected according to the final contract.
11. Rejection remains durably discoverable.
12. Any rejection reason is shown according to the approved requirement.

### Existing lifecycle

13. Once accepted and published, existing Driver claiming continues correctly.
14. Existing Receiver Check-in and Receiver Completion remain correct.
15. Existing Driver completion remains correct.
16. Existing completion history remains correct.
17. Existing Company Sender/Receiver History visibility remains correct.

### Security / integrity

18. Cross-company request reads and writes are rejected.
19. Client-supplied identity cannot bypass authorization.
20. Concurrent decisions remain atomic.
21. Existing RLS/security boundaries remain valid.

### Manual verification

22. Ayush verifies sender request creation.
23. Ayush verifies receiver Accept.
24. Ayush verifies receiver Reject.
25. Ayush verifies Driver marketplace gating for pending/rejected/accepted cases.
26. Ayush verifies sender visibility of rejection.
27. Ayush verifies normal delivery completion after acceptance.

---

## 22. Implementation Boundary

This future feature is **not part of ordinary frontend-only Phase 1b work** because it necessarily introduces persistent business state and marketplace eligibility rules.

Expected implementation classes, once authorized, include potentially:

```text
Database/schema
API/contracts
RLS/security
Business rules
Marketplace gating
Company UI
Driver UI impact if required
Automated tests
Manual E2E verification
```

Therefore implementation must be treated as a controlled architecture expansion, not a small UI patch.

---

## 23. Governance Decision

### Current decision

```text
PRODUCT CONCEPT: RECOMMENDED
FUTURE ARCHITECTURE: PROPOSED
IMPLEMENTATION: NOT YET AUTHORIZED
```

### Company lock implication

Because Ayush has decided that Receiver Accept/Reject should be part of the final Company product before Company is locked, the Company portal should **remain UNLOCKED** until this feature is fully designed, implemented, tested, and manually accepted.

The already completed Company Sender/Receiver History visibility work remains valid and accepted as a completed implementation slice, but it does not by itself close the Company scope while the Accept/Reject requirement is still open.

---

## 24. Required Peer Review / Approval Sequence

The controlled sequence is:

```text
This Future Architecture Package
        ↓
Claude / Grok peer architecture review
        ↓
Reconcile review findings
        ↓
Ayush final architecture approval
        ↓
Create implementation handoff
        ↓
Antigravity preflight
        ↓
Implementation
        ↓
Build + automated validation
        ↓
Implementation report
        ↓
Ayush manual verification
        ↓
Company ACCEPTED / LOCKED
```

No source implementation should begin before the architecture package is explicitly approved.

---

## 25. Final Position

The proposed Receiver Accept/Reject capability is a **real business-workflow expansion**, not merely another Company UI refinement.

The safest architectural direction is:

```text
Existing Trip
      +
Persistent Delivery Request / Handshake
      ↓
Receiver decision
      ↓
Marketplace eligibility gate
      ↓
Existing operational delivery lifecycle
```

This preserves the existing Freight model while adding an explicit agreement layer between Sender and Receiver.

**This record is not a source-implementation authorization.**

---

## 26. Source Records

Primary records used for this package:

- `02_ARCHITECTURE/Chat45_Day18_Node7_Phase1b_Company_Receiver_Accept_Reject_Architecture_Review.md`
- `05_DEBUGGING/investigations/Chat45_Day18_Node7_Phase1b_Company_Receiver_Accept_Reject_Delivery_Investigation_Report.md`
- `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
- `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`
- `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_Authorization_Matrix_v3_Reviewed.md`
- `00_PROJECT_CONTROL/ROADMAP.md`

**Record:** Chat45 / Day18 / Node7 / Phase1b / Company / Receiver Accept-Reject Future Architecture
