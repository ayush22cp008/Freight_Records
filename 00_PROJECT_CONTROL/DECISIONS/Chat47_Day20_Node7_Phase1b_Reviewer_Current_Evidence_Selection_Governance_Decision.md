# Chat47 — Day 20 — Node 7 — Phase 1b
# Reviewer Current-Evidence Selection Governance Decision

**Status:** DECIDED — IMPLEMENTATION AUTHORIZATION BOUNDARY  
**Day:** Day 20  
**Chat:** Chat47  
**Node:** Node 7 — AI + Final Integration + Demo  
**Decision owner:** Ayush  
**Architecture / reasoning:** ChatGPT  

## 1. Problem Confirmed

Live runtime and database evidence established that rejected-applicant recovery preserves historical evidence while creating a new pending evidence record.

Observed state for the tested Driver applicant:

```text
Evidence v1 → REJECTED
Evidence v2 → PENDING
Identity     → PENDING
Reviewer decisions → 1 historical REJECTED decision
```

The Reviewer Verify server page currently assumes exactly one `onboarding_evidence` row for an applicant by using `.single()` against `auth_id`. Because recovery legitimately preserves the rejected historical row and creates a new pending row, the lookup fails to produce a single evidence record. The client therefore receives no evidence object, renders “No evidence document found”, and its Load Evidence handler returns before requesting a signed URL.

## 2. Governance Decision

The authoritative evidence-selection rule for the current Reviewer verification view is:

```text
For a PENDING applicant:

1. Find all onboarding_evidence rows for the applicant.
2. Consider only evidence rows with status = PENDING.
3. Select the newest/current PENDING evidence deterministically.
4. The selected row is the evidence presented for the current review.
5. Historical REJECTED evidence remains preserved and is not selected for the current review.
```

The existing project records already establish this lifecycle principle: recovery inserts a new `PENDING` evidence row and preserves the previous `REJECTED` row. The current review model uses the latest pending evidence as the current review target.

## 3. Scope of Authorization

The implementation may modify only the Reviewer Verify evidence lookup needed to implement the above current-evidence rule.

The implementation may:

- remove the invalid exactly-one-row assumption for Reviewer Verify evidence lookup;
- deterministically select the newest `PENDING` evidence record for the applicant;
- preserve the existing `ApplicantVerificationClient` evidence viewer flow once a valid evidence record is supplied;
- preserve historical rejected evidence and reviewer decision records.

The implementation must NOT:

- delete historical evidence;
- overwrite the old rejected evidence row;
- change the recovery data model;
- create reviewer decisions during applicant re-upload;
- change Reviewer Queue eligibility unless a separate decision authorizes it;
- redesign Reviewer History;
- change RLS/security architecture;
- change Driver/Company product behavior;
- introduce a new evidence versioning scheme;
- broaden this change into unrelated Reviewer fixes.

## 4. Evidence Supporting This Decision

The decision is based on:

```text
A. Live browser evidence:
   /api/onboarding/submit → HTTP 200
   response → success: true
   storage upload → HTTP 200

B. Live database evidence:
   identity.verification_status → PENDING
   onboarding_evidence → 2 rows
      version 1 → REJECTED
      version 2 → PENDING
   reviewer_decisions → 1 row

C. Live Reviewer runtime evidence:
   applicant appears in Queue
   Verify page says “No evidence document found”
   clicking Load Evidence produces no evidence-loading request

D. Current source evidence:
   Reviewer Verify `page.tsx` queries `onboarding_evidence`
   with `.eq('auth_id', identity.auth_id).single()`.
   Client `loadEvidence()` returns immediately when evidence/storage_path is absent.
```

## 5. Acceptance Boundary for Implementation

Implementation is correct only if:

```text
PENDING applicant with historical REJECTED evidence
        ↓
Reviewer Verify
        ↓
current PENDING evidence is selected
        ↓
correct document type is shown
        ↓
Load Evidence produces a signed URL
        ↓
evidence document is visible to Reviewer
        ↓
no historical evidence/decision is deleted or mutated
```

The tested recovery applicant is the primary manual regression case.

## 6. Explicit Non-Decisions

This decision does not authorize changes for these separate observations:

```text
- Applicant post-submit confirmation UX
- Any Queue document-label mismatch
- Reviewer History presentation changes
- Other `.single()` or `.find()` usages outside this exact current-evidence selection path
```

Those require separate evidence and/or governance decisions if pursued.

## 7. Next Step

A narrowly scoped implementation prompt may now be created for the Reviewer Verify current-evidence selection fix. Implementation must reference this decision and remain within the exact scope above.
