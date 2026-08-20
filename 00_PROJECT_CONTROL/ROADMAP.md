# ROADMAP.md

Core MVP — 7 items, build exactly this, nothing else:
1. Driver-only login (driver ID), trip pre-seeded in DB
2. 3 events: Arrival → Check-in → Departure
3. GPS + server timestamp on every event
4. Photo: mandatory Arrival + Departure, optional Check-in
5. Immutable storage (insert-only, service-role route)
6. In-app chronological timeline view
7. AI evidence summary (single LLM call, deterministic data)

| Day | Focus | MVP Items |
|---|---|---|
| Day 1 (Aug 21) | Setup + Auth + pre-seeded trip | Item 1 |
| Day 2 | Event-capture component (GPS+photo+timestamp+immutable insert) | Items 2,3,4,5 |
| Day 3 AM–PM | Timeline view + AI evidence summary | Items 6,7 → CORE MVP DONE |
| Day 3 eve → Day 4 AM | 🛡️ Buffer — bug-fix only | — |
| Day 4 PM–eve | UI polish + demo video + slide deck + submission | — |

Stretch (only if core done early): Company role, read-only shareable evidence link, repeatable "Add Evidence" event, AI inconsistency detection, video capture.
