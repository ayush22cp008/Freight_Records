# Chat46 — Day 19 — Node 7 — Phase 1b
# Reviewer + Driver + Company Database / Existing-System Truth Audit Investigation Handoff

**Status:** INVESTIGATION ONLY — NO IMPLEMENTATION AUTHORIZED  
**Investigator:** Antigravity  
**Architecture / evidence-review owner:** ChatGPT  
**Final authority:** Ayush  

---

## 1. Investigation Purpose

This investigation is the authoritative **truth-audit** follow-up to the recent Reviewer recovery/history work.

The project needs one honest answer about what actually exists and behaves correctly across:

```text
Driver Applicant
Company Applicant
Reviewer Queue
Reviewer Verification
Reviewer Decision
Reviewer History
Rejected Applicant Recovery
Evidence Storage / Database
RLS / Authorization
```

The previous whole-system report made conclusions that were partly based on assumptions. This handoff replaces assumption-based verification with an evidence-first audit.

The central rule is:

> **Do not declare a system behavior VERIFIED unless the report contains direct evidence sufficient to prove it.**

Source-code correctness, database schema existence, live database state, and observed runtime behavior are separate evidence classes and must never be silently combined.

---

## 2. Mandatory Evidence Classification

Every material finding MUST be classified as exactly one of:

```text
VERIFIED
INFERRED
UNKNOWN
BLOCKED
```

### VERIFIED
Direct evidence exists.

Examples:

- exact source code demonstrates a write/query;
- exact migration demonstrates a table/policy/constraint;
- direct production/database inspection demonstrates a row/value/policy;
- controlled runtime test demonstrates observed behavior.

### INFERRED
Strong conclusion from available evidence, but not directly demonstrated.

### UNKNOWN
The available evidence is insufficient to establish the fact.

### BLOCKED
The fact could be established only through an unavailable access/action/test.

**UNKNOWN and BLOCKED must never be converted into VERIFIED by assumption.**

---

## 3. Investigation Boundary

This is a truth audit, not an implementation task.

### Allowed

- inspect repository source;
- inspect migrations;
- inspect schema metadata;
- inspect RLS policies;
- inspect API routes;
- inspect frontend routes/components;
- inspect storage configuration/policies;
- perform safe read-only database inspection;
- perform controlled non-destructive runtime verification;
- create temporary test identities only when an established safe test procedure exists;
- compare production runtime behavior with repository expectations;
- produce an evidence matrix and root-cause classification.

### Forbidden

Do NOT:

- modify application code;
- modify migrations;
- change production schema;
- change RLS policies;
- change storage policies;
- mutate production records merely to make a test pass;
- fix the Queue;
- fix onboarding;
- fix recovery;
- fix Reviewer History;
- change Driver behavior;
- change Company behavior;
- change Reviewer behavior;
- silently clean up test records;
- declare acceptance or lock status.

If an investigation requires an implementation change, report it as a BLOCKED finding and stop that test.

---

## 4. Evidence Hierarchy

For each claim, prefer evidence in this order:

```text
1. Direct live runtime/database evidence
2. Controlled runtime test result
3. Exact repository source/migration evidence
4. Historical project record
5. Inference
```

Historical project records are context, not proof of the current live state.

A report must explicitly identify the evidence source used for every conclusion.

---

## 5. Governing Project Records — Read Before Investigation

Inspect:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
```

Then inspect the locked/current architecture relevant to this audit:

```text
02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md
02_ARCHITECTURE/locked_blueprints/Company_Integrated_Upgraded_Blueprint.md
02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md
```

Relevant prior Reviewer records:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Whole_Existing_System_Investigation_Report.md
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_History_Integrity_Investigation_Report.md
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Whole_Reviewer_Driver_Company_Onboarding_Verification_Recovery_Investigation_Report.md
```

Relevant governance:

```text
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Governance_Decision.md
```

Relevant implementation records:

```text
03_IMPLEMENTATION/prompts/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Implementation_Handoff.md
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Implementation_Report.md
```

Also inspect any newer records added after these paths before finalizing conclusions.

---

## 6. Source Code Truth Audit

Inspect the exact current code in the implementation repository.

At minimum inspect:

```text
src/app/api/onboarding/submit/route.ts
src/app/api/admin/review/route.ts
src/app/api/admin/history/route.ts
src/app/(authenticated)/onboarding/page.tsx
src/app/(authenticated)/onboarding/OnboardingForm.tsx
src/app/(authenticated)/reviewer/queue/page.tsx
src/app/(authenticated)/reviewer/verify/[id]/page.tsx
src/app/(authenticated)/reviewer/verify/[id]/ApplicantVerificationClient.tsx
src/app/(authenticated)/reviewer/history/page.tsx
src/app/(authenticated)/reviewer/history/[id]/page.tsx
src/app/(authenticated)/reviewer/history/[id]/EvidenceViewerClient.tsx
```

Search for all references to:

```text
freight_identities
onboarding_evidence
reviewer_decisions
reviewer_authorizations
requested_role
trusted_role
verification_status
DRIVER
COMPANY
DRIVING_LICENCE
LICENSE
GST
PENDING
REJECTED
VERIFIED
```

Record exact query/update semantics, especially:

```text
.single()
.maybeSingle()
.find()
.order()
.limit()
.range()
```

Do not summarize these calls loosely. Record what they actually constrain.

---

## 7. Database Schema Truth Audit

Inspect repository migrations and, where access exists, the actual production schema.

At minimum reconcile:

```text
freight_identities
onboarding_evidence
reviewer_authorizations
reviewer_decisions
drivers
companies
```

For every table record:

```text
Exists in repository?       VERIFIED / UNKNOWN
Exists in production?       VERIFIED / UNKNOWN / BLOCKED
Columns                     exact list
Primary key                 exact definition
Foreign keys                exact definition
Unique constraints          exact definition
Indexes                     relevant indexes
Policies                    exact policies
Current-state or history   classification
```

### Critical production/repository reconciliation

Explicitly determine:

```text
Repository migration 011 exists?                  YES / NO
Production reviewer_decisions exists?            YES / NO / UNKNOWN
Production schema matches migration?              YES / NO / UNKNOWN
Application source references table?              YES / NO
Application source writes table?                  YES / NO
Application source reads table?                   YES / NO
```

Do NOT write:

```text
"No drift"
```

unless production schema is directly inspected and compared.

---

## 8. Live Data Truth Audit

For the actual test applicants used during the current investigation, inspect live data without mutating it.

For each Driver and Company test identity capture:

```text
auth.users.id
freight_identities.id
freight_identities.auth_id
freight_identities.requested_role
freight_identities.trusted_role
freight_identities.verification_status
freight_identities.reviewed_at
freight_identities.created_at
```

Then enumerate ALL matching `onboarding_evidence` rows:

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

Then enumerate ALL matching `reviewer_decisions` rows:

```text
id
identity_id
auth_id
evidence_id
decision
rejection_reason
reviewed_at
```

For each evidence reference in `reviewer_decisions`, verify that the referenced evidence row actually exists.

Do not infer row existence from UI.

---

## 9. Driver Truth Audit

Run a fresh Driver applicant scenario where safe testing is available.

Record evidence for:

```text
Signup
→ requested_role = DRIVER
→ identity = PENDING
→ Driving Licence evidence inserted
→ document_type exact value
→ Queue visibility
→ Reviewer evidence display
→ Reject or Approve
→ decision record
→ applicant-facing state
```

For rejection/recovery, record:

```text
Review #1
→ REJECTED
→ rejection reason
→ history record

Recovery
→ new evidence
→ identity PENDING
→ Queue re-entry

Review #2
→ VERIFIED or REJECTED
→ second history record
```

For every transition capture actual database state before and after the action.

---

## 10. Company Truth Audit

Repeat the same complete scenario independently for Company.

The Company evidence must be tested independently; do not conclude it works merely because Driver works.

Record:

```text
requested_role = COMPANY
GST evidence type
Queue representation
Reviewer evidence display
Reject
Recovery
PENDING re-entry
History record #1
New decision #2
Company portal access after verification
```

Do not modify locked Company business logic.

---

## 11. Reviewer Queue Truth Audit

Determine exactly how Queue constructs an applicant row.

The audit MUST answer:

```text
What determines applicant eligibility?
What determines evidence existence?
What determines document label?
What happens with zero evidence rows?
What happens with one evidence row?
What happens with two+ evidence rows?
How is the authoritative current evidence selected?
```

Explicitly test the known mismatch risk:

```text
Stored Driver document_type = DRIVING_LICENCE
UI label condition = LICENSE
```

Determine whether the earlier “Driver + GST Document” observation is caused by:

```text
wrong database value
wrong UI mapping
wrong join
wrong identity/evidence association
test-data contamination
unknown
```

A conclusion must name the actual evidence supporting it.

---

## 12. Reviewer Verify Truth Audit

Inspect both server and client behavior.

The audit MUST establish:

```text
Does the identity lookup enforce PENDING?
How is evidence selected?
Does evidence selection require exactly one row?
What happens with multiple PENDING evidence rows?
What evidence ID is used for the decision?
Which evidence rows are changed by Reject?
Which evidence rows are changed by Approve?
```

Particular attention:

```text
currentEvidence = latest PENDING evidence
```

must be compared against the actual mutation query.

It is not sufficient to show that one evidence ID is selected; verify that the subsequent mutation targets that same record and not every pending record.

---

## 13. Reviewer Decision Truth Audit

For both APPROVE and REJECT determine whether the operation is:

```text
atomic
transactional
sequential
partially writable
```

Record every operation and whether its error is checked.

At minimum inspect:

```text
Evidence update
Identity update
History insert
Driver profile insert
Company profile insert
```

Determine possible inconsistent states such as:

```text
identity VERIFIED + no history record
identity VERIFIED + no Driver/Company record
history record exists + identity still PENDING
history record exists + evidence reference missing
identity REJECTED + evidence still PENDING
```

These are investigation targets only; do not create a fix.

---

## 14. Recovery Truth Audit

Inspect the complete current recovery flow.

The report MUST capture:

```text
1. Applicant authentication
2. Current status validation
3. Storage upload
4. Evidence INSERT
5. Identity status UPDATE
6. API response
7. Frontend response handling
8. router refresh/navigation
```

For a successful recovery determine actual final state.

For a failure determine whether the system can leave partial state.

The following states must be explicitly checked:

```text
new evidence + old REJECTED identity
new evidence + new PENDING identity
old evidence + new PENDING identity
multiple PENDING evidence rows
multiple historical evidence rows
```

---

## 15. Reviewer History Truth Audit

Verify the historical model from actual data.

The audit MUST demonstrate whether:

```text
Decision #1 = REJECTED
Decision #2 = VERIFIED/REJECTED
```

can both exist for the same applicant.

For each history record verify:

```text
decision id
identity id
review timestamp
decision value
rejection reason
linked evidence id
linked evidence existence
linked storage path
```

Also verify:

```text
newest-first ordering
pagination
selected-record lookup
read-only behavior
```

Do not call history “immutable” merely because no UI button edits it. Determine whether the database permissions/schema permit mutation.

---

## 16. Historical Evidence Integrity

This is a separate check from history-row existence.

For every completed Reviewer decision determine:

```text
Does evidence_id point to the exact evidence reviewed?
Does that evidence row still exist?
Does its storage object still exist?
Does the evidence document_type match the applicant role?
Does the storage_path resolve to the evidence shown?
```

A history record with `evidence_id` is not sufficient proof of historical evidence integrity unless the referenced evidence remains resolvable.

---

## 17. RLS / Authorization Truth Audit

For each relevant table inspect actual policies.

At minimum determine:

```text
Applicant SELECT
Applicant INSERT
Applicant UPDATE
Applicant DELETE
Reviewer SELECT
Reviewer INSERT
Reviewer UPDATE
Reviewer DELETE
```

Then map service-role usage.

For every privileged API path answer:

```text
Was the user authenticated first?
Was reviewer authorization checked where required?
Is the target identity derived from the authenticated user or arbitrary client input?
Can an applicant alter another applicant's identity?
Can an applicant access another applicant's evidence?
Can an applicant access Reviewer History?
```

Do not assume service-role usage is safe merely because authentication occurs somewhere in the route.

---

## 18. Storage Truth Audit

Verify:

```text
bucket name
public/private status
upload path format
viewer path format
applicant access
Reviewer access
```

For historical evidence, verify that the storage object associated with an old decision remains available to the authorized Reviewer.

---

## 19. Current Runtime vs Repository Comparison

Create an explicit matrix:

| Subject | Repository expectation | Live database/runtime | Match? | Evidence status |
|---|---|---|---|---|
| Driver role mapping | | | | |
| Company role mapping | | | | |
| Driver document type | | | | |
| Company document type | | | | |
| Queue eligibility | | | | |
| Queue evidence selection | | | | |
| Reviewer evidence selection | | | | |
| Reviewer decision persistence | | | | |
| Recovery status transition | | | | |
| Historical decision persistence | | | | |
| Historical evidence linkage | | | | |
| RLS | | | | |
| Storage | | | | |

Every row must have an evidence classification.

---

## 20. Known Suspicious Areas That MUST Be Checked

The investigation MUST explicitly check these items rather than assuming they are already resolved:

### A. `DRIVING_LICENCE` vs `LICENSE`

Determine exact source and live values.

### B. Multiple evidence rows

Recovery intentionally preserves old evidence and inserts new evidence. Determine whether Reviewer pages tolerate that cardinality.

### C. `.single()` queries

Identify every `.single()` against `onboarding_evidence` and determine whether multiple rows can legally exist.

### D. Queue `.find()`

Determine whether it can select an unintended evidence row.

### E. Reviewer decision mutation scope

Determine whether a decision updates one evidence row or all PENDING evidence rows.

### F. Service-role identity update

Verify authorization boundary and actual target scoping.

### G. `reviewer_decisions` schema

Verify production existence and schema parity.

### H. Historical evidence

Verify that the evidence linked to an old decision remains resolvable.

### I. Concurrency

Do not assume Last-Write-Wins is safe. Determine actual state/history consistency under repeated or simultaneous decisions where safe testing is possible.

### J. Applicant feedback

Verify that successful recovery produces a clear user-visible success state and failed recovery does not falsely look successful.

---

## 21. Required Failure Injection / Negative Tests

Where safe, perform non-destructive negative tests.

At minimum:

```text
Unauthenticated Reviewer API
Non-reviewer Reviewer API
Reviewer decision for non-PENDING identity
Missing evidence
Multiple evidence rows
Repeated recovery
Malformed document_type
Cross-user evidence access attempt
Cross-user history access attempt
Invalid identity_id
```

Record:

```text
Expected behavior
Actual behavior
HTTP/result
Database effect
Security classification
```

Do not mutate application code to make negative tests pass.

---

## 22. Required Truth Matrix

Produce the following final matrix with evidence references:

| Claim | Evidence source | Exact evidence | Classification | Consequence |
|---|---|---|---|---|
| Driver identity mapping | | | | |
| Company identity mapping | | | | |
| Driver evidence type | | | | |
| Company evidence type | | | | |
| Reviewer Queue | | | | |
| Reviewer Verify evidence | | | | |
| Approve | | | | |
| Reject | | | | |
| Recovery | | | | |
| History | | | | |
| Historical evidence | | | | |
| RLS | | | | |
| Storage | | | | |
| Cross-portal impact | | | | |

---

## 23. Required Defect Register

For each defect include:

```text
ID
Title
Affected role/surface
First observable symptom
Exact source location
Exact live evidence
Root cause
Contributing factor
Severity: BLOCKER / HIGH / MEDIUM / LOW
Classification: VERIFIED / INFERRED / UNKNOWN / BLOCKED
Does it violate locked blueprint? YES / NO
Does it require governance reopening? YES / NO / UNKNOWN
Implementation allowed now? NO
```

Do not group unrelated defects into one vague finding.

---

## 24. Required “What Is Actually Missing?” Section

This section is mandatory and must distinguish three different meanings of “missing.”

### Missing from repository

Objects/logic expected by current behavior but not present in source/migrations.

### Missing from production

Objects/columns/policies/data expected by source but absent from the live environment.

### Missing from runtime behavior

Objects exist, but the current runtime does not use or surface them correctly.

Use exact examples.

Do NOT write “database missing” without specifying which object and which evidence proves it.

---

## 25. Required “What Is Wrong?” Section

Separate defects into:

```text
CODE DEFECT
DATABASE/SCHEMA DEFECT
DATA DEFECT
RLS/AUTH DEFECT
STORAGE DEFECT
UI DEFECT
CONFIGURATION DEFECT
PRODUCTION/REPOSITORY DRIFT
UNKNOWN
```

A single symptom may have more than one contributing factor, but the root cause must be identified separately from symptoms.

---

## 26. Required Final Decision Boundary

The investigation report MUST end with exactly one of these overall states:

```text
A. VERIFIED HEALTHY — no material defect found
B. VERIFIED DEFECTS — implementation handoff required
C. PARTIALLY VERIFIED — additional investigation blocked/required
D. PRODUCTION DRIFT — repository and live system differ materially
E. BLOCKED — insufficient access/evidence to reach a reliable conclusion
```

Do not use:

```text
"fully verified"
"logically sound"
"no drift"
"safe"
```

unless the report contains direct evidence supporting those claims.

---

## 27. Required Final Investigation Report

Write the final report to:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Driver_Company_Database_System_Truth_Audit_Investigation_Report.md
```

The report MUST include:

1. Executive truth statement.
2. Investigation scope.
3. Evidence classes used.
4. Environment/runtime identity.
5. Repository commit tested.
6. Governing records inspected.
7. Driver audit.
8. Company audit.
9. Reviewer Queue audit.
10. Reviewer Verify audit.
11. Approve/Reject audit.
12. Recovery audit.
13. History audit.
14. Historical evidence audit.
15. Database schema audit.
16. Live data audit.
17. RLS/auth audit.
18. Storage audit.
19. Runtime-vs-repository comparison.
20. Suspicious-area results.
21. Negative-test results.
22. Truth matrix.
23. Defect register.
24. “What is actually missing?” section.
25. “What is wrong?” section.
26. Root-cause summary.
27. Cross-portal impact.
28. Overall decision boundary.
29. Exact next governance/implementation dependency, if any.

---

## 28. Critical Honesty Rule

If Antigravity cannot directly inspect production database state, the final report MUST say so.

For example:

```text
Production schema existence of reviewer_decisions = BLOCKED
```

is valid.

```text
Production schema exists and matches repository
```

is invalid unless directly demonstrated.

Likewise:

```text
Recovery appears correct from source = INFERRED
```

is valid.

```text
Recovery VERIFIED
```

requires runtime/database evidence.

---

## 29. No Fixes in This Investigation

If a defect is found, stop at:

```text
OBSERVATION
→ EVIDENCE
→ ROOT CAUSE
→ CLASSIFICATION
→ IMPACT
→ RECOMMENDED NEXT GOVERNANCE/IMPLEMENTATION DEPENDENCY
```

Do not continue to:

```text
FIX
```

The fix belongs to a separately authorized implementation handoff.

---

## 30. Success Criterion

This truth audit is successful only when Ayush can read the final report and answer, without guessing:

```text
What exists?
What does not exist?
What is only in repository?
What is only in production?
What actually runs?
What actually fails?
What data is actually stored?
What evidence is actually linked?
Which problems are code defects?
Which are database/schema defects?
Which are data/configuration defects?
Which claims remain UNKNOWN or BLOCKED?
What exact implementation work is required next?
```

**No implementation is authorized by this handoff.**
