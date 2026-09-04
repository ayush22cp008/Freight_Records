# Chat37 — Node7 Phase1a — Public Share Evidence Schema/Timestamp Fix — Antigravity Prompt

## Authorization
IMPLEMENTATION AUTHORIZED by Ayush after review of the completed DB + code reconciliation investigation.

Investigation record:
`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_Evidence_DB_Code_Reconciliation_Investigation.md`

Investigation report:
`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_Evidence_DB_Code_Reconciliation_Report.md`

## Objective
Fix the verified Public Share evidence/timeline/delivery-date/AI-summary defects in the smallest possible source-code change.

## Verified Root Causes
Production DB uses:
- `events.server_timestamp`
- `events.created_at`
- `ARRIVED_AT_DELIVERY`
- `RECEIVER_CHECKED_IN`
- `DELIVERY_DEPARTED`

Production DB does NOT have:
- `events.timestamp`
- `events.location_name`

Current `src/lib/public-share-lookup.ts` incorrectly:
- orders events by `timestamp`
- reads `e.timestamp`
- checks `DELIVERY_ARRIVED`
- checks `DELIVERY_CHECKIN`
- reads `e.location_name`

The investigation verified that `.order('timestamp')` causes PostgreSQL `42703`, the error is masked, and the public projection then receives an empty event list. This prevents timeline output, evidence completion, delivery date, and AI summary generation.

## Scope — ONLY THIS FIX
Modify the Public Share lookup implementation as necessary to align it with the verified production schema and canonical event vocabulary.

Required changes:

1. Change event ordering from:
   `timestamp`
   to:
   `server_timestamp`

2. Change event timestamp projection from:
   `e.timestamp`
   to:
   `e.server_timestamp`

3. Change delivery date derivation to use the `DELIVERY_DEPARTED` event's `server_timestamp`.

4. Change evidence/event vocabulary:
   - `DELIVERY_ARRIVED` → `ARRIVED_AT_DELIVERY`
   - `DELIVERY_CHECKIN` → `RECEIVER_CHECKED_IN`
   - preserve `DELIVERY_DEPARTED`

5. Ensure the key-event timeline includes the canonical delivery event sequence required by the existing Public Share design.

6. Replace the nonexistent `location_name` read with a safe projection based on actual available event fields. Do not invent a DB column. If the existing public UI only needs a human-readable fallback and location is not required for acceptance, preserve a safe fallback such as `Location recorded`. If actual latitude/longitude are already available and the current public contract supports them, use them without broadening scope.

7. Do not silently convert an event-query database error into a successful empty timeline. Preserve safe public behavior, but make the error observable in server logs and distinguish query failure from genuinely empty events. Do not redesign the API contract unless required by the existing implementation.

## Preserve Existing Behavior
Do NOT change:

- public-share token generation
- token hashing
- ACTIVE/revoked share semantics
- anonymous rate limiting
- public verification security boundary
- trip/company lookup semantics
- `facility_name` / `destination_name` mapping
- Promise-based route params
- public projection shape unless required by the timestamp/location fix
- AI summary implementation itself
- database schema
- database records
- migrations
- unrelated UI
- unrelated refactors

## Validation
After implementation:

1. Run TypeScript validation.
2. Run the production build.
3. Verify the changed Public Share code statically against the canonical Node5 schema/event vocabulary.
4. If safe local/runtime tests exist, run focused tests for:
   - event retrieval
   - canonical event filtering
   - delivery date
   - public projection
   - evidence completeness
5. Do not claim production verification unless it was actually performed.

## Required Implementation Report
Create:

`03_IMPLEMENTATION/implementation_reports/Chat37_Node7_Phase1a_PublicShare_EvidenceSchemaTimestampFix_Implementation_Report.md`

The report must state:

- status
- exact files changed
- exact defects fixed
- validation commands/results
- production verification status
- any remaining concerns
- whether push/deployment occurred

## Push Rule
Do NOT push or deploy as part of this task unless Ayush separately authorizes the push/deployment action. The implementation report must accurately state the repository/deployment state.

## Stop Condition
Stop after implementation and validation report. Do not perform unrelated fixes or begin Phase3 work.
