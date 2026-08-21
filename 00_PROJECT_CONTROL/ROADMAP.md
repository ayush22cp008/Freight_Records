# ROADMAP.md

Hackathon window: 25 days (Aug 21 – Sep 15, 2026). Node 2 (build plan) revised Aug 21 — day-by-day schedule below is now locked as the working plan; adjust only with explicit reason logged in CHANGELOG.md.

Core MVP — 7 items, build exactly this first, always:
1. Driver-only login (driver ID), trip pre-seeded in DB
2. 3 events: Arrival → Check-in → Departure
3. GPS + server timestamp on every event
4. Photo: mandatory Arrival + Departure, optional Check-in
5. Immutable storage (insert-only, service-role route)
6. In-app chronological timeline view
7. AI evidence summary (single LLM call, deterministic data)

## Day-by-Day Schedule (Aug 21 – Sep 15)

| Day | Date | Focus |
|---|---|---|
| 1 | Aug 21 | Setup + Auth + pre-seeded trip (Core #1) |
| 2 | Aug 22 | Event capture infra: GPS + timestamp + photo upload wiring (Core #3, #4) |
| 3 | Aug 23 | Event 1 — Arrival (full flow: capture → service-role insert → confirm) |
| 4 | Aug 24 | Event 2 — Check-in (photo optional per current core scope) |
| 5 | Aug 25 | Event 3 — Departure + immutability lock verification (Core #2, #5) |
| 6 | Aug 26 | Buffer/catch-up for core event flow |
| 7 | Aug 27 | Timeline view — chronological display (Core #6) |
| 8 | Aug 28 | AI evidence summary — single LLM call wiring (Core #7) |
| 9 | Aug 29 | AI evidence summary — polish output, error handling |
| 10 | Aug 30 | CORE MVP FREEZE + full manual test pass (Ayush) |
| 11 | Aug 31 | Bug fixes from Day 10 testing |
| 12 | Sep 1 | Stretch #1 — Public shareable read-only evidence link |
| 13 | Sep 2 | Stretch #2 — AI inconsistency detection (part 1: logic) |
| 14 | Sep 3 | Stretch #2 (part 2: UI surfacing) |
| 15 | Sep 4 | Stretch #3 — Derived dwell-time display |
| 16 | Sep 5 | Stretch #4 — Mandatory photo at Check-in |
| 17 | Sep 6 | Stretch #5 — Repeatable "Add Evidence" mid-trip event (schema change, riskier) |
| 18 | Sep 7 | Stretch #5 continued / buffer |
| 19 | Sep 8 | Stretch #6 — Geofence proximity badge |
| 20 | Sep 9 | Stretch #7 — Company role (dashboard/assignment) — only if time allows |
| 21 | Sep 10 | Stretch #8 — Video capture — only if everything above is done early |
| 22 | Sep 11 | STRETCH FREEZE + AI-depth enhancement (only if time remains, see note below) |
| 23 | Sep 12 | UI polish pass (dedicated, not squeezed) |
| 24 | Sep 13 | Full regression manual test cycle (Ayush) + bug-fix buffer |
| 25 | Sep 14 | Demo video + slide deck + Devpost submission draft |
| — | Sep 15 | Submission deadline — final upload, no new work |

Rule: Core 7 ship first, always. Stretch attempted top-to-bottom. Never touch core to make room for stretch. Last 3 days are never feature days — polish, test, submission only.

## Stretch — ALL 8 targeted, in priority order (build top-down; if time tightens, cut from bottom)

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

## AI-Depth Enhancement Note (NOT a new item — future upgrade to Core Item 7 + Stretch #2)

Judging criteria scores "Innovation and originality" and "Technical implementation" separately from UX/scalability, so AI needs more depth than a thin single-LLM-call layer. Ideas for later (only after core MVP done, time permitting):
- Confidence/completeness scoring — deterministic checklist wrapped in natural language by the LLM
- Multi-signal cross-check — photo EXIF GPS vs logged GPS coordinates, flagging mismatches
- Natural language Q&A over evidence — extension of Stretch #2, answered from deterministic data only

Core principle unchanged — AI never generates GPS/timestamps/evidence, only interprets/organizes/cross-checks deterministic data.
