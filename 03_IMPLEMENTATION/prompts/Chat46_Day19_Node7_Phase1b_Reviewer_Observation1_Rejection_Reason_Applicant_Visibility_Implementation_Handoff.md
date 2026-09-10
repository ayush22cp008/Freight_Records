# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 1 — Rejection Reason Applicant Visibility Implementation Handoff

### Status

**TARGETED IMPLEMENTATION — AUTHORIZED FOR THIS SMALL CHANGE ONLY**

This handoff addresses one specific manual-verification issue:

> When a Reviewer rejects an applicant with a rejection reason, the rejected applicant must be able to see that actual rejection reason on the applicant-facing verification/rejection status surface.

Do not combine this task with the other Reviewer issues currently under manual review.

---

## 1. Current Intended Behavior

Current rejection flow:

```text
Reviewer
  ↓
Reject Applicant
  ↓
Enter required rejection reason
  ↓
Confirm Rejection
  ↓
Rejected outcome persisted
  ↓
Reviewer / History can see rejection reason
```

Required applicant-facing behavior:

```text
Rejected Applicant
  ↓
Open verification / onboarding status
  ↓
Application Rejected
  ↓
Actual persisted rejection reason is visible
```

Example:

```text
Application Rejected

Reason:
The submitted document is not valid for the claimed role.
```

The applicant should see the rejection reason associated with **their own application only**.

---

## 2. Governing Records

Read before editing:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md

02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md

03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Frontend_Implementation_Report.md

05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Investigation_Report.md
```

Relevant existing decision behavior and data model remain authoritative.

The already-persisted rejection reason is the source of truth. Do not introduce a duplicate rejection-reason field.

---

## 3. Scope

This is a **small applicant-facing visibility change**.

The implementation should:

1. Preserve the existing Reviewer rejection flow.
2. Preserve the existing persisted `rejection_reason` value.
3. Make that value available to the rejected applicant on the existing applicant-facing rejection/status surface.
4. Clearly label it as the rejection reason.
5. Show it only for the applicant's own rejected application.
6. Preserve the existing generic rejection state when no reason exists, without inventing a reason.

Do not redesign the Reviewer Portal.
Do not redesign Verification History.
Do not redesign the rejection lifecycle.

---

## 4. Source-of-Truth / Data Flow

Use the existing rejection reason already produced by:

```text
POST /api/admin/review
```

Do not create a new write path.

Trace the current applicant-facing status data flow and use the smallest change necessary to expose the persisted `rejection_reason`.

Preferred hierarchy:

```text
Existing persisted rejection_reason
        ↓
Existing applicant-facing retrieval path
        ↓
Existing applicant-facing status page
        ↓
Render rejection reason
```

If the existing applicant-facing retrieval path currently omits `rejection_reason`, add only the minimum field/data selection required.

If the value is already available to the page, make only the necessary rendering change.

Do not create a new API when the existing data source can safely support the change.

---

## 5. Privacy / Authorization Boundary

The rejected applicant may see:

```text
Their own rejection reason
```

The applicant must **not** receive:

```text
Other applicants' records
Reviewer internal notes
Internal Reviewer-only metadata
Unrelated verification records
Administrative information
```

Preserve all existing authentication and authorization behavior.

Do not weaken RLS, server authorization, or applicant identity checks.

---

## 6. UI Requirement

On the existing applicant-facing rejected state, add a clear section such as:

```text
Application Rejected

Reason
[actual persisted rejection_reason]
```

The presentation should follow the existing Driver/Company/shared Freight visual language.

Do not introduce a new visual theme.

Do not make the message excessively large or distracting; the reason should simply be clear and readable.

If the stored value is missing/null, show the existing safe fallback rather than fabricating content.

---

## 7. What Must Not Change

Do not change:

```text
Reviewer approval behavior
Reviewer rejection semantics
Required rejection reason behavior
Verification History architecture
R-05 architecture
Authentication
Role model
RLS/security architecture
Evidence architecture
Trip/delivery systems
Driver marketplace
Company workflows
AI behavior
C-05
R-03
```

Do not modify the locked Reviewer Blueprint.

Do not modify the original Reviewer frontend implementation report.

---

## 8. Validation Requirements

After implementation, verify the exact flow:

### Test A — Rejected applicant sees reason

1. Reviewer rejects an applicant with a concrete reason.
2. Confirm the rejection succeeds.
3. Open the rejected applicant's applicant-facing status/onboarding page.
4. Refresh/re-enter the page.
5. Confirm the actual persisted rejection reason is displayed.

### Test B — Correct applicant only

Confirm that one applicant cannot see another applicant's rejection reason.

### Test C — Approval unaffected

Confirm that Verified applicants do not see a rejection-reason block.

### Test D — Missing reason fallback

Where a rejected record has no stored reason, confirm the UI uses a safe existing fallback and does not invent a value.

### Test E — Existing Reviewer behavior preserved

Confirm Reviewer rejection still requires a reason and the existing Reviewer/History views continue to show the correct reason.

---

## 9. Build / Regression

Run the normal project build and relevant tests/checks.

Record:

```text
Build result
Tests/checks
Applicant-facing result
Reviewer regression result
Authorization/privacy result
```

Use evidence classifications accurately:

```text
VERIFIED
INFERRED
UNKNOWN
```

---

## 10. Required Implementation Report

Create a new report:

```text
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Implementation_Report.md
```

The report must contain:

- exact implementation scope;
- files changed;
- source/data flow used;
- how `rejection_reason` reaches the applicant-facing UI;
- applicant-facing verification evidence;
- privacy/authorization verification;
- regression/build/test evidence;
- any deviation or limitation;
- final verification status.

Do not overwrite the investigation report.

---

## 11. Stop Conditions

Stop and report instead of expanding scope if the change unexpectedly requires:

- a new authentication model;
- a role-model change;
- broad RLS/security changes;
- unrelated schema changes;
- unrelated API architecture;
- business-rule changes;
- changes to Reviewer decision semantics;
- changes to Driver/Company workflow behavior;
- any protected project-area modification.

This handoff authorizes only the narrow applicant-facing rejection-reason visibility change.

---

## 12. Final Execution Instruction

Implement **only** the applicant-facing rejection-reason visibility change.

Preserve all existing Reviewer behavior and project boundaries.

After implementation, build/test and record concrete evidence.

Then **STOP and wait for Ayush manual verification.**

Do not declare Reviewer Accepted or Locked.
