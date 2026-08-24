# Chat10 — Node 1 Authorization Matrix — Independent Claude Review

**Project:** Freight — AI Builders Hackathon  
**Active Node:** Node 1 — Product + Authorization Rework  
**Purpose:** Independent review of the current Node 1 authorization model before final lock.  
**Status:** REVIEW REQUEST — NOT FINAL / NOT LOCKED

---

## 1. Review Role

This document is an **independent architecture/security review request for Claude**.

Claude must review the model against the current Freight Records source of truth and identify:

- missing permissions;
- contradictory permissions;
- ambiguous authorization conditions;
- lifecycle/state-transition gaps;
- IDOR/API authorization gaps;
- role/identity inconsistencies;
- evidence-integrity gaps;
- concurrency/atomicity gaps.

Claude must **not silently redesign the product**.

If Claude recommends a change to a previously locked product decision, it must clearly identify the conflict and explain why the change is necessary. Product changes remain subject to Ayush/ChatGPT review before becoming locked.

---

## 2. Source-of-Truth Records

Review these records first:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
01_BRAIN_HANDOFFS/ChatGPT/Chat8_New_Update_Product_Model_and_Security_Direction.md
01_BRAIN_HANDOFFS/ChatGPT/Chat8_New_Update_Authentication_Implementation_Pause_Checkpoint.md
01_BRAIN_HANDOFFS/ChatGPT/Chat9_Roadmap_Review_Node_Subnode_Stretch_Merge.md
01_BRAIN_HANDOFFS/Claude/Chat8_Claude_Independent_Security_Review.md
01_BRAIN_HANDOFFS/Claude/Chat9_Roadmap_Claude_accepted_Node_Subnode_Stretch_Merge.md
```

The current project-control records establish that Node 1 is the active gate, IDOR/API authorization remains OPEN, and authentication implementation remains PAUSED until Node 1 is explicitly locked and verified.

---

## 3. Locked Product / Identity Model

### Roles

Exactly two authenticated application roles exist:

```text
Auth User → Company
OR
Auth User → Driver
```

One Auth user maps strictly to one application identity for the MVP:

```text
1 Auth User ↔ 1 Company identity
OR
1 Auth User ↔ 1 Driver identity
```

A single Auth user must not simultaneously be both Company and Driver.

### Company relationships

The creating/sending Company and receiving Company are **contextual relationships per trip**, not permanent global role types.

The creating Company may also be the receiving Company on a trip.

### Driver model

Drivers are independent marketplace participants.

The Driver does not create/own the trip.

Every authenticated Driver can see a published marketplace opportunity; the Driver decides whether to accept it.

### Trip lifecycle

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

Atomic first-valid acceptance is required:

```text
Multiple valid Driver acceptances
        ↓
Atomic backend/database operation
        ↓
Exactly one winner
        ↓
Trip locks to winning Driver
```

### Delivery lifecycle

```text
Pickup
 ↓
Arrive at pickup
 ↓
Check-in
 ↓
Load goods
 ↓
Depart
 ↓
In transit
 ↓
Arrive at delivery
 ↓
Receiving Company Check-in
 ↓
Unload / delivery
 ↓
Driver Depart
 ↓
Driver completion confirmation
 ↓
Receiving Company delivery confirmation
 ↓
Delivery completed
 ↓
Evidence + AI summary
```

Driver and Receiving Company have distinct delivery responsibilities.

### Emergency changes

Any of the three distinct trip participants may raise an emergency change request:

```text
Sending Company
Assigned Driver
Receiving Company
```

The requester cannot approve their own request.

Every other **distinct** participant must approve or reject.

If any required participant rejects, the request is rejected.

A rejection requires a reason.

If sender and receiver are the same Company, that Company counts as one distinct participant and provides one approval/rejection.

### General Delivery Issues

All three distinct trip participants may raise a General Delivery Issue:

```text
Sending Company   → raise issue
Assigned Driver   → raise issue
Receiving Company → raise issue
```

Other Drivers and unrelated Companies cannot participate in the trip issue.

Other trip participants may respond and provide evidence.

Issue resolution/reopening belongs to the participant directly responsible for the underlying condition:

```text
Driver vehicle/driver responsibility → Driver
Sending Company shipment/document responsibility → Sending Company
Receiving Company warehouse/receiving responsibility → Receiving Company
```

A response does not itself change trip state or trip terms.

If an issue requires a trip-condition change, it can initiate the separate Emergency Change workflow.

### Evidence

The three distinct trip participants may upload photo/video/supporting evidence, but every evidence item must have a structured context:

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

Incorrect evidence is handled through invalidation/reporting plus corrective evidence/events; the original record remains preserved.

Corrections use a new corrective event with a required reason and reference to the original event.

---

## 4. Cross-Cutting Authorization Rules

### Rule A — Server-derived identity

Every protected API action derives authenticated identity from the trusted authentication context.

Client-supplied `user_id`, `company_id`, `driver_id`, or similar identifiers never establish authorization.

### Rule B — Resource ID is not authorization

Knowing a `trip_id`, `issue_id`, `event_id`, `evidence_id`, or other resource identifier never grants access.

### Rule C — Nested resource authorization

For a nested resource, the backend must verify:

```text
Nested resource
 ↓
Parent trip/resource relationship
 ↓
Authenticated actor's relationship
 ↓
Requested action permission
```

### Rule D — Backend enforcement

Authorization must be independently enforced server-side. Frontend visibility/disabled buttons are UX only.

### Rule E — State validation

Every state-changing operation must validate:

```text
Authenticated identity
+ Role
+ Resource relationship
+ Current state
+ Legal transition
+ Action permission
```

### Rule F — Critical concurrency

Critical state-changing operations must be atomic/concurrency-safe so conflicting simultaneous requests cannot create an invalid final state.

---

# 5. Authorization Matrix v2 — Current Draft

Actors:

```text
SC = Sending / Creating Company
D  = Assigned Driver
RC = Receiving Company
OD = Other Driver
OC = Other Company
UA = Unauthenticated
```

## A. Trip lifecycle

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Create trip | ALLOW | DENY | DENY | DENY | DENY |
| Edit own DRAFT | ALLOW | DENY | DENY | DENY | DENY |
| Publish own trip | ALLOW | DENY | DENY | DENY | DENY |
| Cancel DRAFT | ALLOW | DENY | DENY | DENY | DENY |
| Cancel PUBLISHED trip | ALLOW | DENY | DENY | DENY | DENY |
| Edit core fields after publish | DENY | DENY | DENY | DENY | DENY |
| Raise emergency change | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Claim available trip | DENY | ALLOW if eligible | DENY | ALLOW if eligible | DENY |
| Release CLAIMED trip | DENY | ALLOW if assigned and state permits | DENY | DENY | DENY |
| Start CLAIMED trip | DENY | ALLOW if assigned and state permits | DENY | DENY | DENY |

Eligibility for claiming includes authentication, Driver role, trip availability, no conflicting active Driver assignment, and atomic availability check.

## B. Trip visibility

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| View PUBLISHED opportunity | ALLOW | ALLOW | ALLOW | ALLOW | DENY |
| View claimed trip | ALLOW | ALLOW if winner | ALLOW | DENY | DENY |
| View basic assigned Driver identity | ALLOW | ALLOW for self | ALLOW | DENY | DENY |
| View participating Company identities | ALLOW | ALLOW | ALLOW | DENY | DENY |
| View complete trip evidence | ALLOW | ALLOW if assigned | ALLOW | DENY | DENY |

## C. Pickup execution

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Pickup arrival | DENY | ALLOW assigned | DENY | DENY | DENY |
| Pickup check-in | DENY | ALLOW assigned | DENY | DENY | DENY |
| Load goods | DENY | ALLOW assigned | DENY | DENY | DENY |
| Pickup departure | DENY | ALLOW assigned | DENY | DENY | DENY |

## D. Delivery execution

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Delivery arrival | DENY | ALLOW assigned | DENY | DENY | DENY |
| Receiving check-in | DENY | DENY | ALLOW trip receiver | DENY | DENY |
| Unload / delivery | DENY | ALLOW assigned | DENY | DENY | DENY |
| Delivery departure | DENY | ALLOW assigned | DENY | DENY | DENY |
| Driver completion confirmation | DENY | ALLOW assigned | DENY | DENY | DENY |
| Receiver delivery confirmation | DENY | DENY | ALLOW trip receiver | DENY | DENY |

Completion requires both Driver completion confirmation and Receiving Company confirmation.

## E. General Delivery Issues

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Raise issue | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Respond to issue | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Resolve issue | ALLOW only if responsible party | ALLOW only if responsible party | ALLOW only if responsible party | DENY | DENY |
| Reopen issue | ALLOW only if responsible party | ALLOW only if responsible party | ALLOW only if responsible party | DENY | DENY |
| Upload issue evidence | ALLOW | ALLOW | ALLOW | DENY | DENY |

## F. Emergency changes

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Raise emergency request | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Approve own request | DENY | DENY | DENY | DENY | DENY |
| Approve another participant's request | ALLOW if distinct non-requester | ALLOW if distinct non-requester | ALLOW if distinct non-requester | DENY | DENY |
| Reject another participant's request | ALLOW if distinct non-requester | ALLOW if distinct non-requester | ALLOW if distinct non-requester | DENY | DENY |
| Provide rejection reason | REQUIRED on rejection | REQUIRED on rejection | REQUIRED on rejection | DENY | DENY |

## G. Evidence / history

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| View trip evidence | ALLOW | ALLOW assigned | ALLOW | DENY | DENY |
| Upload contextual evidence | ALLOW | ALLOW | ALLOW | DENY | DENY |
| Modify recorded evidence | DENY | DENY | DENY | DENY | DENY |
| Delete recorded evidence | DENY | DENY | DENY | DENY | DENY |
| Report/mark evidence invalid | ALLOW | ALLOW | ALLOW | DENY | DENY |
| View issue history | ALLOW | ALLOW assigned | ALLOW | DENY | DENY |
| View emergency history | ALLOW | ALLOW assigned | ALLOW | DENY | DENY |
| View approval/rejection history | ALLOW | ALLOW assigned | ALLOW | DENY | DENY |

## H. Corrective events

| Action | SC | D | RC | OD | OC |
|---|---:|---:|---:|---:|---:|
| Create corrective event for own authorized event/context | ALLOW if context permits | ALLOW if context permits | ALLOW if context permits | DENY | DENY |
| Edit original event | DENY | DENY | DENY | DENY | DENY |
| Delete original event | DENY | DENY | DENY | DENY | DENY |

---

# 6. Required Claude Review

Review the matrix above against the source records and locked product decisions.

Do not assume a row is correct merely because it is present.

Specifically test:

### Role/identity

1. Can one Auth user accidentally become both Company and Driver?
2. Does every protected action identify the actor from trusted authentication context?
3. Are Company sender/receiver relationships correctly treated as contextual per trip?

### Marketplace

4. Can an unrelated Driver claim a trip after it has already been claimed?
5. Can a losing Driver retain access after atomic claim?
6. Can a Driver claim multiple active trips if the MVP forbids that?

### Lifecycle

7. Can a valid actor skip a state?
8. Can an actor perform an event after the trip is completed?
9. Are Driver and Receiving Company event boundaries correct?
10. Is final completion impossible with only one side's confirmation?

### Issues

11. Can an unrelated actor view an issue?
12. Can the wrong participant resolve/reopen an issue?
13. Can an issue response mutate the trip without the emergency workflow?

### Emergency changes

14. Can requester self-approve?
15. Are all other distinct participants required to decide?
16. Is sender=receiver handled as one participant?
17. Can a rejection occur without a reason?
18. Can a rejected request accidentally mutate the authoritative trip?

### Evidence

19. Can evidence be orphaned?
20. Can evidence be modified/deleted after recording?
21. Can an unrelated actor upload evidence to a trip?
22. Can a corrective event erase the original event?

### IDOR/API

23. Can a known `trip_id` bypass authorization?
24. Can a known `issue_id`, `event_id`, or `evidence_id` bypass parent-trip checks?
25. Are client-supplied identity IDs incorrectly trusted anywhere?
26. Are frontend-only restrictions being mistaken for authorization?

### Concurrency

27. Can two Drivers both successfully claim the same trip?
28. Can two conflicting state transitions both succeed?
29. Can completion occur before all required confirmations exist?

---

# 7. Review Output Required

Return exactly this structure:

```text
REVIEW STATUS: APPROVE / APPROVE WITH CHANGES / REJECT

CRITICAL CONFLICTS:
- ...

MISSING PERMISSIONS:
- ...

IDOR/API GAPS:
- ...

STATE-TRANSITION GAPS:
- ...

CONCURRENCY GAPS:
- ...

EVIDENCE-INTEGRITY GAPS:
- ...

ROLE/IDENTITY GAPS:
- ...

RECOMMENDED CHANGES:
- ...

QUESTIONS THAT MUST BE RESOLVED BEFORE LOCK:
- ...

FINAL RECOMMENDATION:
...
```

If there are no findings in a category, explicitly write `NONE`.

Do not declare Node 1 complete. This is an independent review only.

---

## 8. Review Boundary

This review is **not an implementation request**.

Do not modify application source code.
Do not create database migrations.
Do not implement authentication.
Do not implement authorization.
Do not change GitHub Records.

Only review the design and report findings.

The final authorization model will be locked only after Claude's findings are reviewed and accepted/rejected by ChatGPT + Ayush.

---

## 9. Current Status

```text
Node 1 → ACTIVE
Authorization matrix → DRAFT / REVIEW
IDOR/API authorization → OPEN
Authentication implementation → PAUSED
Final Node 1 lock → NOT YET
```
