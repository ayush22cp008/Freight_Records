# Chat47 — Day 20 — Node 7 — Phase 1b
# Reviewer Current-Evidence Selection Implementation

**Status:** IMPLEMENTATION AUTHORIZED  
**Day:** Day 20  
**Chat:** Chat47  
**Node:** Node 7 — AI + Final Integration + Demo  
**Executor:** Antigravity  
**Architecture / authorization:** ChatGPT + Ayush  

## 1. Governing Decision

Implement only the scope authorized by:

```text
00_PROJECT_CONTROL/DECISIONS/Chat47_Day20_Node7_Phase1b_Reviewer_Current_Evidence_Selection_Governance_Decision.md
```

The decision establishes the authoritative current-evidence rule for Reviewer Verify:

```text
For a PENDING applicant:
1. Find all onboarding_evidence rows for the applicant.
2. Consider only rows with status = PENDING.
3. Select the newest/current PENDING evidence deterministically.
4. Present that selected row to the Reviewer.
5. Preserve historical REJECTED evidence and reviewer decisions unchanged.
```

## 2. Confirmed Defect

The current Reviewer Verify server page assumes exactly one evidence row using an `onboarding_evidence` query with:

```text
.eq('auth_id', identity.auth_id)
.single()
```

Recovery legitimately leaves the historical rejected evidence row and creates a new pending row. For a recovered applicant this produces multiple rows, so the current Verify page receives no usable evidence record. The client then renders `No evidence document found`, and `Load Evidence` cannot create a signed URL because no evidence/storage path is supplied.

## 3. Authorized Change

Modify only the Reviewer Verify evidence lookup in:

```text
src/app/(authenticated)/reviewer/verify/[id]/page.tsx
```

The query must no longer assume exactly one evidence row for the applicant.

Implement deterministic selection of the newest `PENDING` evidence for the authenticated identity's `auth_id`.

The resulting selected record must provide the existing client with the normal `EvidenceRecord` shape, including at minimum:

```text
id
auth_id
document_type
storage_path
status
rejection_reason
mime_type
```

The existing `ApplicantVerificationClient` evidence viewer flow should remain intact unless a minimal type-safe adjustment is strictly required to consume the selected record.

## 4. Selection Semantics

Use a server-side query that is explicit about:

```text
.eq('auth_id', identity.auth_id)
.eq('status', 'PENDING')
.order('created_at', { ascending: false })
```

Select one record deterministically.

Use a safe one-row retrieval pattern appropriate for a query that has been narrowed to the current PENDING evidence, while avoiding the old failure mode caused by multiple historical evidence rows.

Do NOT select the historical `REJECTED` row.

Do NOT replace/delete/update the historical evidence row.

Do NOT create any reviewer decision during this lookup.

## 5. Error / Empty-State Behavior

If there is genuinely no PENDING evidence row for an otherwise PENDING applicant, preserve a safe empty/error state rather than selecting a rejected historical record.

Do not silently select arbitrary evidence.

Do not change applicant status as part of this read path.

## 6. Explicit Non-Scope

Do NOT modify:

```text
- Applicant re-upload submission flow
- /api/onboarding/submit
- Reviewer Queue eligibility/filter behavior
- Queue document-label behavior
- Reviewer History UI or data model
- Reviewer decision mutation endpoint
- RLS/security architecture
- evidence versioning schema
- Driver portal behavior
- Company portal behavior
- unrelated `.single()` / `.find()` calls
```

Do not broaden the task beyond the exact current-evidence selection defect.

## 7. Required Validation

### Static/source validation

Confirm the Reviewer Verify evidence lookup no longer uses the invalid applicant-wide `.single()` assumption.

Confirm the query explicitly targets `status = PENDING` and deterministic newest ordering.

### Build/test

Run the project's standard validation/build/typecheck/lint commands appropriate to the repository.

Report any failures and do not conceal unrelated pre-existing failures.

### Runtime/manual verification target

Using the same recovered Driver applicant from the Day 20 investigation:

```text
historical evidence v1 = REJECTED
current evidence v2 = PENDING
identity = PENDING
reviewer decisions = 1 historical decision
```

Verify:

```text
Reviewer Queue
→ applicant appears

Reviewer Verify
→ Pending Verification
→ Document label is derived from current PENDING evidence
→ no “No evidence document found” message

Click Load Evidence
→ a signed URL request is made
→ current PENDING evidence is displayed

Historical evidence v1
→ remains preserved

Historical reviewer decision
→ remains preserved

No new reviewer decision
→ created by merely loading evidence
```

### Critical regression condition

If the selected evidence is the historical REJECTED row instead of the current PENDING row, the implementation is incorrect.

## 8. Commit / Handoff Requirements

Do not assume the change is pushed or deployed unless the actual workflow confirms it.

After implementation, create an implementation report under:

```text
03_IMPLEMENTATION/implementation_reports/Chat47_Day20_Node7_Phase1b_Reviewer_Current_Evidence_Selection_Implementation_Report.md
```

The report must contain:

```text
1. Files changed
2. Exact logic changed (summary, not full source dump)
3. Why the change implements the governance decision
4. Build/typecheck/lint/test results
5. Runtime verification evidence gathered
6. Any limitations or unresolved issues
7. Git commit SHA / deployment reference when available
8. Confirmation that non-scope areas were not changed
```

Do not create additional architecture/governance decisions unless a genuine unexpected blocker or scope-changing issue is discovered. If such an issue occurs, stop and report it rather than inventing a workaround.

## 9. Postflight

Before reporting completion, confirm:

```text
- only authorized Reviewer Verify evidence-selection code was changed
- historical evidence remains intact
- reviewer decisions remain intact
- no Reviewer decision is created by evidence loading
- validation/build results are reported honestly
- deployment/push status is explicitly identified
- implementation report is saved at the required Records path
```
