# Chat37 — Node7 Phase1a — Public Share AI Summary — DB + Code Reconciliation Investigation

## Status
INVESTIGATION AUTHORIZED — DIAGNOSIS ONLY

## Context
The Public Share flow has now been corrected for the previously verified trip/event schema mismatches. Production manual verification currently shows:

- Public Share opens successfully.
- Status shows `Completed`.
- Delivery Date is populated.
- Pickup and destination are populated.
- Evidence Status is `COMPLETE`.
- Arrival, check-in, and departure evidence are checked.
- Event Timeline shows the three delivery milestones with timestamps.
- AI Evidence Summary still displays `AI summary unavailable.`

This investigation must determine why the AI summary remains unavailable.

## Primary Objective
Trace the complete AI summary path using production evidence and current source code, identify the first divergence, and isolate the exact root cause without guessing.

## Required Trace

```text
Production Public Share
→ getPublicVerificationData()
→ production event rows
→ public event projection
→ generateSummaryForEvents()
→ summary input
→ AI/provider/configuration path
→ summary result/error
→ public projection
→ API response
→ Public Share page
→ rendered AI summary
```

## 1. Production Database Evidence
Using the exact production Public Share/trip currently being manually verified:

- verify the referenced trip and ACTIVE public-share row
- inspect the actual event rows used by the Public Share
- verify canonical event types
- verify `server_timestamp` values
- verify that the required delivery milestone events exist
- inspect fields required by summary generation
- inspect any trip/company fields used as summary context
- determine whether any summary-specific persisted field exists and whether the current implementation uses it
- do not modify the database
- never record the raw public token or token hash

## 2. Current Source Code
Inspect the deployed/current relevant implementation, including at minimum:

- `src/lib/public-share-lookup.ts`
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/share/[token]/page.tsx`
- `src/app/api/summary/route.ts`
- `src/lib/summary.ts`
- any direct AI/provider client or configuration used by summary generation
- relevant environment/config references, without exposing secret values
- event creation/normalization code if needed to determine summary input shape

For each stage identify:

- exact function called
- exact input object/array shape
- required event types/fields
- summary eligibility/completeness conditions
- AI/provider invocation path
- success return shape
- failure/null/error behavior
- public projection field carrying the summary
- UI condition that renders `AI summary unavailable.`

## 3. Summary Function Trace
Determine exactly:

1. Whether `getPublicVerificationData()` calls `generateSummaryForEvents()`.
2. What event objects are passed to it.
3. Whether the current three production delivery events satisfy the function's requirements.
4. Whether the summary function returns a string, null, undefined, fallback, or throws.
5. Whether the AI/provider is actually invoked.
6. If invoked, whether the invocation succeeds or fails.
7. Whether provider/configuration errors are swallowed.
8. Whether the generated summary is included in the public projection.
9. Whether the Public Share page receives the field.
10. Whether the UI is reading the same field.

## 4. AI/Provider Investigation
Determine the exact runtime state relevant to summary generation:

- provider/model path actually used
- required environment/configuration presence, but do not expose secret values
- whether the deployed production runtime can invoke the provider
- response/error status if observable
- whether an API key/configuration failure is occurring
- whether rate limits, model errors, malformed input, or network/runtime errors occur

Do not make unnecessary external AI calls. A focused production/runtime check is permitted only if required to distinguish code-path behavior from configuration/runtime failure.

## 5. Graceful Degradation Check
The architecture intentionally allows graceful AI degradation.

Determine whether `AI summary unavailable.` means:

- legitimate AI unavailable state,
- summary generation was never attempted,
- summary generation failed,
- summary generated but was lost during projection,
- or UI is incorrectly displaying the unavailable state despite a valid summary.

Do not treat graceful degradation as the root cause itself.

## 6. Error Masking Check
Identify every point where a summary error can become:

- `null`
- `undefined`
- fallback text
- empty string
- HTTP 200 with AI status unavailable
- a generic error hidden from the UI

Distinguish:

`AI unavailable by design`

from

`AI summary path is broken`

and from

`AI summary exists but public projection/UI loses it`.

## 7. Expected-vs-Actual Table
The report MUST contain:

| Stage | Expected | Actual production/runtime | Result | Confidence |
|---|---|---|---|---|
| Public share | | | | |
| Trip/events | | | | |
| Summary eligibility | | | | |
| Summary input | | | | |
| `generateSummaryForEvents()` call | | | | |
| AI/provider invocation | | | | |
| Provider response | | | | |
| Summary result | | | | |
| Public projection | | | | |
| API response | | | | |
| Page receipt | | | | |
| UI rendering | | | | |

Allowed confidence labels:

- VERIFIED
- INFERRED
- UNKNOWN

## 8. Root Cause Discipline
Identify the FIRST divergence in the chain.

If multiple independent defects exist, list each separately and state exactly which output it blocks.

Possible affected outputs:

- AI summary generation
- AI summary public projection
- AI summary API response
- AI summary page rendering

Do not collapse multiple defects into one root cause.

## 9. No Changes
This is diagnosis-only.

Absolutely no:

- source-code edits
- database edits
- schema changes
- migrations
- deployment changes
- pushes
- fixes

## 10. Required Investigation Report
Create/update:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare_AI_Summary_DB_Code_Reconciliation_Report.md`

The report MUST include:

- status
- scope
- exact production DB evidence
- source-code trace
- summary function trace
- AI/provider runtime findings
- error/fallback behavior
- expected-vs-actual table
- first divergence
- each root cause with confidence
- what is already correct
- what remains broken
- recommended next decision
- explicit no-change statement

Do not include secret values, API keys, raw tokens, or token hashes.

## Stop Condition
Stop after writing the investigation report. Do not implement a fix.
