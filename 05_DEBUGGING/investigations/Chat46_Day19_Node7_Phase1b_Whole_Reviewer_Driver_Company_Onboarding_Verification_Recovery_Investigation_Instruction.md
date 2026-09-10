# Chat46 — Day 19 — Node 7 — Phase 1b
# Whole Reviewer + Driver + Company Onboarding / Verification / Recovery Investigation Instruction

**Status:** INVESTIGATION ONLY — NO IMPLEMENTATION AUTHORIZED  
**Investigation owner:** Antigravity  
**Architecture / reasoning owner:** ChatGPT  
**Final authority:** Ayush  

---

## 1. Purpose

A sequence of Reviewer manual-verification observations has exposed behavior that may cross the boundaries between:

```text
Applicant identity
→ Driver / Company onboarding
→ Evidence submission
→ Reviewer Queue
→ Evidence Examination
→ Approve / Reject
→ Rejection reason
→ Applicant recovery
→ PENDING re-entry
→ Reviewer History
→ subsequent Reviewer decision
```

The observed recovery behavior has already demonstrated an RLS-induced partial-write condition and a mismatch between current application state and historical Reviewer decision requirements.

The purpose of this investigation is therefore to stop isolated fixes and reconstruct the **whole existing system behavior** across the Driver applicant path, Company applicant path, and Reviewer path before any additional implementation is attempted.

This investigation must establish one authoritative evidence-based model of:

1. Driver onboarding and verification.
2. Company onboarding and verification.
3. Reviewer Queue / Applicant Verification / decision behavior.
4. Rejection and rejection-reason propagation.
5. Rejected-applicant recovery and re-submission.
6. Evidence persistence and evidence identity across review cycles.
7. Reviewer History and historical-decision preservation.
8. Authorization / RLS / service-role boundaries.
9. Current-vs-historical state separation.
10. Cross-portal regressions or hidden coupling.
11. Whether current production schema and source code agree.

No implementation should proceed from assumptions produced by this investigation.

---

## 2. Governing Project Records — Read First

Before forming conclusions, inspect the current repository records and treat them as the project source of truth.

### Project control

- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/PROJECT_STATE.md`

### Identity / onboarding architecture

- `02_ARCHITECTURE/Chat15_Day8_Node2_Onboarding_Verification_Design_Decision.md`
- relevant Node 2 identity/authentication investigation and acceptance records
- relevant Node 6 security / RLS / authorization records

### Driver

- `02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md`
- `03_IMPLEMENTATION/implementation_reports/Chat42_Day16_Node7_Phase1b_Stage1_Driver_Implementation_Report.md`
- relevant accepted Driver post-implementation investigation / completion records
- relevant Driver completion / evidence investigations

### Company

- `02_ARCHITECTURE/locked_blueprints/Company_Integrated_Upgraded_Blueprint.md`
- `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
- `05_DEBUGGING/investigations/Chat45_Day18_Node7_Phase1b_Company_Integrated_Blueprint_Final_System_Audit_Report.md`
- relevant Company implementation / closure records

### Reviewer

- `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
- `05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Whole_Existing_System_Investigation_Report.md`
- `05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_History_Integrity_Investigation_Report.md`
- `00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Governance_Decision.md` if present
- `00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Readiness_Decision.md` if present
- `00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Governance_Decision.md`
- `03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Frontend_Implementation_Report.md`
- `03_IMPLEMENTATION/prompts/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Implementation_Handoff.md`

### Existing recent evidence

Inspect the current repository state for any newer Reviewer recovery implementation report, migration, runtime validation report, or decision record added after the records above.

Do not assume a historical record remains current if a newer record exists.

---

## 3. Investigation Boundary

This is a **source / schema / authorization / runtime investigation only**.

### Allowed

- Inspect source code.
- Inspect migrations and schema definitions.
- Inspect RLS policies.
- Inspect API routes.
- Inspect storage access paths.
- Inspect current Reviewer / Driver / Company routes and components.
- Reconstruct state and data flow.
- Run safe read-only queries and controlled runtime tests.
- Create temporary test identities/data only where existing project-safe testing patterns permit it.
- Compare source expectations with actual runtime behavior.
- Produce evidence and classify findings as VERIFIED / INFERRED / UNKNOWN / BLOCKED.

### Not allowed

Do **not**:

- modify application source code;
- modify migrations;
- create or alter production schema;
- change RLS policies;
- change API behavior;
- change storage objects or policies;
- clean or delete production test data unless an already-approved project-safe cleanup mechanism explicitly permits it;
- change Driver or Company product behavior;
- change Reviewer behavior;
- create permanent new schema as part of investigation;
- implement a fix because a defect is discovered.

If a runtime test requires a change that cannot be performed safely without implementation, STOP and report the dependency.

---

## 4. Primary Investigation Question

Establish whether the following complete lifecycle is actually coherent in the current system for **both Driver and Company applicants**:

```text
Signup
  ↓
requested_role
  ↓
Role-specific onboarding evidence
  ↓
PENDING identity
  ↓
Reviewer Queue
  ↓
Applicant Verification
  ↓
Evidence Examination
  ↓
Identity / Role Verified (frontend action)
  ↓
Approve OR Reject
  ├───────────────┐
  ↓               ↓
VERIFIED        REJECTED
  ↓               ↓
Role access      Rejection reason
  ↓               ↓
Driver/Company   Applicant recovery
portal           ↓
                 Re-upload evidence
                 ↓
                 PENDING
                 ↓
                 Reviewer Queue again
                 ↓
                 New decision
```

The investigation must establish where this flow is true, where it is broken, and exactly why.

---

## 5. Driver Applicant Investigation

Reconstruct the complete Driver path from signup through repeated verification cycles.

### 5.1 Identity and role

Determine:

- How a Driver applicant is created.
- Where `requested_role = DRIVER` is stored.
- How `trusted_role` is established.
- How `verification_status` is initialized.
- Which account/profile rows exist before approval.
- Which role is actually used by authenticated routing.
- Whether any UI field can alter effective role.

### 5.2 Driver evidence

Determine:

- Exact Driver onboarding route.
- Exact Driver evidence form.
- Expected evidence type.
- Actual `document_type` values stored.
- Actual `mime_type` values stored.
- Storage bucket/object path.
- Evidence database row relationship to `auth_id`.
- Number of evidence rows permitted per applicant.
- Whether there is a unique constraint.
- Whether a recovery submission intentionally replaces or appends evidence.
- Whether the UI and Reviewer label the same evidence consistently.

**Critical check:** determine whether a Driver applicant can ever legitimately have a stored/displayed `GST Document` or whether such a label indicates test-data contamination, source mapping error, stale data, or another problem.

### 5.3 Driver approval

Trace:

```text
PENDING
→ Reviewer Approve
→ verification_status
→ trusted_role
→ drivers record
→ Driver routing
→ Driver portal
```

Verify exact writes, ordering, failure behavior, and whether partial success can occur.

### 5.4 Driver rejection

Trace:

```text
Reviewer Reject
→ rejection reason
→ identity state
→ evidence state
→ Reviewer History
→ Driver-facing rejection page
```

Determine where the rejection reason is stored and whether the Driver applicant can see their own reason through an authorized path.

### 5.5 Driver recovery

Trace the full cycle:

```text
REJECTED
→ Re-upload Evidence
→ POST /api/onboarding/submit
→ current evidence mutation
→ identity mutation
→ final state
→ Reviewer Queue
```

Determine whether the flow succeeds, fails, partially succeeds, duplicates evidence, or causes any hidden state change.

---

## 6. Company Applicant Investigation

Repeat the Driver investigation independently for Company applicants.

### 6.1 Identity and role

Determine:

- How a Company applicant is created.
- Where `requested_role = COMPANY` is stored.
- How `trusted_role` is established.
- How `verification_status` is initialized.
- Which Company profile rows exist before approval.
- How authenticated Company routing is derived.

### 6.2 Company evidence

Determine:

- Exact Company onboarding route.
- Expected GST evidence type.
- Actual `document_type` values.
- Actual `mime_type` values.
- Storage path and bucket.
- Evidence row relationship to `auth_id`.
- Evidence cardinality.
- Replacement vs append semantics.
- UI/Reviewer label consistency.

### 6.3 Company approval

Trace:

```text
PENDING
→ Reviewer Approve
→ verification_status
→ trusted_role
→ companies record
→ Company routing
→ Company portal
```

Verify exact behavior and failure boundaries.

### 6.4 Company rejection

Trace rejection reason and completed Reviewer evidence all the way to the Company applicant-facing surface.

### 6.5 Company recovery

Trace:

```text
REJECTED
→ Re-upload Evidence
→ submit
→ current state
→ Queue re-entry
→ Reviewer decision
```

The Company recovery path must be investigated independently; do not assume Driver and Company behave identically merely because they share the same onboarding endpoint.

---

## 7. Reviewer Investigation

Reconstruct the complete Reviewer path against both applicant roles.

### 7.1 Queue

Determine:

- Exact Queue query.
- Exact eligibility filter.
- Whether Queue uses `freight_identities` or another source.
- Whether Queue joins evidence.
- How evidence is selected when multiple rows exist.
- Whether Queue behavior differs for Driver and Company.

### 7.2 Applicant Verification

Determine:

- Exact identity query.
- Exact evidence query.
- Whether `.single()`, `.maybeSingle()`, list, or another cardinality assumption is used.
- Behavior when zero evidence rows exist.
- Behavior when multiple evidence rows exist.
- Behavior when evidence is stale or belongs to a prior review cycle.

### 7.3 Evidence viewer

Determine:

- Exact storage bucket.
- Exact signed URL mechanism.
- TTL.
- Server vs client generation.
- Authorization boundary.
- Whether current and historical evidence are distinguishable.
- Whether the selected evidence is deterministic.

### 7.4 Identity / Role Verified

Verify that this remains only a frontend human action and not a persistent lifecycle state.

Determine whether any client-side state is accidentally reused as authoritative decision state.

### 7.5 Approve / Reject

Trace the final decision endpoint completely:

```text
frontend
→ API
→ authorization
→ identity update
→ evidence update
→ role-profile creation/activation
→ history persistence
→ response
```

Determine whether these operations are atomic, sequential, independently committed, or otherwise partially writable.

---

## 8. Persistent Data Model Investigation

Construct an evidence-based map of all relevant tables and relationships.

At minimum investigate:

```text
auth.users
freight_identities
onboarding_evidence
reviewer_authorizations
reviewer_decisions       (if present)
companies
drivers
```

Also identify any other table introduced by the current Reviewer recovery/history work.

For each table determine:

- Primary key.
- Relevant foreign keys.
- Unique constraints.
- Status fields.
- Timestamps.
- Evidence references.
- Rejection-reason fields.
- Whether the table represents current state or historical state.
- Whether the table is actually referenced by current source code.

### Critical schema question

Reconcile the repository's expected history model against the **actual current database schema**.

Explicitly determine:

```text
Does `reviewer_decisions` exist?
Is there a migration for it?
Does application code write to it?
Does application code read from it?
Is it only manually created in production?
Does the schema differ between repository and production?
```

Do not assume existence in one environment means the application uses it.

---

## 9. Current State vs Historical Decision Model

Produce a precise model separating:

```text
CURRENT APPLICATION
-------------------
verification_status
current evidence
current requested role
trusted role
current profile

COMPLETED REVIEW DECISION
-------------------------
decision status
rejection reason
review timestamp
evidence reference for that decision
applicant identity reference
```

Determine whether the current implementation actually maintains this separation.

Test the following conceptual lifecycle:

```text
Review #1
PENDING
→ REJECTED
→ history record #1

Recovery
REJECTED
→ new evidence
→ PENDING

Review #2
PENDING
→ VERIFIED or REJECTED
→ history record #2
```

The investigation must establish whether both history records survive and remain independently readable.

---

## 10. Recovery / Evidence Cardinality Investigation

This is a critical section.

Determine exactly what happens when a rejected applicant submits new evidence.

At every step inspect:

```text
Before submission:
  identity row(s)
  evidence row(s)
  history row(s)

Submission:
  DELETE?
  INSERT?
  UPDATE?
  RPC?
  API response?

After submission:
  identity row(s)
  evidence row(s)
  history row(s)
```

Explicitly test and document:

### Case 1 — One previous evidence row

```text
old evidence
→ recovery
→ expected new evidence
```

### Case 2 — Multiple evidence rows already exist

Determine whether the application:

- picks one deterministically;
- errors;
- silently picks an arbitrary row;
- exposes wrong reason/document;
- corrupts Reviewer History presentation.

### Case 3 — Recovery fails halfway

Determine whether any of the following can occur:

```text
new evidence exists + identity REJECTED
old evidence deleted + identity REJECTED
identity PENDING + no evidence
identity PENDING + wrong evidence
history record mutated/deleted
```

### Case 4 — Repeated recovery

Determine whether:

```text
REJECTED
→ recovery
→ PENDING
→ recovery again before review
```

is possible and what evidence/state result it produces.

Do not modify behavior during this investigation.

---

## 11. RLS / Authorization Investigation

Map every relevant policy for:

```text
freight_identities
onboarding_evidence
reviewer_decisions (if present)
companies
drivers
```

For each operation determine whether it is allowed for:

```text
Applicant
Reviewer
Sender Company
Receiving Company
Driver
Unauthenticated user
```

Explicitly inspect:

- SELECT
- INSERT
- UPDATE
- DELETE

Also determine where service-role access is used.

For each service-role path answer:

1. Is the user authenticated first?
2. Is the user role/authorization checked first?
3. Is the target identity derived from the authenticated session?
4. Can client-supplied IDs alter the authorization scope?
5. Is the service role limited to the minimum required operation?

Do not recommend broad RLS changes until the exact current failure is demonstrated.

---

## 12. Error / Failure Semantics

Trace how failures propagate from:

```text
Supabase/RLS/database error
→ API response
→ frontend state
→ user-visible message
```

Determine whether any current path can produce:

```text
HTTP failure
but data partially changed

OR

HTTP success
but required state not actually committed

OR

UI success
without server confirmation
```

Test the recovery form specifically for clear success and failure feedback.

---

## 13. Applicant-Facing State Investigation

For both Driver and Company, determine the exact UI for:

```text
PENDING
VERIFIED
REJECTED
```

For REJECTED, determine:

- rejection reason visibility;
- recovery action;
- access to onboarding;
- current evidence visibility;
- success after re-upload;
- state after refresh;
- state after sign-out/sign-in;
- state after direct navigation.

Do not conflate Reviewer-facing history with applicant-facing state.

---

## 14. Reviewer History Investigation

Reconstruct the entire history behavior.

Determine:

- Source table(s).
- Exact query.
- Current filtering logic.
- Ordering field.
- Pagination.
- Selected-record lookup.
- Evidence lookup.
- Rejection reason lookup.
- Decision timestamp lookup.
- Whether multiple decisions for one applicant are supported.
- Whether history is actually immutable.
- Whether history can survive current-state changes.

Test at least:

```text
Applicant A
Decision #1 = REJECTED
→ recovery
→ PENDING
→ Decision #2 = VERIFIED
```

Expected investigation question:

> Can Reviewer History display both Decision #1 and Decision #2 as separate completed decisions without one overwriting or hiding the other?

Do not assume the expected answer; determine it from source/runtime evidence.

---

## 15. Cross-Portal Impact Investigation

Determine whether Reviewer/onboarding/recovery behavior affects the already accepted Driver or Company portals.

### Driver impact

Check that verification/recovery changes do not alter:

- Driver authentication.
- Driver role authorization.
- Driver marketplace eligibility.
- Driver trip claim behavior.
- Driver active/completed-trip behavior.

### Company impact

Check that verification/recovery changes do not alter:

- Company authentication.
- Company role authorization.
- Company trip creation.
- Receiver request Accept/Reject.
- Publish gates.
- Driver Claim gate.
- Company history.

The Company integrated blueprint is already locked and its audit reports the Receiver Request, Publish, Claim, History, and lifecycle behaviors as verified. Treat those as regression-protected surfaces, not targets for opportunistic change.

### Reviewer impact

Check that the recovery work does not introduce:

- `UNDER_REVIEW` state;
- automatic verification;
- new Reviewer authority;
- operational trip/delivery review responsibilities.

---

## 16. Production Schema vs Repository Reconciliation

The project currently has evidence of direct production Supabase schema intervention and may contain structures not yet reflected in the repository.

Therefore explicitly reconcile:

```text
Repository migrations
        vs.
Production schema
        vs.
Current application source
```

For every relevant object record:

```text
Repository state = VERIFIED / MISSING / UNKNOWN
Production state = VERIFIED / MISSING / UNKNOWN
Source usage    = VERIFIED / MISSING / UNKNOWN
```

Any schema drift that materially affects Reviewer History, recovery, or evidence selection must be reported as a first-class finding.

---

## 17. Security / Privacy Checks

Verify that:

```text
Applicant A
→ cannot read Applicant B's evidence
→ cannot read Applicant B's Reviewer History
→ cannot trigger Applicant B's recovery
```

and:

```text
Reviewer
→ can read authorized completed records
→ cannot gain unrelated operational Company/Driver authority
```

Also verify that signed evidence URLs are only issued for authorized evidence records.

Do not treat browser-local state as a security boundary.

---

## 18. Concurrency / Repeated-Action Investigation

Without changing source code, investigate whether the current APIs safely handle:

```text
Two simultaneous Reviewer decisions
Repeated Reject clicks
Repeated recovery submissions
Recovery while Reviewer is viewing the old record
Two browser tabs for the same applicant
```

The goal is to establish whether the current system relies solely on UI disabling or has server/database protections.

Do not invent new concurrency requirements beyond documenting actual behavior and observed risk.

---

## 19. Required Before/After State Matrix

Produce an evidence-based table covering at least:

| Scenario | Identity state | Evidence rows | History records | Queue | Applicant UI | Reviewer UI | Status |
|---|---|---:|---:|---|---|---|---|
| Fresh Driver submitted | | | | | | | |
| Fresh Company submitted | | | | | | | |
| Driver approved | | | | | | | |
| Company approved | | | | | | | |
| Driver rejected | | | | | | | |
| Company rejected | | | | | | | |
| Driver recovery submitted | | | | | | | |
| Company recovery submitted | | | | | | | |
| Recovered Driver re-reviewed | | | | | | | |
| Recovered Company re-reviewed | | | | | | | |
| Recovery failure | | | | | | | |
| Duplicate evidence condition | | | | | | | |

Use `VERIFIED`, `INFERRED`, `UNKNOWN`, or `BLOCKED` per cell where appropriate.

---

## 20. Required Source-to-Behavior Trace Tables

Produce these trace tables.

### A. Driver

```text
Route
→ Component
→ API
→ Table
→ Field
→ RLS/authorization
→ UI result
```

### B. Company

Same structure.

### C. Reviewer

Same structure for:

```text
Queue
Review
Evidence
Approve
Reject
History
History detail
```

### D. Recovery

```text
Applicant action
→ frontend
→ API
→ evidence mutation
→ identity mutation
→ history mutation
→ response
→ Queue
```

---

## 21. Root-Cause Classification

Every material issue must be classified into one or more categories:

```text
UI-only
Frontend state
Routing
API contract
Server authorization
RLS
Database schema
Data-model mismatch
Evidence cardinality
Atomicity / partial write
History-model deficiency
Production/repository drift
Test-data contamination
Unknown
```

Do not classify a symptom as a root cause without tracing the underlying source behavior.

---

## 22. Investigation Output Requirements

Create the final investigation report at:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Whole_Reviewer_Driver_Company_Onboarding_Verification_Recovery_Investigation_Report.md
```

The report must contain:

1. Executive finding.
2. Exact observation(s).
3. Environment/runtime used.
4. Governing records inspected.
5. Driver end-to-end trace.
6. Company end-to-end trace.
7. Reviewer end-to-end trace.
8. Recovery trace.
9. Evidence cardinality analysis.
10. Current-state vs historical-state model.
11. Database/schema findings.
12. Repository vs production drift findings.
13. RLS/security findings.
14. Error/partial-write findings.
15. Applicant-facing findings.
16. Reviewer History findings.
17. Cross-portal impact findings.
18. Concurrency/repeated-action findings.
19. Before/after state matrix.
20. Root causes.
21. Contributing factors.
22. Ruled-out hypotheses.
23. VERIFIED / INFERRED / UNKNOWN / BLOCKED classification.
24. Exact boundaries crossed, if any.
25. Recommended next governance decision(s), without implementing them.

The report must not contain a proposed code patch disguised as an investigation result.

---

## 23. Required Final Conclusions

The final report must explicitly answer all of these questions:

### Identity

1. Is Driver identity mapping correct?
2. Is Company identity mapping correct?
3. Is requested role distinct from trusted role?

### Evidence

4. Does Driver onboarding produce the correct Driving Licence evidence?
5. Does Company onboarding produce the correct GST evidence?
6. Can one applicant have multiple evidence rows?
7. If yes, what is the authoritative row-selection rule?

### Reviewer

8. Does Reviewer Queue correctly represent PENDING applicants?
9. Does Applicant Verification select the correct evidence?
10. Does Approve persist correctly?
11. Does Reject persist correctly?
12. Is rejection reason preserved and visible to the correct party?

### Recovery

13. Can REJECTED safely become PENDING?
14. Does recovery preserve the prior decision?
15. Does recovery preserve the prior evidence reference?
16. Does recovered applicant re-enter Queue?
17. Does the applicant receive accurate success/failure feedback?

### History

18. Can one applicant have multiple completed decisions?
19. Are completed decisions immutable?
20. Can old evidence be resolved for an old completed decision?
21. Does History survive changes to current application state?

### Security

22. Are applicant writes owner-scoped?
23. Is Reviewer history protected?
24. Are service-role paths appropriately authorization-gated?
25. Can client-supplied IDs change authority?

### Cross-portal

26. Does any Reviewer/onboarding/recovery issue affect locked Driver behavior?
27. Does any Reviewer/onboarding/recovery issue affect locked Company behavior?
28. Is there an actual regression requiring governance reopening of another portal, or only a shared identity/evidence dependency?

### Architecture

29. What is the smallest true root cause?
30. Does the problem require a narrow correction or a broader architecture decision?
31. What must be explicitly approved before implementation?

---

## 24. Stop Conditions

Stop the investigation and escalate immediately if any of the following prevents a reliable conclusion:

- production schema cannot be inspected or reconciled;
- runtime behavior differs materially from repository behavior and the reason cannot be established;
- testing would require destructive production changes;
- a new lifecycle state appears necessary;
- a broad audit/event-sourcing model appears necessary;
- a broad evidence-versioning system appears necessary;
- a broad RLS rewrite appears necessary;
- a general transaction framework appears necessary;
- Driver or Company locked business behavior appears genuinely broken rather than merely sharing a Reviewer dependency;
- a protected Node 1–6 contract appears invalid;
- the exact current source cannot establish the authoritative data path.

Do not solve any stop condition during this investigation.

---

## 25. Important Interpretation Rules

### Rule 1 — Current state is not history

Do not describe a current `freight_identities` status as immutable historical evidence unless the system actually persists an independent historical record.

### Rule 2 — Current evidence is not automatically historical evidence

A later upload must not be assumed to represent evidence used in an earlier Reviewer decision.

### Rule 3 — UI labels are not database truth

A label such as `GST Document` or `Driving Licence` is not sufficient evidence of the underlying stored document type.

### Rule 4 — HTTP success is not sufficient

Verify resulting database state.

### Rule 5 — RLS failure is not automatically a bug in RLS

First determine whether the current operation is supposed to be user-authorized or deliberately server-authorized.

### Rule 6 — Locked portals stay protected

Do not use this investigation as permission to redesign Driver or Company. Only identify and classify actual regressions/dependencies.

### Rule 7 — No assumptions across roles

Driver and Company may share infrastructure but must be traced independently.

### Rule 8 — No implementation during investigation

Even an obvious fix remains an investigation finding until governance authorizes implementation.

---

## 26. Final Investigation Principle

The investigation is successful only when the project can answer, with evidence, this single question:

> **For a Driver applicant and for a Company applicant, can the system safely move from application → Reviewer decision → completed decision → rejection → recovery → new PENDING review → new decision, while preserving the correct evidence, decision history, authorization boundaries, and role-specific access at every step?**

If the answer is no, identify the exact boundary and root cause before proposing any fix.

**No implementation is authorized by this document.**
