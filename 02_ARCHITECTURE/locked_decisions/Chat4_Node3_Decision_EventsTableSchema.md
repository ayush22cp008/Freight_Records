# Chat4_Node3_Decision_EventsTableSchema.md

**Project:** Freight — AI Builders Hackathon
**Chat:** #4 | **Node:** 3 (Build execution) | **Decided:** Aug 21, 2026 (before Day 3)
**Status:** LOCKED — do not re-discuss without explicit reason

---

## Decision

Final schema for the `events` table (Arrival/Check-in/Departure evidence records):

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

## Key decisions and rationale

1. **Fixed 3 event types for MVP** (`arrival` / `checkin` / `departure`) — one row per event type per trip. N-events flexibility (repeatable "Add Evidence") is explicitly deferred to Stretch #5 per ROADMAP.md, not part of core scope.

2. **True DB-level immutability — REVOKE UPDATE/DELETE from ALL roles, including `service_role`.**
   Rationale: core product pitch is "evidence notary that cannot be tampered with." App-level discipline (just never writing update/delete code) is not a real guarantee — DB-level grants revocation is. This directly strengthens the "technical implementation" story for judging.
   Trade-off accepted: if a genuine bug inserts bad data, fixing it requires a tracked migration step (`GRANT UPDATE, DELETE ...` temporarily, fix, `REVOKE` again) — not a quick manual fix. Ayush explicitly accepted this trade-off.

3. **`event_type` as `text + CHECK` constraint, not Postgres ENUM.**
   Rationale: ROADMAP Stretch #5 already anticipates new event types/schema changes later. CHECK constraints are altered with a simple `DROP CONSTRAINT` / `ADD CONSTRAINT`, whereas Postgres ENUMs have `ALTER TYPE ... ADD VALUE` restrictions inside transactions. CHECK is the lower-friction choice for a schema expected to evolve.

4. **`(trip_id, event_type)` UNIQUE constraint** — prevents duplicate event rows per trip (e.g. two "arrival" inserts) at the DB level, not just app-level validation.

5. **`server_timestamp` (from Day 2's `/api/server-time` util) is the source of truth for evidence timing — never client device clock.** `created_at` (DB default `now()`) is kept as a secondary/backup timestamp, not the primary evidence field.

## Scope note

This schema is for Day 3 (Event 1 — Arrival) implementation. Migration file to be created and applied as part of the Day 3 Antigravity instruction, before the Arrival event flow is wired.
