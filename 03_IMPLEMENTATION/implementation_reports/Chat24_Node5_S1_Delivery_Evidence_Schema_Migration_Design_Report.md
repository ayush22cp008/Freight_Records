# Chat24 — Node 5 Subnode 5.S1 — Delivery Evidence Schema Migration Design Report

## 1. Subnode Status
READY FOR CHATGPT REVIEW

## 2. Why 5.S1 Was Created
The Chat24 Node 5 source investigation found that the existing `events` schema explicitly limits `event_type` to three specific pickup-related strings (`arrival`, `checkin`, `departure`) and enforces a rigid `UNIQUE (trip_id, event_type)` constraint. This makes it impossible to record delivery-side events (like arrival at destination) because a second 'arrival' event would violate the constraint. This Subnode investigates the safest schema migration strategy to support the locked Node 5 expanded lifecycle without breaking existing history.

## 3. Records Baseline
- `00_PROJECT_CONTROL/ROADMAP.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`
- `03_IMPLEMENTATION/implementation_reports/Chat24_Node5_Current_Source_Investigation_Report.md`
- `02_ARCHITECTURE/locked_decisions/Chat4_Node3_Decision_EventsTableSchema.md`

## 4. Source Baseline
- **Project Root**: `c:\Users\ayush\Desktop\Freight_hackathon`
- **Source Repository**: `ayush22cp008/freight_hackathon`
- **Current Branch**: `main`
- **Commit SHA**: `becf3175a1fe266ce7d81eb3fb7ec2124526493b`

## 5. Current `events` Schema
- **Definition**: Contains `id` (uuid PK), `trip_id`, `driver_id`, `latitude`, `longitude`, `server_timestamp`, `photo_url`, `created_at`.
- **Constraint (event_type)**: `CHECK (event_type IN ('arrival', 'checkin', 'departure'))`
- **Uniqueness**: `UNIQUE (trip_id, event_type)`
- **RLS**: Enabled, with UPDATE/DELETE explicitly revoked from all roles to ensure immutability.

## 6. Current `trips` Status Schema
- **Constraint (status)**: `CHECK (status IN ('active', 'draft', 'published', 'claimed', 'in_progress', 'completed'))`
- **Missing**: No `delivered` or `in_transit` statuses are defined.

## 7. Existing Data Compatibility
- Existing historical `arrival`, `checkin`, and `departure` explicitly represent the **pickup** phase of a trip.
- Renaming historical event values directly in the database (e.g., from `arrival` to `pickup_arrival`) will break the existing UI because `src/app/(authenticated)/events/arrival/page.tsx` and `src/app/api/events/arrival/route.ts` hardcode `.eq('event_type', 'arrival')`.
- Similarly, `page.tsx` and `timeline/page.tsx` directly render and query these exact string values. 

## 8. Event Uniqueness Analysis
- The `UNIQUE (trip_id, event_type)` rule is a core protection against duplicate/replay submissions (e.g., a driver mashing the "Arrive" button).
- **Target Design**: Removing the constraint completely is unsafe. We should retain `UNIQUE (trip_id, event_type)` but expand the vocabulary of `event_type` so that pickup and delivery events are distinct values. This preserves exact duplicate/replay protection while supporting the expanded lifecycle.

## 9. Event Vocabulary Analysis
- The Node 1 locked architecture uses conceptual names (e.g. `ARRIVED_AT_PICKUP`). 
- **Proposed Persisted Vocabulary**:
  - `pickup_arrival` (maps to historical `arrival`)
  - `pickup_checkin` (maps to historical `checkin`)
  - `goods_loaded`
  - `pickup_departure` (maps to historical `departure`)
  - `delivery_arrival`
  - `receiver_checkin`
  - `goods_unloaded`
  - `delivery_departure`

## 10. Proposed Target Schema Contract
- **Events CHECK Constraint**: Expanded to `IN ('arrival', 'checkin', 'departure', 'pickup_arrival', 'pickup_checkin', 'goods_loaded', 'pickup_departure', 'delivery_arrival', 'receiver_checkin', 'goods_unloaded', 'delivery_departure')`. (Note: Old values retained for backward compatibility).
- **Uniqueness**: Retain `UNIQUE (trip_id, event_type)`.
- **Trips CHECK Constraint**: Expanded to `IN ('active', 'draft', 'published', 'claimed', 'in_progress', 'in_transit', 'delivered', 'completed')`.

## 11. State vs Event Responsibility
- `trips.status` acts as the definitive real-time state machine of the delivery.
- `events` act as immutable proof/audit trails.
- Confirmations (`driver_completion`, `receiver_confirmation`) should be fields on the `trips` table (e.g. boolean flags or timestamps) because they represent a final state agreement rather than a geographic GPS-stamped event, avoiding misuse of the `events` table.

## 12. RLS / Authorization Compatibility
- RLS policies on `events` (INSERT allowed, UPDATE/DELETE denied) remain 100% valid.
- The `driver_id` ownership logic still applies.
- A new RLS policy or server-side authorization check will be needed later in Node 5 to allow the Receiving Company to insert their specific events (`receiver_checkin`). This does not require changing the migration.

## 13. Migration Sequence Design
1. **Schema Update**: Drop existing `CHECK` constraints on `events` and `trips` and recreate them with the expanded vocabularies.
2. **Data Preservation**: Do NOT mutate existing historical `arrival`, `checkin`, `departure` rows. Leave them intact.
3. **Application Update (Future Node 5 Task)**: Update the API routes and frontend UI to insert and query the new specific event names (`pickup_arrival` instead of `arrival`), while gracefully falling back to rendering legacy event strings in the timeline.

## 14. Backward Compatibility / Historical Data Preservation
- By retaining the old strings in the new `CHECK` constraint and not mutating historical rows, old trips remain perfectly intact.
- The UI timeline logic will need a minor mapping function to render both `arrival` (legacy) and `pickup_arrival` (new) uniformly.

## 15. Future Validation / Test Requirements
- Verify migration applies cleanly on local Supabase schema.
- Verify old trips still render their timeline.
- Verify new trips can successfully record `pickup_arrival` AND `delivery_arrival` without uniqueness conflicts.
- Verify duplicate `pickup_arrival` submissions are rejected by the database.

## 16. Risks and Rollback Considerations
- **Risk**: If the application is updated before the migration is applied, inserts of `pickup_arrival` will fail the old `CHECK` constraint.
- **Rollback**: Because we are expanding the constraint rather than destroying data, the migration is cleanly reversible (assuming no new data has been written yet).

## 17. Exact Decisions Required Before Implementation
- Confirm the proposed strategy: leaving historical event strings in place while requiring the application code to shift to the new vocabulary (`pickup_arrival`, etc.) for all new trips.

## 18. Subnode Exit Criteria
- Current schema/incompatibilities evidenced.
- Target schema contract explicit.
- Uniqueness and historical preservation strategy explicit.
- Ready for ChatGPT Review.

## 19. Evidence Index
- `src/db/migrations/002_create_events_table.sql`
- `src/db/migrations/006_node3_trip_schema.sql`
- Existing application queries (`src/app/api/events/arrival/route.ts`)

## 20. VERIFIED / INFERRED / UNKNOWN Summary
- **VERIFIED**: `events` strict uniqueness constraint blocks Node 5.
- **VERIFIED**: Source heavily depends on exact string values `arrival`, `checkin`, `departure`.
- **INFERRED**: Expanding CHECK constraints while preserving historical rows is the safest zero-downtime migration path.

## 21. Explicit Non-Changes
```text
Application source modified = NO
Database schema modified = NO
Tests added = NO
Production/shared data changed = NO
Commit = NO
Push = NO
Ayush manual verification = NOT PERFORMED
Implementation = NO
```
