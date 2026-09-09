# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Frontend Implementation Handoff

### Authorization State

**IMPLEMENTATION AUTHORIZED.**

The final pre-implementation architecture/readiness gate is closed.
Claude's final independent architecture-readiness review concluded:

> `SAFE TO START REVIEWER IMPLEMENTATION`

R-05 is READY, the Reviewer Blueprint is LOCKED, and no material architecture contradiction or blocker remains in the Records evidence.

This handoff is now the execution boundary for implementing the Reviewer frontend.

---

## 1. Objective

Implement the Reviewer Portal frontend according to the **LOCKED Reviewer Blueprint** and the currently approved project boundaries.

This is a frontend implementation task around the already-settled architecture and already-supported backend/data capabilities.

Do not redesign the Reviewer architecture during implementation.

Do not reopen completed discovery, gap analysis, R-05 governance, or readiness decisions unless a genuinely new blocker is discovered during implementation.

---

## 2. Governing Records — Read Before Editing

Read these Records first and treat them as the implementation contract:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md

02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md

05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Whole_Existing_System_Investigation_Report.md
02_ARCHITECTURE/Chat46_Day19_Node7_Phase1b_Reviewer_Implementation_Gap_R05_Boundary_Decision.md
01_BRAIN_HANDOFFS/Claude/Chat46_Day19_Node7_Phase1b_Reviewer_Implementation_Gap_Boundary_Decision_Claude_Review.md

00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Governance_Decision.md
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Minimum_Dependency_Implementation_Report.md
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Runtime_Validation_Retest_Report.md
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Readiness_Decision.md

01_BRAIN_HANDOFFS/Claude/Chat46_Day19_Node7_Phase1b_Reviewer_Final_Implementation_Readiness_Architecture_Audit_Claude_final_review.md
```

The locked Blueprint is the primary product/UX authority.
The R-05 governance and readiness records define the authorized dependency boundary.

---

## 3. Locked Reviewer Target

Implement the locked Reviewer information architecture and workflow:

```text
Reviewer
  ↓
Verification Queue
  ↓
Applicant Verification
  ↓
Evidence Examination
  ↓
Decision Result
  ↓
Verification History
  ↓
Read-only Verification Record
  ↓
Submitted Evidence Viewer
```

The Reviewer responsibility remains **Identity & Evidence Verifier**.

The Reviewer verifies applicants using the existing identity/evidence model and approved decision mechanism.

The implementation must remain consistent with the locked states:

```text
Pending Verification
       ↓
Verified / Rejected
```

Do not introduce a persistent `under_review` state.

---

## 4. Required Reviewer Implementation Surfaces

Implement every surface and interaction required by the locked Blueprint.

### A. Reviewer shell / navigation

Build the Reviewer-specific shell and navigation defined by the Blueprint and the shared cross-portal design system.

Navigation must correctly connect the required Reviewer surfaces without introducing unrelated portal destinations.

### B. Verification Queue

Implement the queue as the Reviewer entry surface.

It must clearly communicate the applicant records awaiting verification and lead into the applicant verification experience.

Preserve the established identity/authorization model.

### C. Applicant Verification

Implement the applicant verification view using the existing applicant information available from the current system.

The view should make the verification task understandable without requiring the Reviewer to infer the workflow.

### D. Evidence Examination

Implement the evidence examination experience using the already-supported evidence source and signed-access mechanism.

Do not create a new evidence storage model, upload model, document model, or AI evidence system.

### E. Identity / Role Verified

Implement **`Identity / Role Verified` as its own explicit Reviewer interaction/item**.

This was identified in the previous Claude gap review as a frontend gap and is specifically required by the final alignment review.

Do not convert this into a new persistent backend status unless a separate approved decision exists. It is a Reviewer human-action/verification interaction within the frontend workflow.

### F. Approve / Reject

Implement the locked decision controls:

```text
Approve
Reject
```

Reject must require the required rejection reason.

Use the existing approved `/api/admin/review` decision mechanism.

Do not redesign the decision API or lifecycle.

### G. Decision Result

After a decision, show the locked Result state clearly and consistently.

The user must be able to understand whether the applicant was Verified or Rejected and, for rejection, the relevant reason.

### H. Verification History

Implement the locked Verification History surface using the now-ready R-05 capability.

The History experience must support:

- completed Verified and Rejected records;
- newest-first ordering;
- pagination;
- selected completed record;
- read-only record viewing;
- evidence linkage/viewing using the existing mechanism.

Use the already-authorized history read path:

```text
GET /api/admin/history
```

Do not create another backend read architecture.

### I. Read-only Verification Record

A completed history record must be inspectable without exposing editable decision controls.

The record view should clearly distinguish historical/read-only status from a pending verification task.

### J. Submitted Evidence Viewer

Allow the Reviewer to inspect linked submitted evidence for the selected completed record using the existing signed URL/evidence mechanism.

Do not introduce new evidence infrastructure.

---

## 5. Backend/Data Boundary

The backend/data work required for Reviewer implementation has already been completed and approved.

The authorized R-05 surface is limited to the capabilities represented by:

```text
reviewed_at
GET /api/admin/history
```

and the associated minimum Reviewer authorization/security support already implemented and runtime-validated.

### Do not make additional backend changes unless a genuine implementation blocker is discovered and escalated.

No silent changes to:

```text
Authentication / identity
Role model
Business rules
Trip creation / publishing
Driver marketplace / claiming
Delivery lifecycle
Evidence architecture
AI behavior
Unrelated schema
Unrelated APIs
Unrelated RLS/security
C-05
R-03
Reviewer authority
```

If implementation appears to require any such change, **STOP** and report the dependency instead of implementing it.

---

## 6. Shared Design System

Use the already-locked shared Cross-Portal Design System decisions.

Reviewer should feel like the same product family as the already accepted/locked Driver and Company portals while maintaining the Reviewer-specific information architecture and authority model.

Do not introduce an independent design language.

Preserve the established visual hierarchy, spacing, typography, interaction patterns, responsiveness, and accessibility/readability expectations from the shared system.

---

## 7. Implementation Rules

### Preserve existing business behavior

Do not change existing business rules merely to make UI implementation easier.

### Reuse existing APIs and data contracts

Prefer existing supported endpoints and data structures.

Do not duplicate backend logic in the frontend.

### No hidden architecture expansion

A UI requirement does not automatically authorize a backend change.

When a requirement is already supported, wire the frontend to it.

When a requirement is not supported and would require a protected-area change, stop and escalate.

### No historical rewriting

Do not modify old project records to make the implementation look cleaner.

Record the implementation outcome in a new implementation report.

---

## 8. Build / Test / Evidence Requirements

After implementation:

1. Run the appropriate build.
2. Run relevant automated tests/checks.
3. Verify routes/navigation compile and behave correctly.
4. Verify the Reviewer workflow end-to-end at the frontend level.
5. Verify loading, empty, error, and decision-result states where applicable.
6. Verify responsive behavior and readability.
7. Verify that History uses the R-05 authorized data source and remains read-only for completed records.
8. Record concrete evidence rather than merely stating that something works.

Do not mark the implementation Accepted or Locked at this stage.

Acceptance remains a later gate after Ayush manual verification.

---

## 9. Mandatory Stop Conditions

Stop implementation and report the issue if any of the following occurs:

- Locked Blueprint requirement cannot be implemented within current approved capabilities.
- A new database/schema change is required outside the approved R-05 dependency.
- A new API is required outside the approved R-05 history path.
- New or broader RLS/security changes appear necessary.
- Authentication or role-model changes appear necessary.
- Reviewer authority would need to expand beyond the locked Blueprint.
- Business/lifecycle/evidence/AI behavior would need to change.
- Existing Driver or Company accepted/locked behavior would need to change to support Reviewer.
- Any contradiction is discovered between implementation requirements and the locked architecture.

Do not work around a stop condition by silently changing architecture.

---

## 10. Required Implementation Report

Create:

```text
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Frontend_Implementation_Report.md
```

The report must include:

- implementation scope;
- files/components/routes changed;
- Blueprint requirements implemented;
- API/data sources used;
- Identity / Role Verified implementation evidence;
- History implementation evidence;
- Decision Result evidence;
- Evidence Viewer evidence;
- build/test results;
- responsive/accessibility/readability checks;
- any deviations;
- any discovered blockers;
- exact verification status.

Use:

```text
VERIFIED
INFERRED
UNKNOWN
```

accurately. Do not claim VERIFIED without concrete evidence.

---

## 11. Execution Sequence

```text
READ LOCKED BLUEPRINT + GOVERNING RECORDS
                ↓
INSPECT CURRENT REVIEWER FRONTEND IMPLEMENTATION STATE
                ↓
IMPLEMENT LOCKED REVIEWER FRONTEND
                ↓
BUILD
                ↓
TEST / CHECK
                ↓
RECORD EVIDENCE
                ↓
STOP
                ↓
AYUSH MANUAL VERIFICATION / REVIEW
                ↓
FIX VERIFIED ISSUES ONLY
                ↓
RE-TEST
                ↓
AYUSH ACCEPTANCE
                ↓
REVIEWER LOCK
```

The implementation agent must stop after the implementation/build/test/evidence stage and wait for Ayush's manual verification/acceptance gate.

---

## 12. Final Instruction to Execution Agent

Implement the **LOCKED Reviewer Blueprint**, not a reinterpretation of it.

The architecture/readiness gate is closed and implementation is authorized.

The objective now is execution with strict scope control:

**Reviewer frontend → build/test → evidence → stop for Ayush verification.**

Do not declare the Reviewer accepted or Phase 1b complete yourself.
