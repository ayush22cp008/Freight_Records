# Chat40 — Day 16 — Node 7 — Phase 1b
# Reviewer Final Blueprint

**Status:** DRAFT — FOR CLAUDE ALIGNMENT REVIEW
**Purpose:** Full 13-section Reviewer Blueprint for section-by-section alignment review against the verified Existing-System Investigation, locked Reviewer Mental Model, and seven locked Reviewer Interaction Mapping interactions.

---

## 1. Blueprint Purpose & Authority

### 1.1 Purpose

This blueprint consolidates the completed Reviewer reasoning work into one detailed implementation-facing contract before implementation preparation.

The blueprint exists to answer, in one place:
- what the Reviewer is responsible for;
- what information the Reviewer needs;
- what Reviewer surfaces and navigation paths are required;
- how the seven locked interactions behave;
- what the persistent verification lifecycle is;
- how evidence, verification, approval, rejection, failures, results, and history behave;
- how the verified existing-system defects are responded to;
- what is inside and outside implementation scope.

### 1.2 Governing inputs

This blueprint is derived from:
1. `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Report.md`
2. `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Completion_Report.md`
3. `00_PROJECT_CONTROL/Chat40_Day15_Node7_Phase1b_Reviewer_Mental_Model_Decisions.md`
4. The seven Reviewer Interaction Mapping decisions locked during Chat40 / Day16.

### 1.3 Authority rule

The blueprint consolidates these sources; it does not silently replace them.
- Existing-system facts remain authoritative to the investigation records.
- Mental-model decisions remain authoritative to the locked mental-model record.
- Interaction behavior remains authoritative to the seven locked interaction decisions.
- Requirements not established by the governing sources must not be invented as already approved.
- Any conflict discovered during independent review must be surfaced and resolved before implementation.

### 1.4 Approval state

This document is intentionally **DRAFT — FOR CLAUDE ALIGNMENT REVIEW**.

Implementation preparation must wait until the alignment review is resolved and Ayush explicitly approves/locks the resulting blueprint.

---

## 2. Reviewer Responsibility Boundary

### 2.1 Primary job

**Identity & Evidence Verifier.**

The Reviewer verifies whether the applicant’s submitted evidence supports the applicant’s claimed Driver or Company identity/role and then makes the final verification decision.

### 2.2 Reviewer responsibilities

The Reviewer:
- enters the verification workflow as an authorized Reviewer;
- works on applicants whose verification is pending;
- sees Applicant + Claimed Role + Evidence as the essential context;
- examines submitted onboarding evidence;
- evaluates whether evidence supports the claimed identity and claimed role;
- explicitly confirms **Identity / Role Verified** after evidence evaluation;
- approves or rejects the applicant through the final decision flow;
- provides a required rejection reason when rejecting;
- references completed decisions through Verification History.

### 2.3 Outside Reviewer responsibility

The Reviewer is not responsible for:
- trip operations;
- delivery operations;
- claims;
- Driver operations;
- Company operations;
- operational trip/delivery evidence review;
- general platform administration;
- AI-based verification or scoring.

### 2.4 Responsibility boundary principle

The redesign must make the Reviewer’s actual verification responsibility clearer without expanding the Reviewer persona into a general administrator or operations role.

---

## 3. Reviewer Mental Model

### 3.1 Locked mental model

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

### 3.2 Locked decisions represented in this blueprint

- **Primary Job:** Identity & Evidence Verifier.
- **Primary Object:** Evidence.
- **Information Model:** Evidence + Applicant + Requested Role.
- **Verification / Decision Model:** Applicant + Role + Evidence → Evaluate → Verify → Approve/Reject.
- **State Model:** Pending Verification → Verified / Rejected.
- **Mental Journey:** Verification-first.
- **Trust & Evidence Model:** Evidence supports the claimed identity/role.
- **Responsibility Boundary:** Narrow verification boundary.
- **Current Mental-Model Problem:** navigation/interaction problems and evidence→verification→decision clarity are one coherent Reviewer verification-workflow problem.
- **Principles:** Evidence-centered, identity-aware, decision-driven.

### 3.3 Explicit non-expansions

The mental model does not authorize:
- AI verification;
- automated verification;
- evidence scoring;
- confidence scoring;
- mandatory multiple evidence sources;
- persistent `under_review` state;
- trip/delivery review;
- claims handling;
- general administration.

---

## 4. Information Architecture

### 4.1 Final Reviewer surface model

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

The Reviewer experience is organized around the complete verification workflow rather than the existing single flat queue surface.

### 4.2 Verification Queue

**Purpose:** unfinished Reviewer work.

The Queue represents applicants in **Pending Verification**.

The Queue must:
- provide entry for an authorized Reviewer;
- list pending applicants;
- present applicant identity context and claimed/requested role;
- provide direct submitted-evidence access;
- provide a dedicated **Review** action for each applicant;
- preserve Pending Verification when an applicant is only opened for review.

Opening an applicant must not silently commit a persistent state change.

### 4.3 Applicant Verification

**Purpose:** dedicated workspace for one pending applicant.

The view contains:
- Applicant email;
- Claimed Role;
- Submitted Evidence;
- dedicated evidence viewing area;
- clear **Back to Verification Queue** navigation;
- the explicit verification and final decision interactions defined in Section 6.

### 4.4 Decision Result

After a successful final decision, the result state clearly communicates:
- Applicant email;
- Claimed role;
- Final decision (Verified or Rejected);
- rejection reason when applicable;
- clear return action to Verification Queue.

### 4.5 Verification History

**Purpose:** completed decision records.

History is:
- a simple chronological list;
- ordered newest decision first;
- initially without filtering;
- paginated for larger sets;
- equipped with a clear empty state when no completed records exist.

### 4.6 Read-only Verification Record

Selecting a completed History entry opens a dedicated read-only verification record.

It provides:
- Applicant;
- Claimed Role;
- Final Decision;
- rejection reason when applicable;
- decision date/time context;
- access to submitted evidence;
- a submitted-evidence viewer within the same read-only record;
- **Back to Verification History** navigation.

Completed decisions cannot be changed from History.

---

## 5. State Model

### 5.1 Persistent lifecycle

```text
Pending Verification
        ↓
     Evaluate
        ↓
 ┌──────┴──────┐
 ↓             ↓
Verified     Rejected
```

### 5.2 Persistent-state rules

1. A pending applicant enters the workflow as **Pending Verification**.
2. Opening an applicant does not alter persistent state.
3. Viewing/examining evidence does not alter persistent state.
4. Leaving unfinished review does not create persistent `Under Review` state.
5. **Identity / Role Verified** is a human verification action, not a new persistent lifecycle state.
6. The Reviewer may undo/reconsider the verification action before final decision.
7. Final **Approve** commits the Verified outcome.
8. Final **Reject** commits the Rejected outcome.
9. A completed decision is not editable through Verification History.
10. If a final decision request fails, the applicant remains Pending Verification.

### 5.3 Critical distinction

```text
UI verification action ≠ persistent verification state

Identity / Role Verified
        ↓
   human confirmation
        ↓
Approve / Reject
        ↓
persistent outcome
```

---

## 6. Complete Interaction Contract

This section is the complete consolidated contract for all seven locked Reviewer interactions.

### Interaction 1 — Reviewer Entry → Verification Queue

1. An authorized Reviewer enters directly into the **Verification Queue**.
2. The Queue provides direct preview/access to submitted evidence; no artificial/generated **Evidence Summary** is introduced.
3. Each applicant has a dedicated **Review** action.
4. Review opens **Applicant Verification** containing Applicant, Claimed Role, Submitted Evidence, and a dedicated evidence-viewing area.
5. Opening an applicant does not change persistent state; the applicant remains Pending Verification.

### Interaction 2 — Queue → Applicant Verification

1. Applicant Verification immediately presents applicant identity, claimed role, and submitted evidence.
2. A clear **Back to Verification Queue** control is provided.
3. Applicant identity context is **email + claimed role**; unnecessary profile information is not introduced.
4. Evidence is presented within the same verification view.
5. If evidence cannot be viewed because of a technical/access problem, the UI shows a clear error with retry/open-access behavior; it does not auto-reject or hide the problem.
6. Leaving unfinished review creates no persistent state/progress; the applicant remains Pending Verification.

### Interaction 3 — Applicant Verification → Evidence Examination

1. A dedicated large evidence viewer exists within Applicant Verification.
2. Applicant email and claimed role remain visible while evidence is examined.
3. The viewer supports viewing, scrolling, and zoom/resize where applicable; download is not a required Reviewer behavior.
4. Exactly one onboarding evidence document is handled under the already-approved onboarding rule:
   - Driver → Driving Licence
   - Company → GST document
5. The Reviewer evaluates whether the evidence supports the applicant’s claimed identity and claimed role.
6. The Reviewer explicitly confirms **Identity / Role Verified** after evaluation; this is a human action, not a new persistent state.
7. If evidence does not adequately support the claimed identity/role, the Reviewer rejects with a required rejection reason.
8. Technical inability to view evidence remains separate from substantive evidence insufficiency.
9. After Identity / Role Verified, the Reviewer remains in the same verification view and Approve becomes enabled; verification and approval remain separate.

### Interaction 4 — Evidence Examination → Identity / Role Verification

1. **Identity / Role Verified** is an explicit action after evidence examination; viewing evidence alone does not constitute verification.
2. After selecting it, the same view shows a clear confirmation/state and enables Approve.
3. The Reviewer may undo/reconsider verification before final Approve/Reject; no new persistent state is created.
4. If the Reviewer has not reached a confident verification conclusion, the applicant remains Pending Verification and the Reviewer may exit/return later. This does not override the separate rule that evidence substantively found insufficient should be rejected with a reason.
5. If the Reviewer selects Identity / Role Verified but leaves before Approve, verification is not committed; the applicant remains Pending Verification.

### Interaction 5 — Verification → Approve / Reject

1. After Identity / Role Verified, the same view clearly shows **Approve** and **Reject**.
2. Approve requires final confirmation before commit.
3. Reject requires a required rejection reason and final confirmation before commit.
4. After final confirmation, decision controls are disabled and a clear processing state is shown until the result is confirmed.
5. Successful decision produces a clear result state and then returns to Verification Queue.
6. Completed applicants are removed from the Pending Verification queue; the Queue represents unfinished work only.
7. If the final decision API/server/network request fails, the applicant remains Pending Verification, a clear error is shown, and retry remains possible; success is never assumed without confirmation.

### Interaction 6 — Decision → Result / Verification History

1. After a successful final decision, clear result state shows Applicant email, Claimed role, Final decision, rejection reason when applicable, and a clear action to return to Verification Queue.
2. A dedicated **Verification History** contains completed Verified/Rejected decisions.
3. History entries show Applicant email, Claimed role, Final decision, and rejection reason when applicable. History is a decision-record view, not a duplicate evidence-examination workspace.
4. Selecting a completed History record opens a **read-only verification record** with Applicant, Claimed role, Final decision, rejection reason if applicable, and access to submitted evidence. Completed decisions cannot be changed from History.
5. The read-only record provides a clear **Back to Verification History** control.

### Interaction 7 — Verification History → Completed Record / Navigation

1. Verification History is a simple chronological list of completed verification records.
2. Records are ordered **newest decision first**.
3. Each entry shows Applicant email, Claimed role, Final decision, and decision date/time.
4. Initial History has **no filtering**.
5. Empty History shows **“No completed verification records yet.”** with a **Back to Verification Queue** action.
6. Larger History sets use **pagination**.
7. Selecting a History entry opens a **separate dedicated read-only verification record**.
8. Submitted evidence is viewable through an evidence viewer within that same read-only record.
9. Evidence is **view-only** in completed History; no Identity / Role Verified, Approve, Reject, or decision-change action is available.
10. Returning to History preserves the Reviewer’s previous History page/position context.

---

## 7. Evidence Rules

### 7.1 Evidence role

Evidence is the primary object of Reviewer reasoning. Applicant and Claimed Role provide the necessary verification context.

### 7.2 Active verification evidence

During active verification:
- submitted evidence is directly accessible;
- evidence is handled in a dedicated viewing area;
- Applicant email and Claimed Role remain available as context;
- the Reviewer evaluates the evidence against the claimed identity/role.

### 7.3 Evidence type boundary

Under the already-approved one-document onboarding rule:
- Driver → Driving Licence;
- Company → GST document.

This blueprint does not create a new onboarding rule.

### 7.4 Evidence viewing behavior

The dedicated viewer supports appropriate viewing, scrolling, and zoom/resize where applicable. Download is not a required Reviewer behavior.

### 7.5 Evidence failure handling

Technical/access failure to view evidence must produce a clear error and retry/open-access path.

Technical viewing failure must not automatically become rejection.

Substantive evidence insufficiency is a separate verification outcome and requires rejection with a reason.

### 7.6 Completed-record evidence

Completed History records retain access to submitted evidence for reference. Evidence in the completed record is strictly view-only.

### 7.7 Prohibited additions

No new:
- evidence types;
- multiple-evidence requirements;
- scoring model;
- confidence model;
- AI verification;
- automated verification mechanism.

---

## 8. Decision Rules

### 8.1 Identity / Role Verification

Identity / Role Verified is an explicit human action following evidence examination.

The Reviewer is not considered to have verified an applicant merely because the evidence was opened or viewed.

The Reviewer may reconsider/undo this verification action before the final decision. The action does not create a new persistent lifecycle state.

### 8.2 Approve flow

```text
Evidence Examination
        ↓
Identity / Role Verified
        ↓
Final Approve Confirmation
        ↓
Processing
        ↓
Server-confirmed success
        ↓
Verified Result
        ↓
Verification Queue
```

### 8.3 Reject flow

```text
Evidence evaluated as insufficient
        ↓
Required Rejection Reason
        ↓
Final Reject Confirmation
        ↓
Processing
        ↓
Server-confirmed success
        ↓
Rejected Result
        ↓
Verification Queue
```

### 8.4 Final decision failure

If the final decision request fails:
- the outcome is not assumed;
- applicant remains Pending Verification;
- a clear error is shown;
- the Reviewer can retry.

### 8.5 Completed-decision boundary

Once a decision is committed as Verified or Rejected, the History record is read-only. History does not reopen the decision workflow.

---

## 9. History Rules

### 9.1 Purpose

Verification History is the Reviewer’s completed-decision reference surface.

### 9.2 List model

- Simple chronological list.
- Newest decision first.
- No filtering in the initial workflow.
- Pagination for larger result sets.

### 9.3 Entry information

Each History entry contains:
- Applicant email;
- Claimed Role;
- Final Decision;
- Decision date/time;
- rejection reason when applicable.

### 9.4 Empty History

When no completed records exist, show:

> **No completed verification records yet.**

and provide **Back to Verification Queue**.

### 9.5 Completed record

Selecting a History entry opens a dedicated read-only verification record containing:
- Applicant;
- Claimed Role;
- Final Decision;
- rejection reason when applicable;
- access to submitted evidence.

The submitted evidence viewer remains inside the same read-only record.

### 9.6 Read-only rule

From History, the Reviewer cannot:
- change the final decision;
- approve;
- reject;
- select Identity / Role Verified;
- reopen the completed workflow.

The record provides **Back to Verification History**.

### 9.7 Context preservation

Returning to History preserves the Reviewer’s previous History page/position context.

---

## 10. Navigation & Responsive Requirements

### 10.1 Required navigation relationships

```text
Reviewer Entry
      ↓
Verification Queue
      ↓
Applicant Verification
      ↓
Back to Verification Queue

Successful Decision
      ↓
Verification Queue

Verification History
      ↓
Read-only Verification Record
      ↓
Back to Verification History
```

### 10.2 Existing navigation trap response

The Existing-System Investigation verified that the shared Navbar exposes Dashboard and Timeline paths that can bounce a Reviewer back to the Queue. The redesigned Reviewer experience must not preserve that navigation trap.

Reviewer navigation must expose the actual verification workflow rather than forcing the Reviewer into unrelated or looping navigation.

### 10.3 Explicit navigation rule

Primary workflow navigation must use explicit Reviewer-appropriate controls, including:
- **Back to Verification Queue** from active Applicant Verification;
- **Back to Verification History** from the completed read-only record.

Browser-back is not the sole primary navigation contract.

### 10.4 Responsive requirement

The complete Reviewer workflow must remain usable on:
- desktop;
- tablet;
- mobile.

The exact responsive component layout, dimensions, and evidence-viewer arrangement remain implementation-detail decisions, provided they preserve the locked behavior and do not introduce unsupported functionality.

---

## 11. Existing-System Defects → Blueprint Response

The Existing-System Investigation verified four Reviewer defects.

### REV-01 — Navigation Trap

**Verified baseline:** Shared navigation exposes Dashboard/Timeline paths that redirect/bounce the Reviewer back to Queue.

**Blueprint response:** Replace the defective Reviewer navigation behavior with role-appropriate navigation representing Verification Queue, Applicant Verification, decision result, and Verification History without redirect loops.

### REV-02 — Role-Confusion Lockout

**Verified baseline:** Reviewer authorization is checked before Freight Identity and can unconditionally redirect a dual-role user to `/reviewer/queue`, preventing normal operational dashboard access.

**Blueprint response:** Do not preserve an unconditional dual-role lockout. The exact multi-role routing resolution must remain separately verified/approved before implementation if not already specified by another locked decision.

### REV-03 — RLS Bypass Architecture

**Verified baseline:** Reviewer queue/API operations rely on Supabase service-role access rather than explicit Reviewer RLS policies for the core data mutations/reads; authorization is enforced primarily through manual Reviewer checks.

**Blueprint response:** Correct the security architecture so Reviewer data access/mutation does not unnecessarily depend on service-role god-mode access. The exact secure RLS/authorization implementation must be separately verified and approved before execution.

### REV-04 — Degraded UX / Native Prompt

**Verified baseline:** Rejection uses native browser `prompt()`, errors are raw inline text, and success is represented by a refresh without a dedicated result state.

**Blueprint response:** Replace the native prompt with the application-approved rejection input/confirmation pattern and provide clear processing, success, and failure states consistent with the locked interaction workflow.

### 11.1 Existing behavior to preserve unless a verified change is required

- Reviewer authorization remains required for Reviewer access.
- Evidence storage protection must not be weakened.
- Existing onboarding identity/evidence data should be reused where compatible.
- Reviewer remains outside trip/delivery operational authority.

---

## 12. Implementation Boundary

### 12.1 In scope

The implementation may include:
- Reviewer-specific entry/routing redesign required by the locked workflow and verified defects.
- Verification Queue redesign.
- Dedicated Applicant Verification view.
- Dedicated evidence examination/viewer experience.
- Applicant + Claimed Role + Evidence context.
- Explicit Identity / Role Verified interaction.
- Approve/Reject confirmation behavior.
- Required rejection reason input.
- Processing state.
- Clear success state.
- Clear failure/error state.
- Verification History.
- Dedicated read-only completed verification record.
- View-only evidence access from completed records.
- Explicit Reviewer workflow navigation.
- Responsive Reviewer workflow across desktop/tablet/mobile.
- Verified Reviewer UX/routing defect fixes.
- Required security correction for the verified service-role/RLS bypass defect, after the secure implementation design is independently verified and approved.
- Reuse of existing APIs/data where compatible with the locked behavior.

### 12.2 Out of scope unless separately verified and approved

- New Reviewer business responsibilities.
- Trip management.
- Delivery management.
- Claims.
- Marketplace/claim mechanisms.
- Operational trip/delivery evidence review.
- AI verification.
- Automated verification.
- Evidence scoring/confidence scoring.
- New persistent `under_review` workflow state.
- New evidence types.
- Mandatory multiple-evidence requirements.
- General Reviewer metrics/admin dashboard as a new responsibility.
- Unverified backend business functionality.
- New authorization rules unrelated to the verified defect/security requirements.

### 12.3 Implementation discipline

Antigravity must implement only from an approved implementation plan that traces back to this blueprint.

Antigravity must not:
- infer new Reviewer business rules from visual gaps;
- invent additional persistent states;
- expand Reviewer responsibility;
- add AI/scoring/automation;
- treat an unverified security assumption as a completed fix.

Investigation and fix remain separate. Security/API fixes require their own verification evidence and are not complete merely because the UI appears to work.

---

## 13. Acceptance / Completion Criteria

### 13.1 Source alignment criteria

The blueprint is acceptable only if independent review confirms:
- all 13 sections remain traceable to the governing sources;
- no section contradicts a locked mental-model or interaction decision;
- existing-system facts are represented accurately;
- unsupported assumptions are identified rather than silently accepted.

### 13.2 Interaction acceptance

- All seven locked interactions are represented.
- Verification-first journey is preserved.
- Applicant + Claimed Role + Evidence remain the core context.
- Evidence examination precedes Identity / Role Verified.
- Identity / Role Verified remains an explicit human action.
- Approve/Reject remain separate final decisions.
- Successful decisions produce a clear result and return to Queue.
- Completed work is available through History.
- History records are read-only.

### 13.3 State acceptance

- Persistent lifecycle remains Pending Verification → Verified / Rejected.
- No persistent `under_review` state is introduced.
- Opening/evidence examination/leaving unfinished review does not create a persistent review state.
- Final decision failure preserves Pending Verification.

### 13.4 Evidence acceptance

- Evidence remains central to verification.
- The already-approved one-document onboarding rule remains unchanged.
- Technical evidence-view failure is separate from substantive evidence insufficiency.
- Completed evidence is view-only.
- No AI/scoring/multiple-evidence behavior is added.

### 13.5 Defect acceptance

- REV-01 through REV-04 have explicit blueprint responses.
- Evidence storage protection is preserved.
- Security correction is independently verified before being declared complete.

### 13.6 Scope acceptance

- Reviewer responsibility remains narrow.
- No trip/delivery/claims/general-admin responsibility is introduced.
- No unsupported backend business functionality is added.
- Implementation remains within the stated boundary.

### 13.7 Independent Claude review contract

Claude must review **all 13 sections individually**, not just the document as a whole.

For each section, Claude must classify:
- **✅ Fully Aligned**
- **⚠️ Needs Clarification**
- **❌ Contradiction / Unsupported**

For every ⚠️ or ❌ finding, Claude should identify:
- the exact blueprint section/subsection;
- the exact conflict, omission, or unsupported assumption;
- the governing source and decision/evidence it conflicts with.

Claude must **not redesign the blueprint** or add new functionality during this review.

### 13.8 Current status

**DRAFT — FOR CLAUDE ALIGNMENT REVIEW**

No implementation should begin until the alignment review is resolved and Ayush explicitly approves/locks the blueprint.
