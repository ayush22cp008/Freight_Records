# Chat37 — Node7 Phase1a — Public Share Evidence — DB + Code Reconciliation Investigation Prompt

## Role
You are executing a diagnosis-only investigation for the Freight project. Do not implement fixes.

## Primary Objective
Reconcile the actual production database evidence with the current application code for the Public Share flow. The user has explicitly requested investigation of BOTH database and code because the Public Share currently does not show the expected evidence/timestamps/delivery date/AI summary.

Do not assume the previously identified schema mismatch is the only defect.

## Investigation Record
Follow:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_Evidence_DB_Code_Reconciliation_Investigation.md`

## Required End-to-End Trace

```text
Create Public Share
→ ACTIVE DB row
→ token/hash lookup
→ trip lookup
→ event rows
→ event types
→ timestamp fields
→ evidence completeness
→ delivery date
→ AI summary input
→ public projection
→ API response
→ Public Share page
```

## 1. Production DB
Using the exact production test trip/public share involved in the current failure:

- verify `trip_public_shares` row and ACTIVE status
- verify referenced trip
- inspect actual trip fields relevant to public projection
- inspect every event for that trip
- record exact event types
- compare `server_timestamp` vs `timestamp` if both exist
- inspect location/evidence/photo fields
- verify actual event sequence
- verify relevant trip completion/delivery timestamp fields
- do not modify DB

Never record the raw token or token hash in the report.

## 2. Source Code
Inspect the current deployed production commit and current relevant source files:

- `src/lib/public-share-lookup.ts`
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/share/[token]/page.tsx`
- `src/app/api/summary/route.ts`
- `src/lib/summary.ts`
- public-share creation route
- event creation routes
- relevant Node5 schema/migration files
- relevant timeline UI

For each relevant path, identify:

- expected event types
- expected timestamp column
- evidence-completeness rules
- delivery-date source
- public projection fields
- AI-summary input
- error handling/fallbacks

## 3. Important Vocabulary Check
Explicitly compare the code against the canonical event vocabulary in the Node5 migration:

- `ARRIVED_AT_DELIVERY`
- `RECEIVER_CHECKED_IN`
- `DELIVERY_DEPARTED`

Also identify any legacy values actually present in the DB.

Do not assume names are correct merely because they look semantically similar.

## 4. Timestamp Check
Explicitly determine:

- which timestamp field event creation writes
- which timestamp field authenticated Timeline reads
- which timestamp field Public Share reads
- which timestamp field deliveryDate uses
- whether the selected field actually contains values in production

This is critical because the current Public Share helper appears to use `timestamp`, while other event paths use `server_timestamp`.

## 5. Evidence Check
Determine whether the Public Share helper's completeness logic can ever become COMPLETE for the actual production event vocabulary.

If the DB contains canonical Node5 events but the helper checks different names, mark that as a separate verified code/data mismatch.

## 6. AI Summary Check
Trace exactly what events are supplied to `generateSummaryForEvents()` and whether the summary path can execute for the production event sequence.

Do not call external AI services unnecessarily. Static/code-path analysis is sufficient unless a safe focused runtime test is required.

## 7. Error Masking Check
Identify every point where a database/query error can become:

- `null`
- empty array
- fallback text
- generic 404

Distinguish a genuinely missing share from a downstream data-resolution failure.

## 8. Expected-vs-Actual Table
Your report MUST contain:

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

Allowed confidence labels:

- VERIFIED
- INFERRED
- UNKNOWN

## 9. Root Cause Discipline
Identify the FIRST divergence in the chain.

If there are multiple independent defects, list them separately and state which one blocks each missing output:

- timeline/evidence
- timestamp
- delivery date
- AI summary
- overall Public Share rendering

Do not collapse multiple defects into one root cause.

## 10. No Changes
Absolutely no:

- source-code edits
- DB edits
- schema changes
- migrations
- deployment changes
- pushes
- fixes

## 11. Required Report
Create/update:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_Evidence_DB_Code_Reconciliation_Report.md`

Include:

- status
- scope
- DB findings
- code findings
- runtime/API findings
- expected-vs-actual table
- first divergence
- each root cause with confidence
- what is already correct
- what is broken
- recommended next decision
- explicit no-change statement

## Stop
Stop after writing the investigation report. Do not implement any fix.
