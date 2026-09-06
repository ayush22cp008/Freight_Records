# Chat40 — Day16 — Node 7 — Phase 1b
# Reviewer Interaction Mapping Decisions

**Status:** LOCKED
**Authority:** Human-approved architecture decisions recorded during Chat40 / Day16.
**Purpose:** Standalone authoritative record of the locked Reviewer Interaction Mapping decisions used by the Reviewer Final Blueprint.

---

## 1. Interaction Mapping Scope

The Reviewer Interaction Mapping defines the Reviewer’s end-to-end verification workflow from entry into the Verification Queue through Applicant Verification, Evidence Examination, Identity / Role Verification, final Approve / Reject decision, Decision Result, and Verification History.

This record is the standalone source for the seven locked Reviewer interactions referenced by the Reviewer Final Blueprint.

The mapping does not introduce Reviewer trip/delivery/claim authority, AI verification, scoring, new evidence types, new delivery stages, marketplace behavior, or other operational functionality outside the locked Reviewer responsibility boundary.

---

## 2. Interaction 1 — Reviewer Entry → Verification Queue

**Status: LOCKED**

1. An authorized Reviewer enters directly into the **Verification Queue**.
2. The queue provides **direct preview/access to submitted evidence**. It does not introduce an artificial or generated “Evidence Summary,” assumed document type, AI judgment, or scoring.
3. Each applicant has a dedicated **Review** action that opens Applicant Verification.
4. Review opens an **Applicant Verification View** containing:
   - Applicant
   - Claimed Role
   - Submitted Evidence
   - Dedicated evidence viewing area
   - Responsive behavior across desktop, tablet, and mobile
   - Exact responsive layout and evidence-viewer capabilities deferred to the Final Blueprint.
5. Opening an applicant does not change persistent state. The applicant remains **Pending Verification** until a final decision is committed.

---

## 3. Interaction 2 — Queue → Applicant Verification

**Status: LOCKED**

1. Applicant Verification immediately presents **applicant identity + claimed role + submitted evidence**.
2. A clear **Back to Verification Queue** control is provided.
3. Applicant identity context is limited to **email + claimed role**; unnecessary profile information is not introduced.
4. Evidence is presented within the same verification view, directly alongside or below the applicant information.
5. If evidence cannot be viewed because of a technical or access problem, the UI shows a clear error and provides retry/open-access handling. It does not auto-reject or hide the problem.
6. Leaving an unfinished review does not create persistent state or review progress. The applicant remains **Pending Verification**; there is no `Under Review` state.

---

## 4. Interaction 3 — Applicant Verification → Evidence Examination

**Status: LOCKED**

1. Applicant Verification contains a dedicated large evidence viewer while keeping applicant email + claimed role visible.
2. The viewer supports viewing, scrolling, and zoom/resize where applicable. Download is not a required Reviewer workflow capability.
3. Exactly one onboarding evidence document is used, according to the already-approved onboarding document rule:
   - Driver → Driving Licence
   - Company → GST document
4. The Reviewer evaluates whether the evidence supports the applicant’s **claimed identity and claimed role**.
5. The Reviewer explicitly confirms **Identity / Role Verified** after evaluation. This is a human verification action and is not a new persistent lifecycle state.
6. If evidence does not adequately support the claimed identity/role, the Reviewer **Rejects** with a required rejection reason. Technical inability to view evidence is a separate failure condition and does not auto-reject.
7. After Identity / Role Verified, the Reviewer remains on the same verification view and **Approve** becomes enabled. Verification and final approval remain separate actions.

---

## 5. Interaction 4 — Evidence Examination → Identity / Role Verification

**Status: LOCKED**

1. The Reviewer has an explicit **Identity / Role Verified** action after evidence examination. Viewing evidence alone does not constitute verification.
2. After selecting it, the UI shows a clear confirmation/state and enables **Approve** in the same view; no separate page is required.
3. The Reviewer can undo or reconsider the verification before final Approve/Reject. This does not create a new persistent state.
4. If the Reviewer cannot confidently verify the claimed identity/role because the evaluation is unfinished or inconclusive, the applicant remains **Pending Verification** and the Reviewer may exit/return later. This does not override the separate rule that substantively insufficient evidence is rejected with a required reason.
5. If Identity/Role Verified was selected but the Reviewer leaves before Approve, that verification is not committed as persistent state. The applicant remains **Pending Verification**.

---

## 6. Interaction 5 — Verification → Approve / Reject

**Status: LOCKED**

1. After Identity/Role Verified, the same view clearly presents both **Approve** and **Reject** actions.
2. Approve requires final confirmation before commit.
3. Reject requires a rejection reason and final confirmation before commit.
4. After final confirmation, decision controls are disabled and a clear processing state is shown until the result is confirmed.
5. A successful decision shows a clear success/result state and then returns the Reviewer to the **Verification Queue**.
6. A completed applicant is removed from the Pending Verification queue. The queue contains only unfinished verification work.
7. If the final decision API/server/network request fails, the applicant remains **Pending Verification**. The UI shows a clear error and allows retry; it never assumes success.

---

## 7. Interaction 6 — Decision → Result / Verification History

**Status: LOCKED**

1. After a successful final decision, the result state clearly shows:
   - Applicant email
   - Claimed role
   - Final decision: Verified or Rejected
   - Rejection reason when applicable
   - Clear action to return to the Verification Queue
2. A dedicated **Verification History** records completed Verified/Rejected decisions.
3. Each History entry shows:
   - Applicant email
   - Claimed role
   - Final decision
   - Rejection reason when applicable
4. History is a **decision-record view**, not a duplicate evidence-examination workspace.
5. Selecting a completed History record opens a **read-only verification record** containing:
   - Applicant
   - Claimed role
   - Final decision
   - Rejection reason if applicable
   - Access to submitted evidence
   - No ability to change the completed decision from History
6. The completed record provides a clear **Back to Verification History** control.

---

## 8. Interaction 7 — Verification History → Completed Record / Navigation

**Status: LOCKED**

1. Verification History is a simple chronological list of completed verification records.
2. Records are ordered **newest decision first**.
3. Each entry shows:
   - Applicant email
   - Claimed role
   - Final decision
   - Decision date/time
4. Initial History has **no filtering**.
5. Empty History displays exactly:
   **“No completed verification records yet.”**
   and provides a **Back to Verification Queue** action.
6. Larger History sets use **pagination**.
7. Selecting a History entry opens a separate dedicated **read-only verification record**.
8. Submitted evidence remains viewable through an evidence viewer within that read-only record.
9. Evidence is **view-only** in completed History. The completed record has no Identity/Role Verified, Approve, Reject, or decision-change action.
10. Returning to History preserves the previous History page/position context.

---

## 9. Locked Cross-Interaction Rules

1. Persistent lifecycle states remain **Pending Verification → Verified / Rejected**. There is no persistent `Under Review` state.
2. Identity / Role Verified is a UI-level human confirmation step, not a new persistent lifecycle state.
3. Technical evidence-viewing failure is not equivalent to substantive evidence rejection.
4. Final decision failure preserves Pending Verification and permits retry.
5. Completed decisions are immutable from Verification History.
6. Reviewer scope remains onboarding identity/evidence verification only.
7. No AI judgment, scoring, auto-verification, or generated evidence summary is introduced.
8. No new trip, delivery, claim, marketplace, or operational Reviewer authority is introduced.

---

## 10. Relationship to Reviewer Final Blueprint

This file is the standalone locked Interaction Mapping source referenced by:

`02_ARCHITECTURE/modules/Chat40_Node7_Phase1b_Reviewer_Final_Blueprint.md`

The Final Blueprint consolidates the locked decisions in this record into its complete 13-section architecture contract.

Any future change to these locked interactions requires a new explicit architecture decision and must not silently alter this record.

---

## 11. Final Lock

**Reviewer Interaction Mapping: COMPLETE / LOCKED**

The seven interactions above constitute the approved Reviewer Interaction Mapping for Chat40 / Day16 / Node 7 / Phase 1b.

**Implementation status:** NOT STARTED.

**Next architectural gate:** Re-check Claude alignment against this standalone locked Interaction Mapping record, then lock the Reviewer Final Blueprint if no substantive contradiction remains.
