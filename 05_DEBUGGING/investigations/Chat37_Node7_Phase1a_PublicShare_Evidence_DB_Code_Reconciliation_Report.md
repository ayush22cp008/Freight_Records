# Chat37 — Node7 Phase1a — Public Share Evidence — DB + Code Reconciliation Report

**Status**: INVESTIGATION COMPLETE — MULTIPLE ROOT CAUSES ISOLATED

## 1. Scope
Reconcile actual production database evidence against the application code for the Public Share flow to determine why evidence, timestamps, delivery dates, and AI summaries are not appearing on the public page.

## 2. DB Findings (Verified in Production)
- **Event Vocabulary**: The database strictly uses `ARRIVED_AT_DELIVERY`, `RECEIVER_CHECKED_IN`, and `DELIVERY_DEPARTED` as event types.
- **Timestamp Column**: The database uses `server_timestamp` and `created_at`. There is no `timestamp` column on the `events` table.
- **Location Column**: The database contains `latitude`, `longitude`, and `gps_accuracy`. There is no `location_name` column.

## 3. Code Findings (src/lib/public-share-lookup.ts)
- **Order By Error**: The code attempts to query events using `.order('timestamp')`.
- **Vocabulary Mismatch**: The code filters for `DELIVERY_ARRIVED` and `DELIVERY_CHECKIN`, which do not match the database.
- **Timestamp Read**: The code maps `e.timestamp` into the public projection and uses it for `deliveryDate`.
- **Location Read**: The code attempts to map `e.location_name`.
- **Error Masking**: If `.order('timestamp')` throws a PostgreSQL `42703 column does not exist` error, `events` evaluates to `null`. The code masks this with `(events || [])`, silently collapsing the entire event timeline to an empty array without failing the API request.

## 4. Expected vs Actual

| Stage | Expected by code | Actual production DB/runtime | Result | Confidence |
|---|---|---|---|---|
| Public share row | ACTIVE | ACTIVE | SUCCESS | VERIFIED |
| Trip lookup | `trips` query succeeds | `trips` query succeeds | SUCCESS | VERIFIED |
| Event retrieval | Returns array ordered by `timestamp` | `42703 column events.timestamp does not exist` error | **FAIL** | VERIFIED |
| Error Handling | Data is resolved | Postgres error is masked as `events = []` | **FAIL** | VERIFIED |
| Event vocabulary | `DELIVERY_ARRIVED`, `DELIVERY_CHECKIN` | `ARRIVED_AT_DELIVERY`, `RECEIVER_CHECKED_IN` | **FAIL** | VERIFIED |
| Timestamp field | `e.timestamp` | `e.server_timestamp` | **FAIL** | VERIFIED |
| Evidence completeness | Becomes COMPLETE if key events exist | Always INCOMPLETE (empty array & bad vocabulary) | **FAIL** | VERIFIED |
| Delivery date | Uses `timestamp` | Missing completely | **FAIL** | VERIFIED |
| AI summary input | Generates if COMPLETE | NEVER generates (always INCOMPLETE) | **FAIL** | VERIFIED |
| Public projection | Populates timeline and summary | Timeline empty, summary unavailable | **FAIL** | VERIFIED |

## 5. First Divergence
The **FIRST divergence** occurs when the `events` table query executes `.order('timestamp', { ascending: true })`. Because the `timestamp` column does not exist in the database schema, Supabase throws an error, which causes the returned `data` array to be `null`.

## 6. Root Causes (Multiple independent defects block the data)
1. **Empty Timeline (Primary Root Cause)**: The query fails on `.order('timestamp')`. The failure is silently masked by `(events || [])`, immediately destroying the entire timeline, completeness state, and AI summary generation.
2. **Never COMPLETE (Secondary Root Cause)**: Even if the query succeeded, the code explicitly checks for `DELIVERY_ARRIVED` and `DELIVERY_CHECKIN`, while the database contains `ARRIVED_AT_DELIVERY` and `RECEIVER_CHECKED_IN`. This prevents the AI summary from ever generating.
3. **Null Timestamps (Tertiary Root Cause)**: Even if the events were mapped, the code reads `e.timestamp` (which is `undefined`) instead of `e.server_timestamp`. This causes the delivery date and all timeline entry dates to render as broken or `null`.

## 7. Recommended Next Decision
Implement a fix in `src/lib/public-share-lookup.ts` that:
1. Replaces `.order('timestamp')` with `.order('server_timestamp')`.
2. Replaces `DELIVERY_ARRIVED` and `DELIVERY_CHECKIN` with `ARRIVED_AT_DELIVERY` and `RECEIVER_CHECKED_IN` in the filtering/checklist logic.
3. Maps `e.server_timestamp` into the timeline and `deliveryDate` projections.
4. (Optional) Adjusts location mapping if necessary, or maintains the safe 'Location recorded' fallback.

## 8. Explicit No-Change Statement
No source code, database records, schema, or deployment configurations were modified during this investigation.
