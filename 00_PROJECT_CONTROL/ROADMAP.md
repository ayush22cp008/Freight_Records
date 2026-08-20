# ROADMAP.md

⚠️ Hackathon window corrected: 25 days (Aug 21 – Sep 15), not 4 days. Detailed day-by-day schedule to be finalized as build progresses — high-level phases only below.

Core MVP — 7 items, build exactly this first, always:
1. Driver-only login (driver ID), trip pre-seeded in DB
2. 3 events: Arrival → Check-in → Departure
3. GPS + server timestamp on every event
4. Photo: mandatory Arrival + Departure, optional Check-in
5. Immutable storage (insert-only, service-role route)
6. In-app chronological timeline view
7. AI evidence summary (single LLM call, deterministic data)

| Phase | Focus |
|---|---|
| Setup + Auth + pre-seeded trip | Core Item 1 |
| Event-capture (GPS+photo+timestamp+immutable insert) × 3 events | Core Items 2,3,4,5 |
| Timeline view + AI evidence summary | Core Items 6,7 → FULL CORE MVP DONE |
| Stretch items, priority order 1→8 | As many as time allows, never at core's expense |
| AI-depth enhancement (see note below) | Only if time remains after stretch items |
| UI polish pass | Dedicated time |
| Manual testing cycles (Ayush) | Dedicated time, multiple passes now possible |
| Buffer | Bug-fix only, no new features |
| Demo video + slide deck + submission | Final days before Sep 15 deadline |

Stretch — ALL 8 targeted, in priority order (build top-down; if time tightens, cut from bottom):

| Priority | Item | Why this rank |
|---|---|---|
| 1 | Public shareable read-only evidence link | High demo/judging impact, low build complexity |
| 2 | AI inconsistency detection (e.g. note-time vs GPS-time mismatch) | Deepens core AI differentiator — high innovation/technical score |
| 3 | Derived dwell-time display | Low complexity, purely derived from existing timestamps |
| 4 | Mandatory photo at Check-in (currently optional) | Trivial change, strengthens evidence completeness |
| 5 | Repeatable "Add Evidence" mid-trip event | Moderate complexity — schema needs to support N events, not fixed 3 |
| 6 | Geofence proximity badge ("Verified at Location") | Moderate complexity — distance calc from facility coords |
| 7 | Company role (dashboard, trip creation, driver assignment) | Higher complexity — multi-role auth + new UI surface |
| 8 | Video capture | Highest complexity (storage/bandwidth), lowest incremental evidence value over photo |

Rule: Core 7 ship first, always. Stretch attempted top-to-bottom. Never touch core to make room for stretch.

AI-Depth Enhancement Note (NOT a new item — future upgrade to Core Item 7 + Stretch #2): Judging criteria scores "Innovation and originality" and "Technical implementation" separately from UX/scalability, so AI needs more depth than a thin single-LLM-call layer. Ideas for later (only after core MVP done, time permitting): confidence/completeness scoring, multi-signal cross-check (photo EXIF GPS vs logged GPS), natural language Q&A over evidence. Core principle unchanged — AI never generates GPS/timestamps/evidence, only interprets/organizes/cross-checks deterministic data.
