# Chat24 — Node 5 Architecture Decisions

## Status
Agreed conceptually by Ayush; pending final source/Records consistency review before implementation authorization.

## Decision 1 — Trip Status vs Detailed Events
`trips.status` represents the trip's overall/major lifecycle state. The immutable `events` table represents detailed physical delivery milestones and evidence.

Preferred major trip state flow:

```text
draft → published → claimed → in_progress → completed
```

Do not add `in_transit` to `trips.status` merely to represent the physical transit milestone. Pickup departure and subsequent delivery events provide the detailed lifecycle evidence.

## Decision 2 — Final Completion Confirmations
Driver completion and Receiving Company delivery confirmation are treated as trip-level final acknowledgements rather than ordinary GPS delivery events.

Preferred model:

```text
driver_completion_confirmed_at
receiver_delivery_confirmed_at
```

Final trip completion requires both confirmations and must be performed atomically/server-side.

Exact column names and migration SQL remain implementation details and are not locked here until source/schema review confirms compatibility.

## Decision 3 — Event Vocabulary and UI
Historical Core MVP event values remain unchanged:

```text
arrival
checkin
departure
```

They are legacy historical records and must not be rewritten merely to support Node 5.

New Node 5 lifecycle records use explicit canonical names:

```text
pickup_arrival
pickup_checkin
goods_loaded
pickup_departure
delivery_arrival
receiver_checkin
goods_unloaded
delivery_departure
```

The product should expose ONE unified detailed delivery UI/timeline. It should not show the old three-step UI and new detailed workflow as two simultaneous workflows.

The UI may map legacy records to the same user-facing labels used by the canonical Node 5 workflow.

New Node 5 writes must not create new ambiguous `arrival`, `checkin`, or `departure` records.

## Decision 4 — Uniqueness / Duplicate Protection
Retain duplicate protection at the database level. The preferred strategy is to retain the equivalent of:

```text
UNIQUE (trip_id, event_type)
```

provided final source/schema review confirms every canonical Node 5 event is legitimately single-occurrence within the single-delivery scope.

Canonical event names distinguish pickup and delivery milestones, allowing both `pickup_arrival` and `delivery_arrival` for one trip while preventing duplicate submissions of the same milestone.

Database uniqueness is not authorization. State-transition and actor/relationship authorization must also be enforced server-side.

## Architectural Separation

```text
trips.status
→ overall trip state

trips completion confirmations
→ final human acknowledgements

events
→ detailed physical delivery evidence
```

## Scope Boundary
These decisions do not authorize implementation by themselves. Before implementation, ChatGPT must review the current source/schema evidence and resolve any conflict with existing locked records. Migration SQL, exact column names, exact API contracts, and implementation sequencing remain to be designed/authorized separately.

## Evidence
- `00_PROJECT_CONTROL/ROADMAP.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`
- `03_IMPLEMENTATION/implementation_reports/Chat24_Node5_Current_Source_Investigation_Report.md`
- `05_DEBUGGING/investigations/Chat24_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md`

## Verification State
- VERIFIED: Current schema limitations are evidenced by Chat24 investigation/design reports.
- AGREED: Four conceptual directions were agreed with Ayush in Chat24.
- PENDING: Final consistency review against authoritative source/Records before implementation authorization.
