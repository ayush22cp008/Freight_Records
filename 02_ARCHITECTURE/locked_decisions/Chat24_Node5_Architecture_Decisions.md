# Chat24 — Node 5 Architecture Decisions

## Status
Q1–Q4 have been reviewed with Ayush. Q3 and Q4 were explicitly resolved in Chat25. Implementation remains unauthorized pending completion of the remaining 5.S1 design/evidence work and final source/Records consistency review.

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

### Chat25 Resolution — Option A Approved

Ayush explicitly selected **Option A** after review of the Claude consistency finding. The exact persisted event vocabulary for new Node 5 lifecycle records must follow the authoritative Node 1 FINAL LOCK literal names:

```text
ARRIVED_AT_PICKUP
PICKUP_CHECKED_IN
GOODS_LOADED
PICKUP_DEPARTED
IN_TRANSIT
ARRIVED_AT_DELIVERY
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

The earlier Chat24 lowercase proposal (`pickup_arrival`, `pickup_checkin`, `goods_loaded`, etc.) is superseded for new Node 5 persisted lifecycle records and must not be implemented.

The product should expose ONE unified detailed delivery UI/timeline. It should not show the old three-step UI and new detailed workflow as two simultaneous workflows.

The UI may map legacy records to the same user-facing labels used by the canonical Node 5 workflow.

New Node 5 writes must not create new ambiguous `arrival`, `checkin`, or `departure` records.

## Decision 4 — Uniqueness / Duplicate Protection

### Chat25 Resolution — Option 1 Approved

Ayush explicitly approved **Option 1: retain canonical lifecycle uniqueness and use a separate mechanism for repeatable evidence**.

The canonical Node 5 lifecycle events are single-occurrence milestones within the single-delivery scope. Therefore retain database-level duplicate protection equivalent to:

```text
UNIQUE (trip_id, event_type)
```

for the canonical lifecycle event records.

Canonical lifecycle examples include:

```text
ARRIVED_AT_PICKUP
PICKUP_CHECKED_IN
GOODS_LOADED
PICKUP_DEPARTED
IN_TRANSIT
ARRIVED_AT_DELIVERY
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

If the deferred/stretch **Repeatable Add Evidence** capability is implemented later, it must use a separate repeatable evidence mechanism rather than weakening or duplicating the canonical lifecycle event types.

This preserves database-level protection against duplicate lifecycle milestones while allowing future repeatable evidence without abusing canonical event types.

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

## Q3 / Q4 Resolution State — Chat25

```text
Q3 event vocabulary → RESOLVED / OPTION A APPROVED
Q4 uniqueness model → RESOLVED / OPTION 1 APPROVED
```

The decisions above are conceptually resolved by Ayush. Formal implementation readiness still requires the outstanding 5.S1 schema/design evidence and final source/Records consistency validation.

## Scope Boundary
These decisions do not authorize implementation by themselves. Before implementation, ChatGPT must review the current source/schema evidence and resolve any conflict with existing locked records. Migration SQL, exact column names, exact API contracts, and implementation sequencing remain to be designed/authorized separately.

## Evidence
- `00_PROJECT_CONTROL/ROADMAP.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`
- `01_BRAIN_HANDOFFS/Claude/Node 5 Architecture Consistency Review — Chat24_Node5_Architecture_Decisions.md`
- `03_IMPLEMENTATION/implementation_reports/Chat24_Node5_Current_Source_Investigation_Report.md`
- `05_DEBUGGING/investigations/Chat24_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md` — currently missing; remains an evidence/path issue to resolve before implementation.

## Verification State
- VERIFIED: Q3 Option A was explicitly selected by Ayush in Chat25.
- VERIFIED: Q4 Option 1 was explicitly approved by Ayush in Chat25.
- VERIFIED: The existing source/schema investigation identified the current `(trip_id, event_type)` uniqueness constraint.
- AGREED: Canonical lifecycle milestones remain single-occurrence within the single-delivery scope; repeatable evidence is a separate deferred capability.
- PENDING: Formal 5.S1 schema/design evidence and final source/Records consistency review before implementation authorization.
