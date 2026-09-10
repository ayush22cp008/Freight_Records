# Chat47 — Day 20 — Node 7 — Phase 1b
# Re-upload Root-Cause Evidence Collection Follow-up

**Status:** EVIDENCE COLLECTION ONLY — NO IMPLEMENTATION AUTHORIZED  
**Day:** Day 20  
**Chat:** Chat47  
**Purpose:** Collect the missing direct evidence required to validate or reject the existing Chat47 root-cause investigation.

## Objective

The existing Chat47 investigation report claims that the rejected-applicant re-upload flow:

```text
re-upload
→ /api/onboarding/submit
→ evidence/status update
→ Complete Onboarding redirect
→ Queue omission
→ History presentation
```

However, the evidence artifacts referenced by that report are not currently present in `04_TESTING/evidence/`. Do not treat the report's unbacked claims as VERIFIED. Collect the actual evidence for one controlled rejected Driver applicant flow.

## Non-negotiable rules

- Investigation/evidence collection only.
- Do NOT modify application source code.
- Do NOT modify migrations, schema, RLS, storage policies, or production records.
- Use read-only database inspection except for the applicant's normal UI re-upload action.
- Do NOT manually reject the applicant again after re-upload.
- Do NOT manufacture or alter evidence to make a hypothesis pass.
- Record actual outputs, timestamps, environment, and test identity.
- Clearly classify conclusions as VERIFIED / INFERRED / UNKNOWN / BLOCKED.

## Evidence sequence

### 1. Pre-submit database snapshot

Immediately before pressing Submit Evidence on the Re-upload Evidence page, capture the applicant's actual live state.

Capture from `freight_identities`:

```text
id
auth_id
requested_role
trusted_role
verification_status
reviewed_at
created_at
```

Capture ALL matching `onboarding_evidence` rows:

```text
id
auth_id
role_type
document_type
storage_path
mime_type
size_bytes
version
status
rejection_reason
created_at
```

Capture ALL matching `reviewer_decisions` rows:

```text
id
identity_id
auth_id
evidence_id
decision
rejection_reason
reviewed_at
created_at (if present)
```

Save the exact read-only output.

### 2. Browser/API evidence during re-upload

Perform exactly:

```text
Rejected applicant
→ Re-upload Evidence
→ choose NEW evidence
→ Submit Evidence
```

Do not perform another Reviewer decision.

Capture the `/api/onboarding/submit` request and response:

```text
request method
request URL
request payload (redact secrets/tokens)
HTTP status
response body
response headers relevant to redirect/cache if useful
browser console errors, if any
navigation/redirect target
```

If DevTools provides network timing, preserve it where useful.

### 3. Post-submit database snapshot

Immediately after submission and before any further Reviewer action, capture the same fields from:

```text
freight_identities
onboarding_evidence
reviewer_decisions
```

The comparison MUST explicitly answer:

```text
Did a new onboarding_evidence row appear?
Did the new row have the expected document_type?
Did its status become PENDING?
Did identity verification_status become PENDING?
Did old evidence remain present?
Did reviewer_decisions row count change?
Did any new REJECTED decision appear?
```

### 4. Storage evidence

Verify whether the new evidence object's actual storage path exists and matches the new evidence row.

Capture:

```text
bucket
storage path
object existence
relevant metadata if available
```

### 5. Reviewer Queue evidence

Without performing any Reviewer decision, open the Reviewer Queue.

Capture:

```text
whether the applicant appears
status shown
role shown
evidence/document label shown
evidence reference shown if available
```

Then inspect the exact current Queue query/source code and document the filters that determine visibility.

Do not modify Queue code.

### 6. Reviewer History evidence

Open Reviewer History without performing another decision.

Capture:

```text
what record(s) are displayed for the applicant
whether the original rejection is present
whether a second decision is present
whether rejection reason is shown
whether evidence is shown
whether evidence_id resolves to a real onboarding_evidence row
```

Then compare the UI to the actual `reviewer_decisions` rows.

## Decisive root-cause checks

The evidence must explicitly resolve these questions:

### A. Automatic second rejection

```text
reviewer_decisions BEFORE
vs
reviewer_decisions AFTER
```

If no new Reviewer decision exists, state that the re-upload did not create a second Reviewer decision in the observed test.

If a second decision exists, trace exactly which server operation created it. Do not infer causality without evidence.

### B. Recovery state transition

Determine whether:

```text
REJECTED
→ new evidence
→ PENDING
```

actually occurs in the live database.

### C. Complete Onboarding redirect

Determine whether the redirect is:

```text
normal rendering after successful PENDING transition
OR
incorrect fallback after failure/partial success
OR
another branch
```

### D. Queue omission

If identity/evidence are PENDING but Queue omits the applicant, capture the exact filter/query condition that excludes it and the live values that fail that condition.

### E. History anomaly

Determine whether History is showing:

```text
one original rejection only
OR
an actual second rejection
OR
an incorrect/incomplete representation of an existing decision
```

## Required evidence files

Save the actual evidence under `04_TESTING/evidence/` using these exact paths:

```text
04_TESTING/evidence/Chat47_Day20_Reupload_API_Response.json
04_TESTING/evidence/Chat47_Day20_Reupload_DB_Before_After.sql
04_TESTING/evidence/Chat47_Day20_Reupload_Reviewer_Queue.png
04_TESTING/evidence/Chat47_Day20_Reupload_Reviewer_History.png
```

Also save any additional small evidence artifact only when genuinely necessary (for example, a browser console/network export). Do not create filler artifacts.

## Evidence manifest

Create/update a concise evidence manifest only if necessary to explain how the four artifacts map to the test attempt. Record:

```text
applicant/test identity (non-secret identifier)
test date/time
environment / URL
code commit observed, if available
artifact path
what the artifact proves
limitations
```

## Final result

After collecting the evidence, update the existing Chat47 investigation report only if appropriate; otherwise create the minimum necessary follow-up record. Do not declare a root cause VERIFIED merely because the earlier report said so.

The final evidence reconciliation must distinguish:

```text
VERIFIED — directly supported by collected evidence
INFERRED — plausible but not directly proven
UNKNOWN — insufficient evidence
BLOCKED — requires unavailable access/action
```

Do not create an implementation prompt from these results. Implementation requires a separate ChatGPT review and explicit governance decision.
