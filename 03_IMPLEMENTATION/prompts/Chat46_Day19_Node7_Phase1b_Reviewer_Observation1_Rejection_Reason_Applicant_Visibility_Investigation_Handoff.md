# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 1 Investigation Handoff — Rejection Reason Visibility to Applicant

### Investigation Type

**INVESTIGATION ONLY — NO FIX AUTHORIZED**

This handoff investigates one specific manual-verification observation in the Reviewer Portal.

Do not implement a fix during this task.
Do not modify the application source code.
Do not modify database schema.
Do not modify APIs.
Do not change the locked Reviewer Blueprint.
Do not change applicant-facing behavior.

The purpose is to establish the evidence-backed **root cause** first. A separate implementation prompt will be created only after the investigation report is reviewed and a fix decision is explicitly made.

---

## 1. Observation

During Ayush's manual Reviewer verification, the Reviewer successfully rejected an applicant with a concrete rejection reason.

The Reviewer side showed the rejection reason correctly, and the rejected record later appeared in Verification History with the reason.

However, the affected applicant's onboarding/status page displayed only a generic rejection message and did not show the actual reason entered by the Reviewer.

Observed behavior:

```text
Reviewer
  ↓
Reject Applicant
  ↓
Enter rejection reason
  ↓
Confirm Rejection
  ↓
Reviewer sees rejection result + reason
  ↓
History contains rejected record + reason

BUT

Applicant
  ↓
Application Rejected
  ↓
Generic message only
  ↓
Actual rejection reason not visible
```

This observation must be investigated as a data-flow / presentation-flow problem. Do not assume the root cause from the UI alone.

---

## 2. Investigation Question

Answer exactly:

> **Why is the persisted rejection reason visible to the Reviewer and/or History, but not visible on the applicant-facing rejection/onboarding page?**

Determine the actual root cause from source and runtime evidence.

---

## 3. Required Investigation Path

Trace the complete rejection-reason lifecycle:

```text
Reviewer Reject UI
      ↓
Rejection reason input
      ↓
POST /api/admin/review
      ↓
Database persistence
      ↓
rejection_reason field
      ↓
Applicant-facing status/query/API
      ↓
Applicant onboarding page
      ↓
Rendered rejection message
```

The investigation must establish where the actual value is lost, omitted, blocked, or never requested.

---

## 4. Questions to Answer

### A. Reviewer-side source

Identify:

- where the Reviewer enters the rejection reason;
- the field name used by the frontend;
- the request payload sent to the decision API;
- where the API receives the rejection reason.

### B. Persistence

Verify:

- the exact database field storing the rejection reason;
- whether the value is actually persisted after rejection;
- whether the rejected test applicant's database record contains the expected reason.

Do not create or alter test data unless strictly necessary for investigation.

### C. Reviewer / History retrieval

Determine how Reviewer Result and Verification History obtain the rejection reason.

Establish whether those surfaces read the same persisted field and from which endpoint/query.

### D. Applicant-facing retrieval

Trace the applicant-facing rejection/status page:

- route/page/component;
- server/client data source;
- API or direct database query;
- selected fields;
- filtering logic;
- authorization logic;
- transformation/mapping logic.

Explicitly check whether `rejection_reason` is:

```text
queried → omitted → renamed → discarded → unavailable
```

### E. Applicant-facing rendering

Determine whether the applicant page:

- receives the rejection reason but does not render it;
- never receives the rejection reason;
- receives a different/empty field;
- intentionally suppresses the reason;
- encounters an authorization/data-access limitation.

### F. Authorization / privacy boundary

Determine whether showing the rejection reason to the applicant is already compatible with the existing authorization model.

Do not assume a new authorization rule is required.

Also determine whether exposing the rejection reason could accidentally expose Reviewer-only information. The investigation must separate the applicant's own rejection reason from internal Reviewer metadata.

---

## 5. Compare With Existing Evidence

Use the current manual evidence and project records as starting evidence, but do not treat them as proof of root cause.

Relevant observations include:

```text
Reviewer Rejected Result → reason visible
Reviewer History         → reason visible
Applicant Rejected Page  → generic text only
```

The investigation must verify the source/data path behind these observations.

---

## 6. Governing Project Records

Read the relevant records before investigation:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md

02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md

02_ARCHITECTURE/Chat46_Day19_Node7_Phase1b_Reviewer_Implementation_Gap_R05_Boundary_Decision.md

00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Governance_Decision.md
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Readiness_Decision.md

03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Frontend_Implementation_Report.md
```

These records define context and boundaries. They do not determine the technical root cause in advance.

---

## 7. Scope Boundary

This investigation is limited to **rejection reason visibility to the rejected applicant**.

Do not investigate or modify:

```text
Identity / Role Verified behavior
Evidence-load gating
Reviewer navigation
Reviewer visual redesign
Authentication redesign
Role model redesign
RLS redesign
Trip/delivery systems
Driver marketplace
Company trip workflows
AI behavior
C-05
R-03
```

Those will be handled separately.

---

## 8. Evidence Standards

Use the project's evidence classification:

```text
VERIFIED
INFERRED
UNKNOWN
```

A root cause may be marked **VERIFIED** only when supported by concrete source/runtime evidence.

Do not write a root cause such as "the applicant page forgot to display the field" unless the source inspection actually demonstrates that behavior.

---

## 9. Required Root-Cause Analysis

The report must explicitly distinguish:

### Observation
What Ayush saw.

### Evidence
What the source/database/runtime inspection proves.

### Root cause
The specific technical reason the applicant cannot see the persisted rejection reason.

### Contributing factors
Any secondary issue that helped produce the observed behavior.

### Not the root cause
Any plausible hypotheses that were investigated and ruled out.

### Scope of the eventual fix
State whether the evidence suggests the eventual fix would be:

- frontend rendering only;
- applicant-facing query/data contract adjustment;
- API adjustment;
- authorization adjustment;
- another narrowly-scoped change.

Do **not** implement the fix yet.

---

## 10. Required Investigation Report

Create a new report:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Investigation_Report.md
```

The report must contain:

1. Investigation objective
2. Exact observation
3. Environment tested
4. Source/data-flow traced
5. Database/persistence evidence
6. Reviewer-side evidence
7. Applicant-facing retrieval evidence
8. Applicant-facing rendering evidence
9. Authorization/privacy assessment
10. Evidence classification (`VERIFIED / INFERRED / UNKNOWN`)
11. Root cause
12. Contributing factors
13. Ruled-out hypotheses
14. Recommended fix scope (recommendation only; no implementation)
15. Stop conditions / unresolved questions
16. Final investigation status

---

## 11. Mandatory Stop Point

After the investigation is complete and the report is created:

**STOP.**

Do not fix anything.
Do not create an implementation prompt.
Do not modify the project architecture.
Do not change the original implementation report.

Wait for ChatGPT/Ayush to review the investigation findings and decide whether a separate implementation prompt should be created.
