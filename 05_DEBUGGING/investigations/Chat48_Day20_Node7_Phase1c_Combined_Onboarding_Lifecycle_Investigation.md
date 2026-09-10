# Chat48 / Day20 / Node7 / Phase1c
# Combined Onboarding Lifecycle Investigation

**Status:** INVESTIGATION ONLY — OPEN
**Project:** Freight — AI Builders Hackathon
**Node:** Node 7 — AI + Final Integration + Demo
**Phase:** Phase 1c
**Chat:** Chat48
**Day:** Day20
**Investigation scope:** Fresh account signup → onboarding evidence submission → Pending Verification → Reviewer Queue → Reviewer Verify → rejection → rejected-applicant re-upload/recovery → Pending Verification
**Implementation authorization:** NONE

## 1. Observation

During whole-account creation testing after the Phase1c rejected-applicant onboarding fix, a newly created Driver account and a newly created Company account both redirected to `/onboarding` and immediately displayed `Pending Verification` even though evidence had not yet been uploaded. The UI showed `Evidence Type: Unknown`.

The same fresh accounts were not visible in the Reviewer Verification Queue, which displayed `All clear / No pending applications at this time`.

This behavior is being investigated together with the previously fixed rejected-applicant recovery onboarding defect because both behaviors involve the same `freight_identities` and `onboarding_evidence` state flow.

## 2. Existing Phase1c Context

The preceding Phase1c fix addressed a rejected applicant whose historical `REJECTED` evidence row remained in `onboarding_evidence` after a new `PENDING` evidence row was created. The previous onboarding page used an applicant-wide `.single()` evidence lookup, which failed when multiple rows existed. The approved implementation changed the lookup to select the newest `PENDING` evidence deterministically and changed the Pending Verification UI gate to use `identity.verification_status === 'PENDING'` as the primary condition.

That rejected-applicant behavior has already been live-UI verified and must be preserved unless this investigation proves it is incorrect.

## 3. Current Evidence

### 3.1 Source-level evidence

Current `/onboarding` behavior:
- authenticated identity is required;
- `PENDING` and `REJECTED` identities are allowed onto `/onboarding`;
- the page queries `onboarding_evidence` for the authenticated `auth_id`, filters to `status = PENDING`, orders by `created_at DESC`, and takes one row;
- the page renders `Pending Verification` whenever `identity.verification_status === 'PENDING'`;
- `Evidence Type` falls back to `Unknown` when no matching evidence row is available.

Current onboarding submission behavior:
- an authenticated applicant in `PENDING` or `REJECTED` state may submit evidence;
- submission inserts a new `onboarding_evidence` row with `status = PENDING`;
- historical evidence is not deleted;
- for a rejected applicant, the identity is transitioned back to `PENDING` after the new evidence row is inserted.

Current Reviewer Queue behavior:
- it loads identities with `verification_status = PENDING`;
- it loads `onboarding_evidence` rows with `status = PENDING`;
- it pairs each pending identity with a matching pending evidence row;
- applicants without matching pending evidence are filtered out of the queue.

### 3.2 Existing architecture evidence

The signup trigger creates a `freight_identities` row with `requested_role` from auth metadata and a default `verification_status` of `PENDING`.

Therefore, the investigation must explicitly verify whether a fresh signup is expected to create:

1. a `PENDING` identity with no evidence yet, or
2. a state/data combination that must already contain onboarding evidence.

The current source and the observed screenshots support the first condition occurring in practice, but the intended state contract must be confirmed from authoritative project records before a fix is chosen.

### 3.3 Live UI evidence from Ayush

Fresh Driver account:
- redirected to `/onboarding`;
- displayed `Pending Verification`;
- displayed `Role Requested: DRIVER`;
- displayed `Evidence Type: Unknown`;
- displayed `Status: PENDING`;
- no evidence upload form was shown.

Fresh Company account:
- redirected to `/onboarding`;
- displayed `Pending Verification`;
- displayed `Role Requested: COMPANY`;
- displayed `Evidence Type: Unknown`;
- displayed `Status: PENDING`;
- no evidence upload form was shown.

Reviewer Queue during the same testing:
- displayed `All clear`;
- displayed `No pending applications at this time`.

These screenshots establish the live behavior but do not by themselves prove the exact database rows for those accounts.

## 4. Relationship to the Previous Phase1c Fix

### Verified relationship

Both behaviors use the same onboarding lifecycle data:

`freight_identities.verification_status` + `onboarding_evidence.status/document_type/created_at`.

The preceding Phase1c change also altered the `/onboarding` decision boundary from an evidence-dependent gate to an identity-status-primary gate. That change is directly relevant to the fresh-signup behavior now observed.

### Not yet verified

It is not yet established whether:

- the previous Phase1c code change itself introduced the fresh-signup behavior;
- the fresh-signup behavior existed previously but was masked by the old `.single()`/evidence-dependent gate;
- the signup state contract was already inconsistent before Phase1c; or
- another recent backend/runtime change is contributing.

This distinction must be established by evidence rather than assumption.

## 5. Investigation Questions

1. What is the authoritative intended state immediately after successful fresh signup for both DRIVER and COMPANY?
2. Is `freight_identities.verification_status = PENDING` intentionally used before evidence submission?
3. If yes, what exact condition distinguishes `PENDING but not yet submitted` from `PENDING and awaiting review` without introducing a new lifecycle state?
4. What exact redirect/gate behavior is intended for the fresh-signup state?
5. What exact state/data combination is required for Reviewer Queue visibility?
6. Does the current signup flow create zero `onboarding_evidence` rows as expected, or should it create one?
7. Does `OnboardingForm` remain reachable for a fresh PENDING applicant under the intended architecture?
8. Does the Phase1c fix correctly preserve the rejected-applicant recovery path while making fresh signup wrong?
9. Can the same correction cover both fresh signup and rejected recovery without changing the protected lifecycle model?
10. Are Driver and Company symmetric for this lifecycle, including role-specific evidence mapping?
11. Does any proposed correction risk affecting existing verified users, Reviewer Queue, Reviewer Verify, rejected recovery, or locked Driver/Company portals?

## 6. Required Evidence Collection

The next investigation pass must collect and document:

- live database state for a freshly created DRIVER account immediately after signup and before evidence submission;
- live database state for a freshly created COMPANY account at the same point;
- whether a corresponding `onboarding_evidence` row exists for each fresh account;
- current `freight_identities` state and any relevant timestamps for each test account;
- exact runtime behavior of `/onboarding` for a fresh applicant;
- exact runtime behavior of `/reviewer/queue` for the same accounts;
- behavior after submitting role-specific evidence;
- behavior after Reviewer processing;
- behavior after rejection and re-upload;
- confirmation that historical REJECTED evidence remains preserved;
- confirmation that the original reviewer decision remains preserved;
- confirmation that Reviewer Queue and Reviewer Verify select the correct current evidence;
- confirmation that no duplicate reviewer rejection is created by re-upload;
- build/test evidence for any eventual implementation.

## 7. Hypothesis to Test

**Primary hypothesis:** the onboarding state model currently conflates two distinct situations under the same `PENDING` identity status:

- fresh applicant with no evidence yet;
- applicant whose evidence has already been submitted and is awaiting review.

The observed `Evidence Type: Unknown` and empty Reviewer Queue are consistent with this hypothesis. It is currently an **INFERRED** diagnosis, not yet a final root cause.

## 8. Constraints / Protected Boundaries

Do not:

- introduce a new lifecycle state unless separately investigated and explicitly approved;
- revert the already verified Phase1c rejected-applicant recovery fix without evidence;
- change Reviewer authority;
- change RLS/security architecture;
- redesign the evidence model;
- change authenticated role rules;
- modify the locked Driver or Company portals;
- implement a fix before the investigation and evidence stage is complete.

## 9. Required Investigation Pipeline

```text
OBSERVATION
    ↓
INVESTIGATION
    ↓
EVIDENCE
    ↓
ROOT CAUSE
    ↓
DECISION
    ↓
FIX (separate instruction)
    ↓
BUILD / TEST
    ↓
AYUSH MANUAL VERIFICATION
```

## 10. Current Classification

- Fresh-signup Pending Verification with `Evidence Type: Unknown` → **VERIFIED as live UI observation**
- Reviewer Queue empty for the fresh test accounts → **VERIFIED as live UI observation**
- Shared onboarding lifecycle between the fresh-signup and rejected-recovery cases → **VERIFIED at source/data-flow level**
- Exact root cause of the fresh-signup behavior → **INFERRED / OPEN**
- Whether the previous Phase1c fix directly caused it → **UNKNOWN**
- Final fix decision → **NOT YET AUTHORIZED**

## 11. Investigation Outcome Target

The investigation is complete only when the project can state, with evidence, the correct behavior for all four states:

```text
Fresh signup
→ evidence-upload / Complete Onboarding

Evidence submitted
→ Pending Verification

Rejected applicant
→ Re-upload Evidence

Rejected applicant re-uploaded
→ Pending Verification
```

The chosen solution must preserve existing verified behavior and remain within the locked Phase1c scope unless new evidence requires explicit governance escalation.
