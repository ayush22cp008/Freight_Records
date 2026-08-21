# Chat4_Node3_MasterPrompt_Claude.md

**Project:** Freight — AI Builders Hackathon (Devpost)
**Chat:** #4 → handing off to Chat #5
**Active reasoning brain:** Claude
**Date:** Aug 21, 2026
**Purpose:** Paste this whole file into new chat to continue with zero context loss.

---

## 1. What This Product Is (LOCKED — do not re-discuss)

**Evidence notary, NOT a detention calculator.**

Driver logs 3 events at a freight facility: **Arrival → Check-in → Departure**. Each event captures GPS coordinates + server timestamp + photo (mandatory at Arrival + Departure, optional at Check-in). All records are **immutable** (insert-only via Supabase RLS + service-role server route — no edits ever). AI generates a **single factual evidence summary** after all events captured — one LLM call on deterministic data only. No AI decisions, no blame, no interpretation.

**Core principle:** *"We do not decide who is right. We create a reliable evidence record."*

Full product/scope detail: see `MASTER_ARCHITECTURE.md` and `ROADMAP.md` in `00_PROJECT_CONTROL/` — do not re-derive, just reference.

---

## 2. Timeline (LOCKED)

25-day hackathon window: **Aug 21 – Sep 15, 2026**. Full day-by-day schedule locked in `ROADMAP.md`. Currently on **Day 2 of 25**.

---

## 3. Stack & Infra (LOCKED)

- **Stack:** Next.js (App Router, `src/` dir convention) + Supabase (DB+Auth+Storage) + Vercel
- **Active Supabase project:** `freight_hackathon`, `https://nzsexdmcvhoqsywxxnpe.supabase.co`, ap-south-1 Mumbai, Nano tier
- **Write pattern (VERIFIED, non-negotiable):** ALL inserts go through Next.js API routes using `service_role` key server-side only (never `NEXT_PUBLIC_*`). RLS stays enabled everywhere as defense-in-depth. Client-side anon-key writes are never used — confirmed platform bug on anon-key inserts in earlier testing (Node 2.5).
- **Records repo:** `Freight_Records` (GitHub, public) — source of truth once synced
- **Records replica (Claude's write-access workaround):** Google Drive folder `Freight_hackathon_records` — Claude writes here directly, Ayush syncs to GitHub
- **Source code repo:** `freight_hackathon` (GitHub, public)

---

## 4. Node-Map Status

- ✅ Node 0 — Problem research & selection — LOCKED
- ✅ Node 1 — Solution design + stack — LOCKED
- ✅ Node 2 — Build plan (25-day scope, revised Aug 20) — LOCKED, full schedule in ROADMAP.md
- ✅ Node 2.5 — Core logic test (GPS, camera→storage upload, immutable insert-only via service-role) — LOCKED
- 🔄 **Node 3 — Build execution — ACTIVE, currently Day 2 of 25**
  - ✅ Day 1 (Core Item #1: Next.js setup, Supabase clients, driver-only login, pre-seeded trip) — **LOCKED, verified working**
  - 🔄 Day 2 (Core Items #3, #4: GPS + timestamp + photo upload wiring) — next task, not yet started
- ⬜ Node 4 — Demo video + slide deck + submission — NOT STARTED

---

## 5. Day 1 — Completion Evidence (LOCKED, do not re-open)

Built: Next.js project (`src/` convention), both Supabase clients (anon for reads, service-role for writes), driver-only login (`driver_code` based, no password — intentional MVP simplification), pre-seeded trip.

**Bug found + fixed during Day 1:** `drivers`/`trips` tables were never created in Supabase (`relation "drivers" does not exist`) — original Day 1 spec described column structure but never included the actual `CREATE TABLE` statement. Fixed via new migration `src/db/migrations/001_create_core_tables.sql`, committed to repo.

**Verified (Ayush, manual browser test):**
- Valid driver code (`DRV001`) → login succeeds → redirects to protected root page → shows Driver ID from session cookie
- Invalid code → proper "Invalid driver code" error, no crash
- Supabase Table Editor confirms seeded driver + trip rows exist correctly

Full detail: `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day1_SetupAuthPreseededTrip.md` and `Chat4_Node3_Report_FixMissingTables.md`.

---

## 6. Workflow Rules (unchanged — per `general-project-setup` skill, do not re-explain)

- Reasoning brain (Claude/ChatGPT/Grok, one active at a time) = architecture/diagnosis/spec-writing only, never touches source code
- Antigravity = implementation/execution only — code changes, file editing, terminal/build verification, no manual UI testing
- Ayush = final authority, all manual browser/UI testing
- Investigation and fix are always separate instruction files (never mixed)
- Manual DB/console changes must be captured into a committed migration file immediately — this was the exact failure mode in the Day 1 bug, now corrected
- Antigravity instructions → `03_IMPLEMENTATION/prompts/`, no permission needed
- All other files → explicit permission required (already given for this master prompt + records updates)
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.md`

---

## 7. Immediate Next Step

Write Day 2 implementation instruction for Antigravity: **Event capture infra — GPS (`navigator.geolocation`) + server timestamp + photo upload wiring (Core Items #3, #4)**. This is infra/plumbing only — the actual Arrival/Check-in/Departure event flows come Days 3–5 per ROADMAP.md. Scope Day 2 tightly: don't build the 3 event UIs yet, just the reusable capture components/utilities and the service-role insert pattern they'll plug into.

Reference verified patterns from Node 2.5 testing (GPS via `navigator.geolocation`, camera via `<input capture="camera">`, upload to Supabase Storage `test-photos` bucket pattern — adapt to real bucket for this phase) before writing the spec, rather than re-deriving from scratch.
