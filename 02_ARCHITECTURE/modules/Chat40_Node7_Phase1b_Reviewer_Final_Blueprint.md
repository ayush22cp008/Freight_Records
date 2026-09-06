# Chat40 — Node 7 — Phase 1b
# Reviewer Final Blueprint

**Status:** DRAFT FOR CLAUDE ALIGNMENT REVIEW
**Day:** Day 16
**Current Node:** Node 7 → Phase 1b
**Scope:** Reviewer Portal

## 1. Blueprint Purpose & Authority

This document consolidates the completed Reviewer Existing-System Investigation, the locked Reviewer Mental Model, and the seven locked Reviewer Interaction Mapping interactions into one implementation-facing blueprint for independent review before implementation preparation.

This blueprint does not replace the underlying evidence records. Existing-system facts remain authoritative to the investigation records; mental-model decisions remain authoritative to the locked mental-model record; interaction behavior remains authoritative to the locked interaction decisions. Any requirement not established by those sources must not be silently invented.

Primary source records:
- `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Report.md`
- `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Completion_Report.md`
- `00_PROJECT_CONTROL/Chat40_Day15_Node7_Phase1b_Reviewer_Mental_Model_Decisions.md`

The seven Interaction Mapping decisions were locked during Chat40 / Day16 and are consolidated in Section 6.

## 2. Reviewer Responsibility Boundary

### Reviewer responsibility

The Reviewer is an **Identity & Evidence Verifier**.

The Reviewer:
- examines submitted onboarding evidence;
- considers the applicant and claimed/requested role context;
- evaluates whether the evidence supports the claimed identity and role;
- explicitly verifies identity/role before final approval;
- approves or rejects the applicant.

### Outside Reviewer responsibility

The Reviewer is not responsible for:
- trip operations;
- delivery operations;
- claims;
- Driver operations;
- Company operations;
- general platform administration;
- operational trip/delivery evidence review.

No new Reviewer responsibilities are introduced by this blueprint.

## 3. Reviewer Mental Model

The locked Reviewer mental model is:

```text
Applicant
    +
Claimed Role
    +
EVIDENCE
    ↓
Evaluation
    ↓
Identity / Role Verification
    ↓
Approve / Reject
```

### Core model

- **Primary Job:** Identity & Evidence Verifier
- **Primary Object:** Evidence
- **Information Model:** Evidence + Applicant + Requested Role
- **Verification / Decision Model:** Applicant + Role + Evidence → Evaluate → Verify → Approve/Reject
- **State Model:** Pending Verification → Verified / Rejected
- **Mental Journey:** Verification-first
- **Trust Model:** Evidence must support the claimed identity/role
- **Principles:** Evidence-centered, identity-aware, decision-driven

The mental model does not introduce scoring, AI decision-making, automated verification, multiple-evidence requirements, a persistent `under_review` state, or new Reviewer business responsibilities.

## 4. Information Architecture

The Reviewer experience is organized around the complete verification workflow rather than the existing flat queue-only surface.

```text
Reviewer
├── Verification Queue
├── Applicant Verification
│   └── Evidence Examination
├── Decision Result
└── Verification History
    └── Read-only Verification Record
        └── Submitted Evidence Viewer
```

### Verification Queue

Purpose: unfinished Reviewer work.

- Contains applicants in **Pending Verification**.
- Provides applicant identification and claimed/requested role context.
- Provides direct access to submitted evidence.
- Each applicant has a dedicated **Review** action.
- Opening an applicant does not change persistent state.

### Applicant Verification

Purpose: dedicated verification workspace for one pending applicant.

Required context:
- Applicant email
- Claimed/requested role
- Submitted evidence

Provides:
- dedicated evidence viewing area;
- explicit Back to Verification Queue navigation;
- verification and final-decision actions according to the locked interaction flow.

### Decision Result

Purpose: clearly communicate the result after a successful final decision.

Shows:
- Applicant email
- Claimed role
- Final decision
- Rejection reason when applicable
- Clear return action to Verification Queue

### Verification History

Purpose: completed decision records.

- Simple chronological list.
- Newest decision first.
- No filtering in the initial workflow.
- Pagination for larger result sets.
- Clear empty state when no completed records exist.

### Read-only Verification Record

Purpose: reference a completed verification decision without reopening the decision workflow.

Shows:
- Applicant
- Claimed role
- Final decision
- Rejection reason when applicable
- Decision date/time as represented by the History entry
- Access to submitted evidence

Provides:
- evidence viewer in the same read-only record view;
- **Back to Verification History** control.

Completed decisions cannot be changed from History.

## 5. State Model

The persistent lifecycle remains:

```text
Pending Verification
        ↓
     Evaluate
        ↓
 ┌──────┴──────┐
 ↓             ↓
Verified     Rejected
```

### Persistent state rules

- Opening a queue item does not change state.
- Leaving unfinished review does not create persistent review progress.
- There is no persistent `Under Review` state.
- Selecting **Identity / Role Verified** is a human verification action, not a new persistent state.
- Final **Approve** commits the Verified outcome.
- Final **Reject** commits the Rejected outcome.
- If a final decision request fails, the applicant remains Pending Verification and the Reviewer can retry.

## 6. Complete Interaction Contract

### Interaction 1 — Reviewer Entry → Verification Queue

1. An authorized Reviewer enters directly into the **Verification Queue**.
2. The Queue provides direct preview/access to submitted evidence; no artificial/generated Evidence Summary is assumed.
3. Each applicant has a dedicated **Review** action that opens Applicant Verification.
4. Applicant Verification contains Applicant, Claimed Role, Submitted Evidence, and a dedicated evidence viewing area; exact responsive layout/viewer capabilities are finalized within this blueprint without inventing unsupported evidence behavior.
5. Opening an applicant does not change persistent state; it remains Pending Verification.

### Interaction 2 — Queue → Applicant Verification

1. Applicant Verification immediately presents applicant identity, claimed role, and submitted evidence.
2. A clear **Back to Verification Queue** control is provided.
3. Applicant identity context is limited to email + claimed role unless separately justified by verified requirements.
4. Evidence is presented within the same verification view.
5. If evidence cannot be viewed because of a technical/access issue, show a clear error with retry/open-access behavior; do not auto-reject or hide the problem.
6. Leaving unfinished review does not create persistent state/progress; applicant remains Pending Verification.

### Interaction 3 — Applicant Verification → Evidence Examination

1. A dedicated large evidence viewer exists within Applicant Verification while applicant email and claimed role remain visible.
2. The viewer supports viewing, scrolling, and zoom/resize where applicable; download is not a required Reviewer behavior.
3. Exactly one onboarding evidence document is handled under the already-approved onboarding rule:
   - Driver → Driving Licence
   - Company → GST document
4. Reviewer evaluates whether evidence supports the applicant's claimed identity and claimed role.
5. Reviewer explicitly confirms **Identity / Role Verified** after evaluation; this is a human action, not a new persistent state.
6. If evidence does not adequately support claimed identity/role, Reviewer rejects with a required rejection reason. Technical inability to view evidence remains a separate failure case.
7. After Identity / Role Verified, Reviewer stays in the same verification view and Approve becomes enabled; verification and approval remain separate.

### Interaction 4 — Evidence Examination → Identity / Role Verification

1. **Identity / Role Verified** is an explicit action after evidence examination; viewing evidence alone does not constitute verification.
2. After selection, show a clear confirmation/state and enable Approve in the same view.
3. Reviewer can undo/reconsider verification before final Approve/Reject; no new persistent state is created.
4. If Reviewer has not reached a confident verification conclusion, applicant remains Pending Verification and Reviewer can exit/return later. This does not override the separate rule that evidence substantively found insufficient should be rejected with a reason.
5. If Reviewer selects Identity / Role Verified but leaves before Approve, verification is not committed and applicant remains Pending Verification.

### Interaction 5 — Verification → Approve / Reject

1. After Identity / Role Verified, the same view clearly shows Approve and Reject.
2. Approve requires final confirmation before commit.
3. Reject requires a required rejection reason and final confirmation before commit.
4. After final confirmation, decision controls are disabled and a clear processing state is shown until the result is confirmed.
5. Successful decision shows a clear result state and then returns to Verification Queue.
6. Completed applicants are removed from the Pending Verification queue; the queue represents unfinished work only.
7. If the final decision request fails, the applicant remains Pending Verification, a clear error is shown, and retry remains possible; success is never assumed without confirmation.

### Interaction 6 — Decision → Result / Verification History

1. After successful final decision, result state shows Applicant email, Claimed role, Final decision, rejection reason when applicable, and a clear action to return to Verification Queue.
2. A dedicated **Verification History** contains completed Verified/Rejected decisions.
3. History entries show Applicant email, Claimed role, Final decision, and rejection reason when applicable; History is a decision-record view, not a duplicate evidence-examination workspace.
4. Selecting a completed History record opens a **read-only verification record** with Applicant, Claimed role, Final decision, rejection reason if applicable, and access to submitted evidence. Completed decisions cannot be changed from History.
5. The read-only record provides a clear **Back to Verification History** control.

### Interaction 7 — Verification History → Completed Record / Navigation

1. Verification History is a simple chronological list of completed verification records.
2. Records are ordered newest decision first.
3. Each entry shows Applicant email, Claimed role, Final decision, and decision date/time.
4. Initial History has no filtering.
5. Empty History shows a clear message such as **“No completed verification records yet.”** with a **Back to Verification Queue** action.
6. Larger History sets use pagination.
7. Selecting a History entry opens a separate dedicated read-only verification record.
8. Submitted evidence is viewable in an evidence viewer within that same read-only record.
9. Evidence is view-only in completed History; no Identity/Role Verified, Approve, Reject, or decision-change action is available.
10. Returning to History preserves the Reviewer’s previous History page/position context.

## 7. Evidence Rules

The evidence model remains evidence-centered.

- Evidence directly supports the identity/role evaluation.
- The Reviewer sees submitted evidence within the verification workflow.
- The already-approved onboarding rule determines the single expected document by claimed role:
  - Driver → Driving Licence
  - Company → GST document
- No additional evidence types are introduced.
- No mandatory multiple-evidence requirement is introduced.
- No scoring or confidence automation is introduced.
- Technical evidence viewing failure is not treated as substantive rejection.
- Completed History provides evidence access only for reference; evidence is view-only there.

## 8. Decision Rules

### Verification

- Evidence examination precedes Identity / Role Verified.
- Identity / Role Verified is explicit and human-controlled.
- It is not a persistent workflow state.
- Reviewer may reconsider/undo the verification action before final decision.

### Approve

```text
Identity / Role Verified
        ↓
Final confirmation
        ↓
Processing
        ↓
Server-confirmed success
        ↓
Verified result
        ↓
Return to Verification Queue
```

### Reject

```text
Evidence insufficient for claimed identity/role
        ↓
Required rejection reason
        ↓
Final confirmation
        ↓
Processing
        ↓
Server-confirmed success
        ↓
Rejected result
        ↓
Return to Verification Queue
```

### Decision failure

If the final request fails:
- no successful decision is assumed;
- applicant remains Pending Verification;
- clear error is shown;
- Reviewer can retry.

## 9. History Rules

Verification History represents completed decisions only.

- Chronological list.
- Newest decision first.
- Entry summary: Applicant email + Claimed role + Final decision + Decision date/time.
- No filtering in the initial workflow.
- Pagination for larger result sets.
- Empty state: **“No completed verification records yet.”** plus Queue navigation.
- Selecting an entry opens a dedicated read-only verification record.
- The completed record includes submitted evidence access.
- Evidence is view-only.
- No completed decision can be changed from History.
- Clear **Back to Verification History** navigation is provided.
- Previous History page/position context is preserved when returning.

History is not a duplicate active verification workspace and does not reintroduce final decision controls.

## 10. Navigation & Responsive Requirements

### Navigation

The Reviewer experience must provide role-appropriate navigation rather than trapping the Reviewer in the existing shared navigation behavior.

Required workflow navigation includes:
- Reviewer entry → Verification Queue
- Queue → Applicant Verification
- Applicant Verification → Back to Verification Queue
- Successful decision → Verification Queue
- Verification History → Read-only Verification Record
- Read-only Verification Record → Back to Verification History

The existing investigation identified the shared Navbar Dashboard/Timeline navigation trap. The final implementation must not preserve a navigation pattern that repeatedly redirects a Reviewer back to the Queue.

The exact visual navigation component and layout are implementation details and must remain within the locked behavioral boundary.

### Responsive behavior

The Reviewer workflow must remain usable across:
- desktop;
- tablet;
- mobile.

The exact responsive component arrangement, dimensions, and evidence-viewer presentation are implementation-detail decisions subject to this behavioral contract and must not introduce new functionality.

## 11. Existing-System Defects → Blueprint Response

The verified existing-system defects are carried forward as implementation concerns.

| Defect | Blueprint response boundary |
|---|---|
| **REV-01 Navigation Trap** | Reviewer-specific navigation must provide the complete verification workflow without Dashboard/Timeline redirect loops. |
| **REV-02 Role-Confusion Lockout** | Reviewer routing must not permanently prevent a user with another valid operational identity from reaching their operational experience. Exact multi-role routing behavior must be verified/approved before implementation if not already specified elsewhere. |
| **REV-03 RLS Bypass Architecture** | Reviewer API/data mutation security requires correction away from reliance on service-role god-mode access, using an explicitly approved secure authorization/RLS design. Exact implementation must be separately verified before fix execution. |
| **REV-04 Degraded UX** | Replace native browser rejection prompt behavior with the application's approved UI interaction pattern, including proper rejection input, confirmation, and clear error/success handling. |

### Existing-system items to preserve

- Reviewer storage evidence RLS is correctly protected and should not be weakened.
- Existing onboarding identity/evidence data model should be reused where appropriate.
- Existing verified authentication/authorization boundary remains the basis for Reviewer access, subject to the approved security correction.

## 12. Implementation Boundary

### In scope

- Reviewer frontend redesign around the locked verification workflow.
- Dedicated Verification Queue experience.
- Dedicated Applicant Verification view.
- Dedicated evidence viewing area.
- Explicit Identity / Role Verified interaction.
- Explicit Approve/Reject confirmation flow.
- Clear processing, success, and failure states.
- Verification History.
- Read-only completed verification record.
- View-only evidence access from completed records.
- Reviewer-appropriate navigation and responsive behavior.
- Fixes for the verified Reviewer UX/routing defects.
- Required security correction for the verified RLS/service-role bypass defect, after implementation-level security design is verified and approved.
- Reuse of existing APIs/data where compatible with the locked behavior.

### Out of scope unless separately verified and approved

- New Reviewer business responsibilities.
- Trip/delivery management or review.
- Claims or marketplace functionality.
- Operational evidence review outside onboarding identity verification.
- AI verification or AI decision-making.
- Evidence scoring/confidence scoring.
- New persistent Reviewer workflow states such as `under_review`.
- New evidence types or multiple-evidence requirements.
- New authorization rules not required by a verified defect fix.
- General administration dashboard/metrics as a new Reviewer responsibility.
- Unverified backend business functionality.

## 13. Acceptance / Completion Criteria

This blueprint is ready for implementation preparation only when independent review confirms alignment with the three governing inputs.

### Behavioral acceptance

- All seven locked interactions are represented.
- Reviewer mental journey remains verification-first.
- Evidence, Applicant, and Claimed Role remain the core verification context.
- Persistent lifecycle remains Pending Verification → Verified/Rejected.
- No persistent `under_review` state is introduced.
- Identity/Role Verified remains a human action, not a new persistent state.
- Approve/Reject remain explicit final decisions.
- Final decision failure preserves Pending Verification.
- Completed History is read-only and cannot change decisions.

### Scope acceptance

- Reviewer responsibility remains narrow.
- No trip/delivery/claims/general-admin responsibilities are introduced.
- No AI/scoring/automation is introduced.
- No unsupported evidence requirements are introduced.

### Existing-system acceptance

- The blueprint addresses the verified Reviewer defects REV-01 through REV-04 within the stated boundaries.
- Existing evidence-storage protection is preserved.
- Existing data/domain relationships are reused where compatible.
- Security corrections are not treated as completed until independently verified.

### Review status

**Current status: DRAFT FOR CLAUDE ALIGNMENT REVIEW**

This document is intentionally not marked Final/Locked. Claude review should identify any contradiction, omission, unsupported assumption, scope expansion, or mismatch against:
1. Existing Reviewer System Investigation;
2. Reviewer Mental Model;
3. Seven locked Reviewer Interaction Mapping interactions.

No implementation should begin from this draft until the alignment review is resolved and the blueprint is explicitly approved/locked.
