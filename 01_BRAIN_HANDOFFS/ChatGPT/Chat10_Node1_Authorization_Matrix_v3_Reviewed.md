# Chat10 — Node 1 Authorization Matrix v3 — Claude Findings Incorporated

**Project:** Freight — AI Builders Hackathon
**Node:** Node 1 — Product + Authorization Rework
**Status:** REVIEWED DRAFT — NOT FINAL NODE 1 LOCK

## Purpose

This record supersedes the working matrix conceptually after review of Claude's Chat10 findings and Ayush's decisions 60–69. It is a reviewed design record, not an implementation prompt.

Historical Chat10 review request remains preserved at:

```text
01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_Authorization_Matrix_Claude_Review.md
```

Claude's independent review returned `APPROVE WITH CHANGES`. The findings were reviewed in ChatGPT and the accepted product/security decisions below are now incorporated.

---

## 1. Core identity model

Exactly two authenticated application roles:

```text
Auth User → Company
OR
Auth User → Driver
```

MVP invariant:

```text
1 Auth User ↔ exactly 1 application identity
1 Auth User ↔ exactly 1 application role
```

A user cannot simultaneously be Company and Driver, nor have multiple Company/Driver identities.

The invariant must be technically enforced; the exact database/schema/transaction/application mechanism remains an implementation architecture decision.

---

## 2. Trip participants

A trip can contain these distinct participant relationships:

```text
Sending Company
Assigned Driver
Receiving Company
```

The Sending Company may also be the Receiving Company. In that case the Company is one distinct identity/participant, not two.

Driver is an independent marketplace participant and does not create/own the trip.

Before claim, use the actor category:

```text
ANY AUTHENTICATED DRIVER
```

After successful claim, that Driver becomes:

```text
ASSIGNED DRIVER
```

Do not model an Assigned Driver on an unclaimed trip.

---

## 3. Trip lifecycle

Primary lifecycle:

```text
DRAFT
  ↓
PUBLISHED / AVAILABLE
  ↓
CLAIMED
  ↓
IN_PROGRESS
  ↓
DELIVERED / COMPLETED
```

Cancellation is separate:

```text
DRAFT → CANCELLED
PUBLISHED → CANCELLED
CLAIMED → CANCELLED   (Sending Company may cancel before IN_PROGRESS)
```

Normal cancellation is not available after `IN_PROGRESS`. Post-start problems use General Issue / Emergency Change workflows so operational history is preserved.

Driver release is different from cancellation:

```text
CLAIMED
  ↓
Assigned Driver RELEASES
  ↓
PUBLISHED / AVAILABLE
```

The release is preserved historically. The released Driver may later attempt to claim the trip again subject to normal eligibility and atomic claim rules.

---

## 4. Atomic marketplace claim

Every authenticated Driver can see a PUBLISHED / AVAILABLE opportunity.

Claim requires:

```text
Authenticated Driver
+ trip is PUBLISHED / AVAILABLE
+ Driver satisfies active-trip eligibility
+ atomic availability check
```

Exactly one first-valid acceptance wins.

Once claimed:

```text
Trip → CLAIMED
Winning Driver → Assigned Driver
Other Drivers → no private trip access
```

An already-claimed/in-progress/completed trip cannot be claimed by another Driver.

---

## 5. Strict delivery state machine

Delivery execution is sequential and server-enforced:

```text
PICKUP
  ↓
ARRIVED_AT_PICKUP
  ↓
PICKUP_CHECKED_IN
  ↓
GOODS_LOADED
  ↓
PICKUP_DEPARTED
  ↓
IN_TRANSIT
  ↓
ARRIVED_AT_DELIVERY
  ↓
RECEIVER_CHECKED_IN
  ↓
GOODS_UNLOADED
  ↓
DELIVERY_DEPARTED
  ↓
DRIVER_COMPLETION_CONFIRMED
  ↓
RECEIVER_DELIVERY_CONFIRMED
  ↓
DELIVERED / COMPLETED
```

The backend must reject an event that is not the legal next transition. Frontend sequencing is UX only.

Examples:

```text
ARRIVED_AT_DELIVERY → UNLOAD
DENY until RECEIVER_CHECKED_IN

RECEIVER_CHECKED_IN → UNLOAD
ALLOW to Assigned Driver
```

### Event ownership

```text
Pickup arrival        → Assigned Driver
Pickup check-in       → Assigned Driver
Load goods            → Assigned Driver
Pickup departure      → Assigned Driver
Delivery arrival      → Assigned Driver
Receiving check-in    → Receiving Company
Unload / delivery     → Assigned Driver
Delivery departure    → Assigned Driver
Driver completion     → Assigned Driver
Receiver confirmation → Receiving Company
```

Final completion requires both Driver completion and Receiving Company confirmation.

---

## 6. Atomic final completion

Driver and Receiver may confirm concurrently.

The backend must atomically evaluate the two confirmations and ensure:

```text
both confirmations exist
→ DELIVERED / COMPLETED exactly once
```

No completion with only one confirmation.

Duplicate confirmations and conflicting final states are prevented.

Final AI evidence-summary processing must be idempotent so one completion cannot create duplicate final summaries.

---

## 7. General Delivery Issues

All three distinct trip participants can raise an issue:

```text
Sending Company   → ALLOW
Assigned Driver   → ALLOW
Receiving Company → ALLOW
Other Driver      → DENY
Other Company     → DENY
```

Other trip participants may respond and upload evidence.

The responsible participant resolves/reopens according to the underlying condition:

```text
Driver responsibility   → Driver
Sender responsibility   → Sending Company
Receiver responsibility → Receiving Company
```

A response does not itself mutate trip terms/state.

If trip conditions must change, use Emergency Change.

Sender=Receiver is one Company participant; no duplicate approval, notification, or resolution requirement is created merely because the same identity occupies both contextual relationships.

---

## 8. Emergency Change

Any distinct trip participant may request an emergency change.

The requester cannot approve itself.

Every other distinct trip participant must submit exactly one decision:

```text
APPROVE
OR
REJECT + required reason
```

All required approvals are necessary.

Any required rejection rejects the request.

If sender=receiver, that Company counts once.

Emergency decisions must be processed atomically so duplicate decisions, approval-after-finalization, rejection-after-finalization, and inconsistent final outcomes are prevented.

A rejected request never becomes authoritative trip state.

---

## 9. Evidence model

Only the three distinct trip participants can upload evidence.

Every evidence item must be attached to a structured context:

```text
Delivery Event
OR General Delivery Issue
OR Emergency Change Request
OR Corrective Event
```

No orphan evidence.

Recorded evidence is immutable:

```text
Modify → DENY
Delete → DENY
```

Incorrect evidence is handled through invalidation/reporting and corrective evidence/events; the original remains preserved.

Corrective event requirements:

```text
original event reference
+ correction reason
+ corrective information
+ actor
+ server timestamp
+ supporting evidence where applicable
```

Sender=Receiver remains one participant for evidence permissions and trip history visibility.

---

## 10. Authorization matrix v3

Actors:

```text
SC = Sending Company
D  = Assigned Driver (post-claim)
RC = Receiving Company
AD = Any authenticated Driver (pre-claim marketplace)
OD = Other/unassigned Driver for a claimed trip
OC = Other Company
UA = Unauthenticated
```

### Trip lifecycle

| Action | SC | D | RC | AD | OD | OC |
|---|---|---|---|---|---|---|
| Create trip | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Edit own DRAFT | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Publish own DRAFT | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Cancel DRAFT | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Cancel PUBLISHED | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Cancel CLAIMED before IN_PROGRESS | ALLOW | DENY | DENY | DENY | DENY | DENY |
| Cancel IN_PROGRESS or later | DENY via ordinary cancel | DENY | DENY | DENY | DENY | DENY |
| Edit core fields after publish | DENY | DENY | DENY | DENY | DENY | DENY |
| Raise emergency change | ALLOW | ALLOW | ALLOW | DENY | DENY | DENY |
| Claim PUBLISHED trip | DENY | N/A | DENY | ALLOW if eligible | ALLOW if eligible | DENY |
| Release CLAIMED trip | DENY | ALLOW if assigned and before IN_PROGRESS | DENY | DENY | DENY | DENY |
| Start CLAIMED trip | DENY | ALLOW if assigned and legal next state | DENY | DENY | DENY | DENY |

`AD` claim authorization is based on authenticated Driver identity, availability, eligibility, and atomic claim. `D` does not exist before claim.

### Visibility / lists

| Action | SC | D | RC | AD | OD | OC |
|---|---|---|---|---|---|---|
| View PUBLISHED marketplace opportunity | N/A/own | N/A | N/A | ALLOW | ALLOW | DENY |
| View own Company trip list | ALLOW | DENY | ALLOW if this is same Company identity | DENY | DENY | DENY |
| View own Driver trip list/history | DENY | ALLOW | DENY | ALLOW for own history | DENY | DENY |
| View claimed trip as Sending Company | ALLOW | DENY | DENY | DENY | DENY | DENY |
| View claimed trip as Assigned Driver | DENY | ALLOW | DENY | DENY | DENY | DENY |
| View claimed trip as Receiving Company | DENY | DENY | ALLOW | DENY | DENY | DENY |
| View complete authorized trip evidence | ALLOW | ALLOW | ALLOW | DENY | DENY | DENY |

For simplicity, the application should expose role-appropriate dashboards rather than treating `N/A` as a security permission. The backend scopes every list query from authenticated identity and relationship.

### Pickup

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| Pickup arrival | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Pickup check-in | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Load goods | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Pickup departure | DENY | ALLOW if legal next state | DENY | DENY | DENY |

### Delivery

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| Delivery arrival | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Receiving check-in | DENY | DENY | ALLOW if legal next state | DENY | DENY |
| Unload / delivery | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Delivery departure | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Driver completion confirmation | DENY | ALLOW if legal next state | DENY | DENY | DENY |
| Receiver delivery confirmation | DENY | DENY | ALLOW if legal next state | DENY | DENY |

### Issues

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| Raise issue | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Respond | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Resolve | ALLOW only if responsible | ALLOW only if responsible | ALLOW only if responsible | DENY | DENY |
| Reopen | ALLOW only if responsible | ALLOW only if responsible | ALLOW only if responsible | DENY | DENY |
| Upload contextual issue evidence | ALLOW | ALLOW | ALLOW | DENY | DENY |

### Emergency

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| Raise request | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Approve own request | DENY | DENY | DENY | DENY | DENY |
| Approve another participant | ALLOW if distinct non-requester | ALLOW if distinct non-requester | ALLOW if distinct non-requester | DENY | DENY |
| Reject another participant | ALLOW if distinct non-requester | ALLOW if distinct non-requester | ALLOW if distinct non-requester | DENY | DENY |
| Rejection reason | REQUIRED on rejection | REQUIRED on rejection | REQUIRED on rejection | DENY | DENY |

### Evidence/history

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| View trip evidence | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Upload contextual evidence | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Modify recorded evidence | DENY | DENY | DENY | DENY | DENY |
| Delete recorded evidence | DENY | DENY | DENY | DENY | DENY |
| Report/mark evidence invalid | ALLOW | ALLOW | ALLOW | DENY | DENY |
| View issue history | ALLOW | ALLOW | ALLOW | DENY | DENY |
| View emergency history | ALLOW | ALLOW | ALLOW | DENY | DENY |
| View approval/rejection history | ALLOW | ALLOW | ALLOW | DENY | DENY |

### Corrective events

| Action | SC | D | RC | OD | OC |
|---|---|---|---|---|---|
| Create corrective event for authorized context | ALLOW if responsible/authorized context | ALLOW if responsible/authorized context | ALLOW if responsible/authorized context | DENY | DENY |
| Edit original event | DENY | DENY | DENY | DENY | DENY |
| Delete original event | DENY | DENY | DENY | DENY | DENY |

---

## 11. Cross-cutting IDOR/API rules

### Server-derived identity

The backend derives identity from authenticated session/token context. Client-supplied identity IDs never establish authorization.

### Resource ID is not authorization

Knowing a `trip_id`, `issue_id`, `event_id`, `evidence_id`, etc. never grants access.

### Parent-trip verification

For every nested-resource request, the backend derives the resource's actual parent trip and verifies it against any supplied trip context before authorization.

Conceptually:

```text
resource
 ↓
actual parent_trip_id
 ↓
if supplied trip_id exists: cross-check equality
 ↓
authenticated actor ↔ trip relationship
 ↓
action-specific authorization
```

This applies whether the API is nested (`/trips/:tripId/issues/:issueId`) or flat (`/issues/:issueId`).

### Backend enforcement

Frontend restrictions are UX only. Every protected API action independently validates authorization.

### State validation

Every state-changing API validates identity + role + relationship + current state + legal transition + action permission.

### Unauthorized enumeration

For protected resources that the requester is not authorized to know about, the external API should not reveal existence; use a not-found style response (404) while internal security logs may retain the authorization failure.

### Concurrency

Critical transitions are atomic/concurrency-safe. At minimum:

```text
claim
release
start
emergency decisions
final confirmations/completion
```

Duplicate decisions/confirmations are rejected or treated idempotently as appropriate, and final side effects are idempotent.

---

## 12. Accepted Claude findings / resolved questions

```text
Release CLAIMED → PUBLISHED / AVAILABLE
Strict delivery event ordering → LOCKED
Atomic final completion → LOCKED
Atomic emergency decision processing → LOCKED
Nested-resource parent verification → LOCKED
Unauthorized resource enumeration protection → LOCKED
One Auth User ↔ One Application Identity invariant → LOCKED
Post-claim cancellation before IN_PROGRESS → LOCKED
Company trip list visibility → LOCKED
Driver trip/history list visibility → LOCKED
Pre-claim actor = Any Authenticated Driver → LOCKED
```

---

## 13. Still not final

This record is **not** the final Node 1 completion/lock record yet.

Remaining work is to perform a final consistency review of this v3 matrix against the project-control records and then derive the authentication contract from the locked identity/authorization model.

No implementation should begin from this record until Node 1 is formally locked and the required verification stage is complete.

```text
Node 1 → ACTIVE
Authorization Matrix → REVIEWED DRAFT v3
IDOR/API Authorization → substantially defined; final lock pending
Authentication implementation → PAUSED
Final Node 1 Lock → NOT YET
```
