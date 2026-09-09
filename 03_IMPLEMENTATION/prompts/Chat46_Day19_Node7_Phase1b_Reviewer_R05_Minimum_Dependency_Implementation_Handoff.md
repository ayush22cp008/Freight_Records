# Chat46 / Day19 / Node7 / Phase1b
# Reviewer R-05 Minimum Dependency Implementation Handoff

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign  
**Day:** Day 19  
**Chat:** Chat46  
**Execution owner:** Antigravity  
**Reasoning / governance owner:** ChatGPT  
**Final authority:** Ayush  
**Status:** **AUTHORIZED NARROW R-05 DEPENDENCY WORK**

---

## 1. Purpose

This handoff authorizes Antigravity to implement only the minimum backend/data-access/security dependencies required to make the locked Reviewer Verification History (R-05) technically supportable.

This is **not** the Reviewer frontend implementation prompt.

Do not redesign the Reviewer UI in this task.

Do not expand scope beyond the exact R-05 dependencies authorized below.

The GitHub Records repository is the bridge between ChatGPT architecture/governance and Antigravity execution. Record all implementation evidence and findings back through the Records repository.

---

## 2. Governing Records

Read these before touching source code:

1. `00_PROJECT_CONTROL/ROADMAP.md`
2. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
3. `00_PROJECT_CONTROL/PROJECT_STATE.md`
4. `00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Governance_Decision.md`
5. `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
6. `02_ARCHITECTURE/Chat46_Day19_Node7_Phase1b_Reviewer_Implementation_Gap_R05_Boundary_Decision.md`
7. `01_BRAIN_HANDOFFS/Claude/Chat46_Day19_Node7_Phase1b_Reviewer_Implementation_Gap_Boundary_Decision_Claude_Review.md`
8. `05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Whole_Existing_System_Investigation_Report.md`
9. `03_IMPLEMENTATION/prompts/Chat42_Day16_Node7_Phase1b_Master_Implementation_Prompt.md`

Treat the locked Blueprint and the approved R-05 governance decision as authoritative for scope. Existing-system facts must be verified directly in the codebase before implementation.

---

## 3. Approved Authorization

Ayush explicitly approved **Option B** in the R-05 Final Governance Decision.

Authorized scope is limited to the minimum dependencies required for Verification History:

### A. Persisted decision timestamp

Add one persisted timestamp such as `reviewed_at`, or another explicitly justified equivalent, so the final verification decision has a reliable decision-time value.

Requirements:
- populated as part of the existing final decision path;
- preserve current Verified/Rejected semantics;
- no new lifecycle state;
- no `under_review` persistence;
- no unrelated schema redesign;
- use the smallest compatible schema/data-model change.

### B. Completed-record read path

Provide one narrow read mechanism that allows the Reviewer workflow to retrieve completed:
- Verified records;
- Rejected records;
- ordered by the approved decision timestamp, newest first;
- a selected completed record for read-only History detail.

The mechanism may reuse or extend existing server-side patterns only when doing so remains within the approved R-05 scope and preserves authorization boundaries.

Do not create a general admin/reporting interface.

### C. Minimal authorization/security support

Provide only the minimum corresponding authorization/RLS/security change required to make the completed-record read path valid and secure for the Reviewer use case.

Do not broaden Reviewer permissions beyond the completed verification-record read requirement.

---

## 4. Explicitly Out of Scope

Do not implement or modify:

- Reviewer frontend redesign;
- Reviewer History UI;
- Reviewer navigation redesign;
- Applicant Verification UI redesign;
- Evidence Examination UI redesign;
- rejection modal/UI redesign;
- Identity / Role Verified frontend interaction;
- authentication changes;
- role-model redesign;
- unrelated authorization changes;
- unrelated RLS policies;
- unrelated database/schema changes;
- new API contracts unrelated to R-05;
- new evidence types or evidence requirements;
- new persistent lifecycle states;
- persistent `under_review` state;
- business-rule changes;
- trip/delivery functionality;
- claiming/marketplace behavior;
- AI verification/scoring/automation;
- Reviewer authority expansion;
- C-05;
- R-03.

Do not use this task to clean up unrelated technical debt.

---

## 5. Mandatory Pre-Implementation Investigation

Before implementation, inspect the actual source and confirm:

1. current final decision write path;
2. current identity and evidence tables/fields involved in decision persistence;
3. existing timestamp fields and their update semantics;
4. current Reviewer authorization checks;
5. current RLS policies affecting identity/evidence access;
6. current Queue read/query mechanism;
7. whether an existing secure read mechanism can support completed records without introducing a new contract;
8. exact minimal change set needed for R-05.

Do not assume the prior investigation is sufficient for implementation details. Re-verify the source before changing it.

Classify important findings as:
- VERIFIED
- INFERRED
- UNKNOWN

If implementation can be achieved with less change than the approved minimum, use the smaller safe change and record why.

---

## 6. Required Implementation Behavior

The resulting backend/data capability must support these R-05 facts:

### 6.1 Decision time

A successful final Approve or Reject decision must persist a reliable decision timestamp.

The timestamp must represent the decision commit event, not page-load time or applicant creation time.

### 6.2 Completed records

Completed records must be distinguishable through the existing Verified/Rejected status model.

Do not introduce another completion state.

### 6.3 Ordering

The completed-record query must support deterministic newest-first ordering based on the approved decision timestamp.

If ties are possible, use an existing stable identifier as a secondary ordering key without changing business semantics.

### 6.4 Selected completed record

The read path must support retrieving one completed record sufficient for the future read-only Verification Record experience.

The returned data should be limited to what the locked Blueprint requires, including applicant identity context, claimed role, final decision, rejection reason when applicable, decision timestamp, and access linkage needed for existing submitted-evidence viewing.

Do not add unrelated applicant/profile data.

### 6.5 Evidence mechanism

Reuse the existing submitted-evidence/signed-URL mechanism where possible.

Do not create a new evidence type, storage architecture, or evidence access model.

---

## 7. Authorization / Security Requirements

The new completed-record read capability must be secure by construction.

Specifically:

- Reviewer access must be explicit and scoped to the intended completed verification data.
- Do not rely on accidental access or client-side hiding.
- Do not expose service-role credentials to the client.
- Do not bypass RLS/security without documenting the exact server-only boundary and why it is necessary.
- Do not broaden access to unrelated rows or roles.
- Preserve existing authentication and role semantics.

If a secure implementation requires an authorization design different from the expected minimal RLS adjustment, STOP and report the dependency before expanding scope.

---

## 8. API / Contract Discipline

Preserve existing final decision semantics.

A new completed-record read path may be introduced only if necessary and only for the R-05 requirement.

Do not modify unrelated APIs.

Do not change response shapes outside what is strictly required to persist/read the authorized R-05 information.

If an existing API can be safely extended instead of adding a new endpoint, evaluate that option first, but do not force reuse if it would weaken authorization or create coupling outside scope.

---

## 9. Validation Requirements

After implementation, run relevant checks covering at minimum:

### Decision timestamp
- Approve persists decision timestamp.
- Reject persists decision timestamp.
- Timestamp remains tied to the final decision event.

### Completed-record read
- Verified completed records are retrievable.
- Rejected completed records are retrievable.
- Pending records are not incorrectly treated as completed History records.
- Newest-first ordering works using the decision timestamp.
- A selected completed record can be retrieved.

### Authorization/security
- Authorized Reviewer can retrieve only the intended completed verification data.
- Unauthorized roles cannot use the new capability.
- No client-side security bypass exists.
- Existing Pending Queue behavior remains functional.
- Existing Approve/Reject behavior remains functional.

### Regression
- No unrelated lifecycle/state behavior changed.
- No evidence-type behavior changed.
- No C-05/R-03 behavior changed.

Record exact checks/tests performed.

---

## 10. Stop Conditions

Stop immediately and hand back through the Records repository if any of the following occurs:

- more than the approved single timestamp/schema dependency is required;
- broad schema redesign appears necessary;
- broad RLS redesign is required;
- authentication or role-model changes are required;
- new lifecycle states are required;
- new business rules are required;
- new evidence requirements are required;
- Reviewer authority would expand;
- a new API contract materially changes unrelated behavior;
- the locked Blueprint conflicts with source behavior in a way requiring a new architecture decision;
- a safe implementation cannot be achieved inside this narrow authorization.

Do not silently expand the task.

---

## 11. Required Output / Implementation Report

Create the implementation report through the GitHub Records bridge at:

`03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Minimum_Dependency_Implementation_Report.md`

The report must include:

1. implementation summary;
2. exact files changed;
3. exact schema/data changes;
4. exact API/read-path changes;
5. exact RLS/authorization changes;
6. reason for each meaningful change;
7. tests/checks run;
8. build/runtime findings;
9. security findings;
10. VERIFIED / INFERRED / UNKNOWN classification;
11. known limitations;
12. any stop-condition or scope concern;
13. final implementation status.

Do not claim R-05 is READY merely because code has been changed.

---

## 12. Post-Implementation Gate

After the implementation report is recorded:

1. Stop.
2. Do not begin Reviewer frontend implementation.
3. Wait for ChatGPT to review the implementation evidence.
4. R-05 must undergo a **fresh readiness assessment** against every locked History requirement.
5. Only after the fresh assessment returns **R-05 READY** may a separate Reviewer frontend implementation prompt be created/used.

The current task ends after the authorized R-05 dependency work and evidence are recorded.

---

## 13. Final Execution Rule

```text
INVESTIGATE
    ↓
IMPLEMENT ONLY APPROVED R-05 DEPENDENCIES
    ↓
BUILD / TEST / SECURITY CHECK
    ↓
RECORD EVIDENCE IN Freight_Records
    ↓
STOP
    ↓
CHATGPT FRESH R-05 READINESS REVIEW
    ↓
ONLY IF READY → REVIEWER FRONTEND IMPLEMENTATION
```

**Current authorization:** APPROVED — NARROW R-05 DEPENDENCY ONLY  
**Reviewer frontend authorization:** NOT YET GRANTED  
**R-05 current status:** NOT READY  
**Next gate after execution:** FRESH R-05 READINESS ASSESSMENT
