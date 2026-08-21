# Chat4_Node3_MasterPrompt_Claude.md

**Project:** Freight — AI Builders Hackathon (Devpost)
**Chat:** #4 → handing off to Chat #5
**Active reasoning brain:** Claude
**Date:** Aug 21, 2026
**Purpose:** Paste this whole file into new chat to continue with zero context loss.

---

## 1. What This Product Is (LOCKED — do not re-discuss)

**Evidence notary, NOT a detention calculator.**

Driver logs 3 events at a freight facility: **Arrival → Check-in → Departure**. Each event captures GPS coordinates + server timestamp + photo (mandatory at Arrival + Departure, optional at Check-in). All records are **immutable** (insert-only via Supabase RLS + service-role server route — UPDATE/DELETE revoked at the DB level from ALL roles including service_role — no edits ever, by anyone). AI generates a **single factual evidence summary** after all events captured — one LLM call on deterministic data only. No AI decisions, no blame, no interpretation.

**Core principle:** *"We do not decide who is right. We create a reliable evidence record."*

Full product/scope detail: see `MASTER_ARCHITECTURE.md` and `ROADMAP.md` in `00_PROJECT_CONTROL/` — do not re-derive, just reference.

---

## 2. Timeline (LOCKED)

25-day hackathon window: **Aug 21 – Sep 15, 2026**. Full day-by-day schedule locked in `ROADMAP.md`. Currently on **Day 3 of 25, COMPLETE**. Next up: Day 4.

---

## 3. Stack & Infra (LOCKED)

- **Stack:** Next.js (App Router, `src/` dir convention) + Supabase (DB+Auth+Storage) + Vercel
- **Active Supabase project:** `freight_hackathon`, `https://nzsexdmcvhoqsywxxnpe.supabase.co`, ap-south-1 Mumbai, Nano tier
- **Write pattern (VERIFIED, non-negotiable):** ALL inserts go through Next.js API routes using `service_role` key server-side only (never `NEXT_PUBLIC_*`). RLS stays enabled everywhere as defense-in-depth.
- **Records repo:** `Freight_Records` (GitHub, public) — source of truth, confirmed byte-for-byte synced with the Drive replica as of Aug 21
- **Records replica (Claude's write-access workaround):** Google Drive folder `Freight_hackathon_records` — Claude writes here directly, Ayush syncs to GitHub
- **Source code repo:** `freight_hackathon` (GitHub, public)
- **Storage bucket:** `event-photos` (Supabase Storage) — live, confirmed working

---

## 4. Node-Map Status

- ✅ Node 0 — Problem research & selection — LOCKED
- ✅ Node 1 — Solution design + stack — LOCKED
- ✅ Node 2 — Build plan (25-day scope, revised Aug 20) — LOCKED, full schedule in ROADMAP.md
- ✅ Node 2.5 — Core logic test (GPS, camera→storage upload, immutable insert-only via service-role) — LOCKED
- 🔄 **Node 3 — Build execution — ACTIVE, currently Day 3 of 25, COMPLETE**
  - ✅ Day 1 (Core Item #1: Next.js setup, Supabase clients, driver-only login, pre-seeded trip) — **LOCKED, verified working**
  - ✅ Day 2 (Core Items #3, #4 infra: GPS + timestamp + photo upload utilities, throwaway `/test-day2` test page) — **LOCKED, verified working**, also re-verified post a Supabase platform JWT incident (see Section 7) — no impact
  - ✅ Day 3 (Event 1 — Arrival: `events` table migration + API route + UI, full flow) — **LOCKED, verified working**, including the duplicate-arrival 409 edge case
  - ⬜ **Day 4 (Event 2 — Check-in, photo optional) — NEXT TASK, not yet started**
  - ⬜ Day 5 (Event 3 — Departure + immutability lock re-verification)
- ⬜ Node 4 — Demo video + slide deck + submission — NOT STARTED

---

## 5. Day 3 — Completion Evidence (LOCKED, do not re-open)

Built: `events` table via migration `src/db/migrations/002_create_events_table.sql` (schema below), `/api/events/arrival` route (service-role insert, server-side-hardcoded `event_type`, graceful 409 on duplicate), `/events/arrival` UI page (mandatory photo, GPS+timestamp+photo capture reusing Day 2 utils unmodified, clear success/error states).

**Locked `events` table schema** (full rationale: `02_ARCHITECTURE/locked_decisions/Chat4_Node3_Decision_EventsTableSchema.md`):

```sql
CREATE TABLE events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_id uuid NOT NULL REFERENCES trips(id),
  driver_id uuid NOT NULL REFERENCES drivers(id),
  event_type text NOT NULL CHECK (event_type IN ('arrival', 'checkin', 'departure')),
  latitude numeric NOT NULL,
  longitude numeric NOT NULL,
  gps_accuracy numeric,
  server_timestamp timestamptz NOT NULL,
  photo_url text,
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (trip_id, event_type)
);

ALTER TABLE events ENABLE ROW LEVEL SECURITY;
REVOKE UPDATE, DELETE ON events FROM PUBLIC, anon, authenticated, service_role;
```

Key locked decisions: fixed 3 event types only for MVP (N-events is Stretch #5, deferred); `event_type` is `text + CHECK` not Postgres ENUM (extensibility for later); `(trip_id, event_type)` UNIQUE prevents duplicate events; immutability is enforced at the DB grant level, not just app discipline — this was an explicit, accepted trade-off (harder to patch bad data later, but true tamper-proof guarantee, which matters for the core product pitch).

**Verified (Ayush, manual browser test):**
- Full arrival flow: GPS capture → server timestamp → mandatory photo upload → submit → DB insert → confirmation UI, all PASS
- Duplicate-arrival submission correctly returns 409 with "Arrival already recorded for this trip" — PASS

Full detail: `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day3_ArrivalEventFlow.md`.

---

## 6. Day 2 — Completion Evidence (LOCKED, do not re-open)

Built: `getGpsLocation.ts`, `/api/server-time` route + `getServerTime.ts`, `/api/upload-photo` route (service-role, `event-photos` bucket) + `uploadPhoto.ts`, throwaway `/test-day2` test page. All manually verified PASS by Ayush. Full detail: `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day2_GPS_Timestamp_PhotoUpload.md`.

---

## 7. Day 1 — Completion Evidence (LOCKED, do not re-open)

Built: Next.js project (`src/` convention), both Supabase clients (anon for reads, service-role for writes), driver-only login (`driver_code` based, no password — intentional MVP simplification), pre-seeded trip.

**Bug found + fixed during Day 1:** `drivers`/`trips` tables were never created in Supabase — fixed via migration `src/db/migrations/001_create_core_tables.sql`, committed to repo.

Full detail: `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day1_SetupAuthPreseededTrip.md` and `Chat4_Node3_Report_FixMissingTables.md`.

---

## 8. Non-project event (resolved, no action needed)

A Supabase platform incident ("401 errors due to JWT rejections," affecting a subset of new projects on some JWT renewals, window Aug 18–20 UTC, fixed Aug 20 16:37 UTC) surfaced via email Aug 21. Since `freight_hackathon` is a new project from the same window, all three Day 2 utilities were re-tested post-fix on Aug 21 (~07:15 UTC) — GPS, timestamp, and photo upload all returned clean 200s, no 401s. **Freight was not affected. No action was or is needed.** Addendum recorded in the Day 2 implementation report (Section 6 above).

---

## 9. Sync status (verified Aug 21)

Google Drive replica (`Freight_hackathon_records`) and GitHub records repo (`Freight_Records`) were cross-checked file-by-file via the GitHub API — folder structure and file sizes matched exactly (`CURRENT_STATUS.md`, Day 2/Day 3 reports, and the schema decision file all byte-identical). Records are in sync as of this handoff.

---

## 10. Workflow Rules (unchanged — per `general-project-setup` skill, do not re-explain)

- Reasoning brain (Claude/ChatGPT/Grok, one active at a time) = architecture/diagnosis/spec-writing only, never touches source code
- Antigravity = implementation/execution only — code changes, file editing, terminal/build verification, no manual UI testing
- Ayush = final authority, all manual browser/UI testing
- Investigation and fix are always separate instruction files (never mixed)
- Manual DB/console changes must be captured into a committed migration file immediately
- Antigravity instructions → `03_IMPLEMENTATION/prompts/`, no permission needed
- All other new files → explicit permission from Ayush before creating
- Chat only ever shows file paths, never file content pasted inline
- GitHub push is manually triggered by Ayush only

---

## 11. Next Required Action

**Day 4 — Event 2: Check-in.** Same pattern as Day 3's Arrival flow (API route + UI page), but:
- `event_type = 'checkin'`
- Photo is **optional**, not mandatory (per Core Item #4 and ROADMAP — Stretch #4 later makes it mandatory, but that's not yet)
- Reuse Day 2 utilities and the Day 3 API/UI pattern — no new architecture decisions expected, this should be a fast, low-risk day

No open questions or blockers going into Day 4. Awaiting Ayush's go-ahead in the new chat to write the Day 4 Antigravity instruction.
