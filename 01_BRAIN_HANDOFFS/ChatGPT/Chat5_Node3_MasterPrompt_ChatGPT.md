# Freight — Chat5 Node3 Master Prompt (ChatGPT)

## 1. PROJECT

**Project:** Freight — driver-controlled evidence/accountability web app for freight facility events.

**Purpose:** Record reliable evidence of what happened, when, and where using deterministic GPS, server timestamps, and photos, then present a chronological evidence timeline and AI evidence summary.

**Source repository:** `ayush22cp008/freight_hackathon`

**Records repository:** `ayush22cp008/Freight_Records`

**Claude records replica:** Google Drive `Freight_hackathon_records`; GitHub Records repo is the canonical records source for ChatGPT. Claude uses the Drive replica according to the project setup skill, with manual sync back to GitHub.

**Current brain:** ChatGPT
**Current Chat:** Chat5
**Current Node:** Node 3 — Build execution

## 2. OPERATING RULES — FOLLOW GENERAL-PROJECT-SETUP SKILL

- One reasoning brain at a time: ChatGPT / Claude / Grok switch sequentially.
- Reasoning brains do not modify source code directly. Antigravity executes source changes.
- Records repo is the coordination/memory layer; source repo is the actual application source of truth.
- Investigation and implementation are separate.
- Required investigation pipeline: OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION → FIX → BUILD/TEST → AYUSH MANUAL VERIFICATION.
- Do not guess what the code does. Inspect actual source/records first.
- Tag uncertain claims as VERIFIED / INFERRED / UNKNOWN.
- No side quests: record unrelated problems and keep them separate.
- Only create necessary records; do not duplicate information unnecessarily.
- Chat should show file paths rather than dumping large file contents.
- Ayush is final authority.
- Antigravity is the implementation/execution agent.
- Pushes are manually triggered by Ayush; Antigravity must not push without explicit permission.
- After each completed checkpoint, update the relevant records and report DONE / REMAINING / NEXT STEP.

## 3. LOCKED PRODUCT / ARCHITECTURE

### Core MVP — do not redesign without explicit reason

1. Driver-only authentication.
2. One pre-seeded/assigned trip for the MVP; no trip-creation UI.
3. Exactly three core events: **Arrival → Check-in → Departure**.
4. GPS + server timestamp on every event.
5. Photo mandatory at Arrival and Departure; Check-in photo optional.
6. Events are immutable/insert-only.
7. Driver-facing chronological Timeline.
8. One factual AI Evidence Summary over deterministic stored evidence.

### Navigation/state architecture — LOCKED

- Dashboard / Trip Hub is the single source of truth for the driver's current workflow state and next required action.
- Next-event decisions come from authoritative database state, not temporary client state.
- Normal driver flow is Dashboard → current required event → Dashboard → next required event.
- Navbar is for persistent application navigation, not for bypassing workflow state.
- Do not add Arrival/Check-in/Departure as permanent Navbar shortcuts merely to compensate for missing test data.
- Direct developer test pages may be accessed directly when needed.
- Multi-stop freight journey expansion is deferred for the current MVP.

## 4. WHAT HAS BEEN COMPLETED

### Day 1 / Foundation — completed early

- Next.js + Supabase foundation.
- Driver authentication foundation.
- Pre-seeded/assigned trip model.
- Initial navigation shell.

### Day 2 / Event capture infrastructure — completed early

- Reusable GPS capture utility.
- Server timestamp utility.
- Photo capture/upload utility.
- Supabase Storage integration.
- Immutable event insertion architecture.

### Day 3 / Arrival — completed and manually verified

- `/events/arrival` real workflow implemented.
- Mandatory Arrival photo.
- GPS capture.
- Server timestamp.
- Event insert through server/service-role route.
- Arrival confirmation UI.
- Duplicate Arrival protection verified.
- Full browser/manual Arrival test passed.

### Chat5 — Authentication + Dashboard/Navbar + navigation foundation

- Supabase email/password authentication foundation implemented.
- Driver-to-auth mapping via `drivers.auth_id` implemented.
- Signup/login/logout flow implemented.
- Authenticated application shell/Navbar implemented.
- Dashboard/Trip Hub implemented as workflow controller.
- Dashboard determines next required event from DB state.
- Navigation/state behavior investigated and approved.
- Chat5 navigation architecture is locked.

### DRV002 clean test driver

- `DRV002` was added as a separate test driver.
- A fresh active test trip was assigned to DRV002.
- DRV002 can log in and reach the Dashboard with a fresh Arrival state.

### Manual test just completed

Using DRV002:

`Login → Dashboard → Start Arrival → Arrival page → photo → GPS → server timestamp → Submit Arrival → Arrival Recorded → Return to Dashboard → Arrival Complete → Start Check-in`

This flow was manually verified successfully.

Verified observations:
- Start Arrival appears when DRV002 has an active trip.
- Arrival route opens correctly.
- Photo capture/upload works.
- Browser GPS permission/capture works.
- Server timestamp is recorded.
- Arrival event is successfully saved.
- Confirmation screen appears.
- Returning to Dashboard works.
- Dashboard state changes from Arrival Pending to Arrival Complete.
- Dashboard now exposes **Start Check-in**.

## 5. IMPORTANT INVESTIGATION RESULTS

### DRV002 originally showed no active trip

Root cause was data, not application code: DRV002 existed as a driver but had no matching active trip.

Correct resolution was to add a trip assigned to DRV002 using the existing schema. No Dashboard workaround was required.

### Event Test vs Arrival

The existing `/test-day2` developer test page and real Arrival page reuse the same GPS, server timestamp, and photo utilities.

Decision: do not permanently add Event Test or Arrival as Navbar shortcuts. Use:
- `/test-day2` for isolated developer utility testing.
- Dashboard → Start Arrival for real workflow testing.

## 6. CURRENT PROJECT POSITION

The first three planned days of core work have been completed significantly faster than the original schedule. Dashboard/auth/navigation work was also completed as additional foundation work.

Current verified flow:

`Authentication → Dashboard → Active Trip → Arrival → Arrival Complete → Check-in pending`

**CURRENT ACTIVE FEATURE:** Event 2 — Check-in.

## 7. NEXT WORK — DAY 4 / CHECK-IN

We have permission from Ayush to start the planned Day 4 work early because time is available. The roadmap date is Aug 24, but execution may begin earlier without changing the roadmap unless scope/schedule is formally changed.

Before implementation, investigate the existing Check-in code and reusable event-capture infrastructure.

Target Check-in flow:

`Dashboard → Start Check-in → Check-in page → GPS + server timestamp → optional photo → immutable event insert → Check-in Recorded → Dashboard → Start Departure`

### Required approach

1. Investigate actual current source code first.
2. Produce investigation report in records.
3. Review investigation here.
4. Create implementation plan.
5. Ayush approves plan.
6. Write Antigravity implementation instruction.
7. Antigravity implements.
8. Build/test.
9. Ayush manually verifies.
10. Update project records.

Do not directly implement Check-in before the investigation.

## 8. DAY-4 CHECK-IN REQUIREMENTS

Check-in must reuse the existing event-capture infrastructure rather than duplicate GPS/timestamp/photo logic.

Required:
- GPS.
- Server timestamp.
- Optional photo.
- Immutable event insertion.
- Correct event type = `checkin` set server-side.
- Correct ordering: Arrival must already exist before Check-in is accepted.
- Duplicate Check-in must be rejected cleanly.
- Dashboard must transition to Departure after successful Check-in.
- Existing Arrival behavior must remain unchanged.

## 9. ROADMAP POSITION

The locked working roadmap is:

- Day 1 — Aug 21: Setup + Auth + pre-seeded trip.
- Day 2 — Aug 22: Event capture infrastructure.
- Day 3 — Aug 23: Arrival full flow.
- Day 4 — Aug 24: Check-in.
- Day 5 — Aug 25: Departure + immutability verification.
- Day 6 — Aug 26: Buffer/catch-up.
- Day 7 — Aug 27: Timeline.
- Day 8 — Aug 28: AI evidence summary wiring.
- Day 9 — Aug 29: AI polish/error handling.
- Day 10 — Aug 30: Core MVP freeze + full manual test.

The schedule can be executed early when Ayush chooses, but do not silently rewrite the locked roadmap.

## 10. RECORDS ALREADY UPDATED

Recent Chat5 project-control records were updated for the current state, including:

- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/CHANGELOG.md`
- `00_PROJECT_CONTROL/Chat5_Node3_Flow_And_Navigation_Decision.md`

Relevant investigation/implementation records include the Chat5 Hub/navigation reports and DRV002 investigation records.

## 11. IMPORTANT TESTING STATE

`DRV001` already has Arrival completed and is not the clean starting state.

`DRV002` is the preferred clean manual test driver for continuing the end-to-end flow.

Do not modify or reset DRV001 just to test Check-in.

## 12. CURRENT HANDOFF OBJECTIVE

The next reasoning task is:

**Investigate Check-in implementation readiness and determine the smallest safe implementation plan using the already-verified Arrival/event infrastructure.**

Do not redesign the MVP.
Do not add multi-stop.
Do not add new event types.
Do not bypass Dashboard state logic.
Do not duplicate event-capture utilities.

## 13. SUCCESS CONDITION FOR THIS CHAT HANDOFF

A new ChatGPT / Claude / Grok session should be able to understand immediately:

- what Freight is,
- what architecture is locked,
- what was actually built,
- what was manually verified,
- why DRV002 exists,
- why Navbar does not contain event shortcuts,
- where the project currently is,
- and that **Check-in is the next active task**.
