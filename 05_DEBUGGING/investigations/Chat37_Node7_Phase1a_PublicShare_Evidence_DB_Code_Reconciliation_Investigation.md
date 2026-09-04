# Chat37 — Node7 Phase1a — Public Share Evidence — DB + Code Reconciliation Investigation

## Status
AUTHORIZED — INVESTIGATION ONLY

## Objective
Investigate the production Public Share evidence mismatch by reconciling the actual database evidence with the current source-code expectations. Do not assume the previously identified schema mismatch is the only remaining issue.

## Problem
The production Public Share page has been observed not to display the expected evidence/timestamps/delivery date/AI summary. A previous investigation isolated a `trips` column-name mismatch, but the current code must now be checked against the actual event/database vocabulary and populated fields end-to-end.

## Required Investigation Chain

```text
Production DB evidence
  ↓
Trip + events + timestamps + evidence fields
  ↓
Event creation routes
  ↓
Public-share lookup helper
  ↓
Public verification API
  ↓
Public Share page
  ↓
Evidence state / timeline / delivery date / AI summary
```

## A. DATABASE INVESTIGATION

Using the exact production test trip/public-share created for this investigation:

1. Verify the `trip_public_shares` row exists and is `ACTIVE`.
2. Verify its `trip_id` references the expected trip.
3. Verify the trip exists and record the actual relevant columns/values without exposing secrets.
4. Inspect all events for the trip:
   - exact `event_type` values
   - `server_timestamp`
   - `timestamp` if present
   - location fields
   - photo/evidence fields
   - ordering
5. Verify the actual delivery evidence sequence present in production.
6. Verify whether the production DB uses the canonical Node5 event vocabulary:
   - `ARRIVED_AT_DELIVERY`
   - `RECEIVER_CHECKED_IN`
   - `DELIVERY_DEPARTED`
   and identify any legacy values actually present.
7. Verify relevant trip completion/delivery timestamp fields if present.
8. Do not modify any DB records or schema.

## B. CODE INVESTIGATION

Inspect the current source at the deployed production commit and relevant current main branch files, including:

- `src/lib/public-share-lookup.ts`
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/share/[token]/page.tsx`
- `src/app/api/summary/route.ts`
- `src/lib/summary.ts`
- relevant event creation routes
- relevant trips/public-share creation route
- Node5 migration/schema files
- relevant timeline UI implementation

For each, determine:

1. Which event types are expected?
2. Which timestamp column is queried/read?
3. Which event types are used to calculate evidence completeness?
4. Which event determines `deliveryDate`?
5. Which fields are projected to the public response?
6. Which fields are used as AI-summary input?
7. Which DB/query errors are swallowed as `null`, empty arrays, or fallback values?
8. Whether code expectations match the actual DB schema and production rows.

## C. EXACT RECONCILIATION

Produce a stage-by-stage table:

| Stage | Expected by code | Actual production DB/runtime | Result | Confidence |
|---|---|---|---|---|
| Public share row | | | | |
| Token/hash lookup | | | | |
| Trip lookup | | | | |
| Event retrieval | | | | |
| Event vocabulary | | | | |
| Timestamp field | | | | |
| Evidence completeness | | | | |
| Delivery date | | | | |
| AI summary input | | | | |
| Public projection | | | | |
| API response | | | | |
| Page rendering | | | | |

Use only:
- VERIFIED
- INFERRED
- UNKNOWN

## D. FIRST DIVERGENCE

Identify the earliest stage where the actual DB/runtime behavior differs from what the code expects.

Do not declare a root cause until the DB evidence and code evidence agree.

If multiple independent defects exist, list each separately and identify which one blocks the expected public evidence output.

## E. SECURITY / PRIVACY

- Never record the raw public token.
- Never record token hashes.
- Do not expose credentials, API keys, service-role keys, or private user data.
- Use identifiers only when necessary and redact them appropriately.

## F. NO-CHANGE RULE

This is diagnosis only.

Do NOT:
- modify source code
- modify database records
- modify database schema
- create migrations
- change deployment configuration
- push or deploy
- fix the issue during investigation

## G. REQUIRED OUTPUT

Create:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_Evidence_DB_Code_Reconciliation_Report.md`

The report must contain:

1. Status
2. Exact investigation scope
3. DB findings
4. Code findings
5. Runtime/API findings
6. Expected-vs-actual reconciliation table
7. First divergence
8. Root cause(s), each with confidence
9. What is already correct
10. What is broken/misaligned
11. Recommended next decision
12. Explicit no-change statement

## Stop Condition
Stop after the investigation report is complete. Do not implement any fix as part of this task.
