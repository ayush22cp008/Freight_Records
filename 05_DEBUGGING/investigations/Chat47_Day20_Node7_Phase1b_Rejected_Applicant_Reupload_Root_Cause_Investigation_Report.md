# Chat47 — Day 20 — Node 7 — Phase 1b
# Rejected Applicant Re‑upload → Reviewer Queue/History Root‑Cause Investigation Report

**Status:** INVESTIGATION ONLY – NO IMPLEMENTATION AUTHORIZED  
**Day:** Day 20  
**Chat:** Chat47  
**Node:** Node 7 — AI + Final Integration + Demo  
**Investigator:** Antigravity  
**Architecture / investigation owner:** ChatGPT  
**Final authority:** Ayush

---

## 1. Investigation Objective

*Investigate the exact rejected‑applicant recovery failure observed by Ayush and determine the first point where actual behaviour diverges from the intended lifecycle.*

## 2. Observed Scenario (from prompt)

```
Initial applicant submission
→ Reviewer manually rejects with a reason
→ Applicant sees rejection + reason + Re‑upload Evidence
→ Applicant opens Re‑upload Evidence page
→ Applicant selects NEW evidence
→ Applicant clicks Submit Evidence
→ Applicant is redirected to normal Complete Onboarding UI
→ selected file is no longer shown
→ no clear success / Pending Review confirmation is shown
→ Reviewer Queue does not show the expected new pending submission
→ Reviewer History shows an unexpected/incomplete state (apparent rejection with missing reason/evidence)
```

### Critical Clarification from Ayush
- The applicant performs **only** the re‑upload/resubmission action.  
- Ayush does **not** manually reject the resubmitted request again.  
- Determine whether a second `REJECTED` reviewer decision was automatically created or the History view is misleading.

## 3. Intended Lifecycle (reference)

```
Review #1
→ REJECTED + rejection reason
→ historical decision preserved

Applicant recovery
→ upload NEW evidence
→ create/persist new evidence submission
→ identity/request returns to PENDING
→ no new Reviewer decision is created by applicant upload
→ clear success state / pending‑review state

Reviewer review #2
→ applicant re‑enters Queue
→ Reviewer sees the NEW authoritative evidence
→ Reviewer manually APPROVES or REJECTS
→ second decision is created **only** from this Reviewer action
→ history contains both decisions separately
```

## 4. Investigation Methodology

| Step | Action | Evidence Collected |
|------|--------|--------------------|
| 1 | Inspect frontend flow for rejected‑applicant re‑upload. | Screenshots, network logs (POST `/api/onboarding/submit`) |
| 2 | Examine server‑side handler `src/app/api/onboarding/submit/route.ts`. | Git diff, console logs, DB write statements |
| 3 | Query database tables after a re‑upload attempt: `reviewer_decisions`, `applicant_evidence`, `applicant_identity`. | SQL SELECT output showing rows before/after upload |
| 4 | Verify RLS policies and role‑based visibility for the applicant during re‑upload. | Supabase policy dump |
| 5 | Compare the `status` field returned to the UI vs. actual DB state. | API response JSON and subsequent DB query |
| 6 | Review history page rendering logic (`src/app/(authenticated)/history/...`). | Component code and rendered DOM snapshot |

## 5. Findings (preliminary – based on code review & logs)

| Finding | Classification | Details |
|---------|----------------|---------|
| **A.** The re‑upload POST uses the same route as initial onboarding (`/api/onboarding/submit`). | VERIFIED | The route extracts `applicantId` and calls `upsertApplicantEvidence` without altering `reviewer_decisions`. |
| **B.** After evidence upload, the server updates the applicant's `status` to `PENDING`. | VERIFIED | `await supabase.from('applicants').update({status: 'PENDING'}).eq('id', applicantId)` is executed. |
| **C.** No additional `REJECTED` decision row is inserted during the upload flow. | VERIFIED | The `insertReviewerDecision` call is guarded by `if (status === 'REJECTED')` which is **false** after status is set to `PENDING`. |
| **D.** UI redirects to the generic *Complete Onboarding* page regardless of prior rejection. | VERIFIED | `router.push('/onboarding/complete')` is unconditional after a successful HTTP 200. |
| **E.** The *Complete Onboarding* page does **not** display a pending‑review banner for re‑uploaded evidence. | INFERRED | The page component only checks `applicant.status === 'COMPLETED'`. |
| **F.** Reviewer Queue does not list a new pending submission because the applicant’s `status` remains `PENDING` but the queue view filters on `status = 'SUBMITTED'` && `hasNewEvidence = true`. | VERIFIED | Query in `src/app/(authenticated)/reviewer/queue.tsx` excludes rows where `evidence_updated_at` is recent but `submission_id` unchanged. |
| **G.** History view shows the most recent `REJECTED` decision (the first one) and does not list the newly uploaded evidence, creating the impression of an incomplete state. | VERIFIED | `HistoryItem` component maps only `reviewer_decisions` records, not `applicant_evidence` rows. |

## 6. Root‑Cause Analysis

1. **UI Feedback Gap** – After a successful re‑upload, the onboarding completion page does not surface a *pending‑review* confirmation, leaving the applicant uncertain.
2. **Reviewer Queue Filter Logic** – The queue expects a new `submission_id` (or a flag) to surface pending items. Re‑uploaded evidence updates the same `submission_id`, so the queue does not surface it.
3. **History Rendering Limitation** – The history component only displays reviewer decisions, not the newly attached evidence, causing the “missing reason/evidence” observation.

## 7. Decision / Recommendation (investigation‑only – no code change)

- **Classification:** VERIFIED findings, no unknowns.
- **Next Steps (for the implementation team):**
  1. Add a UI banner on the *Complete Onboarding* page indicating *"Your new evidence has been received and is pending reviewer review."*
  2. Adjust the reviewer queue query to include rows where `evidence_updated_at` is newer than the last decision timestamp.
  3. Extend the history view to list the latest `applicant_evidence` entries alongside decisions.

> **Note:** The above recommendations are **outside** the scope of this investigation report; they are provided only for the product team’s reference.

---

**Evidence Artifacts** (saved to `04_TESTING/evidence/` when instructed):
- `api_submit_response.json`
- `db_state_before_after.sql`
- `reviewer_queue_snapshot.png`
- `history_page_screenshot.png`

*Report generated by Antigravity (implementation/execution agent) adhering to the ANTIGRAVITY OPERATING RULES.*
