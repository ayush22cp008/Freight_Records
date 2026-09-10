# Chat47 — Day 20 — Node 7 — Phase 1b
# Rejected Applicant Re-upload → Reviewer Queue/History Root-Cause Investigation

**Status:** INVESTIGATION ONLY — NO IMPLEMENTATION AUTHORIZED  
**Day:** Day 20  
**Chat:** Chat47  
**Node:** Node 7 — AI + Final Integration + Demo  
**Investigator:** Antigravity  
**Architecture / investigation owner:** ChatGPT  
**Final authority:** Ayush  

## 1. Investigation Objective

Investigate the exact rejected-applicant recovery failure observed by Ayush and determine the first point where actual behavior diverges from the intended lifecycle.

Observed scenario:

```text
Initial applicant submission
→ Reviewer manually rejects with a reason
→ Applicant sees rejection + reason + Re-upload Evidence
→ Applicant opens Re-upload Evidence page
→ Applicant selects NEW evidence
→ Applicant clicks Submit Evidence
→ Applicant is redirected to normal Complete Onboarding UI
→ selected file is no longer shown
→ no clear success / Pending Review confirmation is shown
→ Reviewer Queue does not show the expected new pending submission
→ Reviewer History shows an unexpected/incomplete state (apparent rejection with missing reason/evidence)
```

Critical clarification from Ayush:

- The applicant performs only the re-upload/resubmission action.
- Ayush does **not** manually reject the resubmitted request again.
- Therefore, determine whether a second `REJECTED` reviewer decision was actually created automatically, or whether History is incorrectly displaying an existing/older decision.

The investigation must establish the true root cause rather than assuming that the redirect itself caused the Reviewer-side state.

## 2. Required Intended Lifecycle

The expected lifecycle for an already rejected applicant is:

```text
Review #1
→ REJECTED + rejection reason
→ historical decision preserved

Applicant recovery
→ upload NEW evidence
→ create/persist new evidence submission
→ identity/request returns to PENDING
→ no new Reviewer decision is created by applicant upload
→ clear success state / pending-review state

Reviewer review #2
→ applicant re-enters Queue
→ Reviewer sees the NEW authoritative evidence
→ Reviewer manually APPROVES or REJECTS
→ second decision is created only from this Reviewer action
→ history contains both decisions separately
```

Do not redesign this lifecycle. Verify the existing implementation against it.

## 3. Investigation Rules

This is a truth audit only.

Do NOT:

- modify application source code;
- modify migrations;
- modify production schema, RLS, storage policies, or records to make tests pass;
- fix Queue, onboarding, recovery, or History during this investigation;
- create a workaround;
- infer a database write from UI behavior alone;
- call an apparent second rejection a confirmed second rejection without direct evidence.

Follow:

```text
OBSERVATION
→ INVESTIGATION
→ EVIDENCE
→ ROOT CAUSE
→ DECISION
```

Classify every material finding as:

```text
VERIFIED / INFERRED / UNKNOWN / BLOCKED
```

## 4. Exact Source-Code Path to Trace

Inspect the current source repository and trace the exact code executed by the rejected-applicant re-upload flow.

At minimum inspect:

```text
src/app/(authenticated)/onboarding/page.tsx
src/app/(authenticated)/onboarding/OnboardingForm.tsx
src/app/api/onboarding/submit/route.ts
```

Also inspect all current components/routes used by the rejected-applicant recovery link/button and any shared onboarding-status logic.

Determine:

```text
- how the page decides between Complete Onboarding and Re-upload Evidence;
- what state/role/status is read to make that decision;
- whether the same submit handler is used for fresh onboarding and recovery;
- what request payload the re-upload submits;
- whether the payload differs from fresh onboarding;
- what server-side branch handles a REJECTED applicant;
- all database writes performed during re-upload;
- all storage operations performed during re-upload;
- API success/error responses;
- frontend success/error handling;
- redirect/router refresh/navigation behavior.
```

Search for all references to:

```text
REJECTED
PENDING
Re-upload Evidence
Complete Onboarding
onboarding_evidence
freight_identities
reviewer_decisions
reviewed_at
rejection_reason
```

## 5. Required Before/After Database Truth

For the exact test applicant used by Ayush, establish state before and after the re-upload attempt.

Capture:

### Identity

```text
auth.users.id
freight_identities.id
auth_id
requested_role
trusted_role
verification_status
reviewed_at
created_at
```

### ALL evidence rows for this applicant

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

### ALL reviewer decisions for this applicant

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

Compare:

```text
BEFORE re-upload
vs
AFTER re-upload
```

The report MUST explicitly answer:

```text
Did a NEW onboarding_evidence row get inserted?
Did the identity return to PENDING?
Did the old rejected evidence remain preserved?
Did a second reviewer_decisions row get created?
If a second decision exists, what exact code path created it?
If no second decision exists, why does History appear to show an additional/incomplete rejection?
```

## 6. Storage Truth

Determine whether the newly selected file was actually uploaded.

Verify:

```text
bucket
object path
object existence
object metadata where available
relationship to onboarding_evidence.storage_path
```

Do not treat the browser file input becoming empty as evidence that the storage upload failed.

## 7. API / Frontend Truth

Determine the exact response from `/api/onboarding/submit` for the re-upload.

Capture:

```text
HTTP status
response body
server-side error, if any
success branch, if any
frontend branch taken
redirect/navigation target
```

Determine why the UI lands on `Complete Onboarding`.

Distinguish explicitly between:

```text
A. successful recovery → expected page/state rendering
B. partial backend success + incorrect frontend redirect
C. backend failure + incorrect frontend fallback
D. a different unexpected code path
E. other / unknown
```

## 8. Reviewer Queue Truth

After the same re-upload attempt, determine why the applicant does or does not appear in Queue.

Trace the current Queue query and establish:

```text
eligibility filter
identity status filter
role filter
Join/evidence logic
evidence status filter
multiple-evidence handling
current/authoritative evidence selection
```

Determine whether Queue omission is caused by:

```text
identity not PENDING
no new evidence
wrong evidence status
wrong evidence association
multiple evidence cardinality
selection/order bug
other
unknown
```

Do not alter Queue code during this investigation.

## 9. Reviewer Verify Truth

Inspect:

```text
src/app/api/admin/review/route.ts
src/app/(authenticated)/reviewer/verify/[id]/page.tsx
src/app/(authenticated)/reviewer/verify/[id]/ApplicantVerificationClient.tsx
```

Establish:

```text
identity lookup
PENDING enforcement
current evidence selection
selected evidence_id
mutation target
mutation scope
history insert behavior
error handling
```

Particular attention:

```text
Does the selected evidence ID match the evidence ID used in every mutation/history write?
Does any operation update more than the intended evidence row?
Can this route create a reviewer decision without an actual Reviewer action?
```

## 10. Reviewer History Truth

Inspect:

```text
src/app/api/admin/history/route.ts
src/app/(authenticated)/reviewer/history/page.tsx
src/app/(authenticated)/reviewer/history/[id]/page.tsx
src/app/(authenticated)/reviewer/history/[id]/EvidenceViewerClient.tsx
```

Establish from actual data and code:

```text
What history rows exist?
Which decision is displayed?
How is the rejection reason sourced?
How is evidence_id sourced?
How is evidence resolved?
Why can reason/evidence appear missing?
Is a new rejection actually present?
```

Do not infer a second rejection from a UI label alone.

## 11. Critical Root-Cause Test

The key causal question is:

> Did the applicant's re-upload submission itself cause Reviewer decision state to change, or are the Reviewer symptoms independent/secondary effects?

Produce an explicit causal chain:

```text
Re-upload submit
→ [exact server operation]
→ [exact DB/storage state change]
→ [exact frontend result]
→ [exact Queue result]
→ [exact History result]
```

Identify the FIRST state transition that is inconsistent with the intended lifecycle.

## 12. Special Checks

Explicitly check the known suspicious areas from the prior Day 19 audit:

```text
- multiple onboarding_evidence rows after recovery
- `.single()` against onboarding_evidence
- `.find()` evidence selection in Queue
- currentEvidence vs mutation evidence ID
- reviewer_decisions schema/existence
- history evidence linkage
- non-transactional or partially writable Reviewer decisions
- service-role writes and their target scoping
```

However, do not assume any of these is the root cause merely because it was previously flagged.

## 13. Required Final Report

Save the investigation report under:

```text
05_DEBUGGING/investigations/Chat47_Day20_Node7_Phase1b_Rejected_Applicant_Reupload_Root_Cause_Investigation_Report.md
```

The report must contain:

```text
1. Exact observed scenario
2. Test applicant/environment used
3. Source-code path traced
4. Before/after database evidence
5. Storage evidence
6. API response evidence
7. Frontend redirect/state evidence
8. Queue evidence
9. Reviewer Verify evidence
10. History evidence
11. Hypothesis results
12. First divergence from intended lifecycle
13. Root cause
14. Secondary symptoms explained
15. VERIFIED / INFERRED / UNKNOWN / BLOCKED classification
16. Recommended fix scope (diagnosis only — no implementation)
17. What remains blocked or requires Ayush manual verification
```

Do not create an implementation prompt from this investigation. The next implementation decision will be made only after ChatGPT reviews the investigation report and Ayush authorizes the appropriate decision/fix stage.

## 14. Postflight

Before reporting completion, confirm:

```text
- no source-code changes made
- no schema/RLS/storage changes made
- no production data mutation except explicitly approved safe test setup, if any
- exact records/files produced
- investigation evidence is tied to the current code/runtime state
- unexpected changes, if any, are reported
```
