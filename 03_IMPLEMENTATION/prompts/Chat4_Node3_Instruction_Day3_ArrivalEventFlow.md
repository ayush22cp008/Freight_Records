# Chat4_Node3_Instruction_Day3_ArrivalEventFlow.md

**Project:** Freight — AI Builders Hackathon
**Chat:** #4 | **Node:** 3 (Build execution) | **Day:** 3 of 25
**Type:** Implementation instruction (build, not a fix)
**For:** Antigravity

---

## Scope (LOCKED — do not exceed)

Build the **first real event: Arrival**, full flow, using Day 2's utilities.

1. `events` table migration (schema locked — see below)
2. Arrival event API route (service-role insert)
3. Arrival event UI page (capture → submit → confirm)

Do NOT build Check-in or Departure yet — those are Day 4/5. Do NOT build the timeline view or AI summary — later days.

---

## 1. Migration — `events` table

Schema is LOCKED. Full detail and rationale: `02_ARCHITECTURE/locked_decisions/Chat4_Node3_Decision_EventsTableSchema.md` — reference it, do not re-derive.

Create `src/db/migrations/002_create_events_table.sql`:

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

Apply this migration to the `freight_hackathon` Supabase project. Confirm table exists and grants are correctly revoked (e.g. via Supabase SQL editor `\dp events` or equivalent check) before moving to step 2.

Per workflow rule: this is a manual DB change outside normal app code — it's already being captured as a committed migration file, which satisfies that rule. No separate action needed beyond committing this file.

---

## 2. Arrival event API route

Create `src/app/api/events/arrival/route.ts`:

- POST route, uses `service_role` key server-side (per locked write pattern — never client-side anon insert).
- Accepts: `trip_id`, `driver_id`, `latitude`, `longitude`, `gps_accuracy` (optional), `server_timestamp`, `photo_url`.
- `event_type` is hardcoded to `'arrival'` server-side — do not trust/accept it from the client body (prevents a client from posting an arrival to the wrong endpoint's type).
- Insert into `events` table. Handle the UNIQUE constraint violation gracefully — if an arrival already exists for this `trip_id`, return a clear error (e.g. 409 Conflict, "Arrival already recorded for this trip") rather than a generic 500.
- Return the created row (or at least its `id` and `server_timestamp`) on success.

---

## 3. Arrival event UI page

Create `src/app/events/arrival/page.tsx` (or fit into existing routing pattern from Day 1 — Antigravity's judgment on exact path given the driver-only login flow already in place).

- Only accessible to a logged-in driver (reuse Day 1's session/auth check).
- Photo is **mandatory** for Arrival (per Core Item #4 — enforce this in the UI: disable submit until a photo is selected).
- Flow:
  1. On page load or button press, capture GPS (`getGpsLocation.ts` from Day 2).
  2. Fetch server timestamp (`getServerTime.ts` from Day 2).
  3. User selects/captures photo, uploads via `uploadPhoto.ts` (Day 2) to get `photo_url`.
  4. On submit, POST all captured data to `/api/events/arrival`.
  5. On success, show a clear confirmation (e.g. "Arrival recorded" + timestamp + a thumbnail of the uploaded photo).
  6. On failure (including the 409 duplicate case), show a clear, specific error — not a generic failure message.
- Which `trip_id` to use: pull from the driver's currently pre-seeded/active trip (reuse whatever Day 1 already established for identifying "this driver's current trip" — do not invent a new trip-selection mechanism).

---

## Out of scope (do not touch)

- Check-in and Departure events
- Timeline view
- AI evidence summary
- The `/test-day2` test page — leave it as-is for now, do not delete yet (useful if Day 4/5 needs quick utility re-verification). Can be cleaned up later in a dedicated cleanup pass.

---

## After implementation

Report back as a file (not pasted in chat):
- Files created/modified
- Migration applied confirmation (table + grants check)
- Build result
- Any deviation from this instruction and why
- Save to `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day3_ArrivalEventFlow.md`

Ayush will do manual browser verification (full arrival flow: capture GPS → capture photo → submit → see confirmation; plus test the duplicate-arrival 409 case) before Day 3 is marked done — per evidence rule.

---

## Reminder (per locked decisions)

- All writes go through Next.js API routes with `service_role`, server-side only.
- `events` table is immutable at the DB level — no UPDATE/DELETE path exists or should be built, for anyone.
- `server_timestamp` must come from Day 2's server-time util — never `new Date()` on the client.
