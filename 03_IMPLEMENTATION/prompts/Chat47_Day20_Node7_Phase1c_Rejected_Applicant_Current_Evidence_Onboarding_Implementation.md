# Chat47 Day20 Node7 Phase1c — Rejected Applicant Current Evidence Onboarding Implementation Prompt

**Implementation status:** APPROVED FOR IMPLEMENTATION  
**Architecture / reasoning owner:** ChatGPT  
**Independent review:** Grok — Root Cause CONFIRMED / Proposed Fix APPROVE / Implementation Readiness READY  
**Final authority:** Ayush  
**Implementation executor:** Antigravity  
**Scope:** Narrow Phase1c applicant onboarding recovery display fix only

---

## 1. Implementation Authorization

Ayush has approved the Phase1c governance decision after independent Grok review.

Grok independently confirmed:

- **Root Cause:** CONFIRMED
- **Proposed Fix:** APPROVE
- **Implementation Readiness:** READY
- **Governance Scope:** CORRECTLY SCOPED

Do not expand this task beyond the exact scope below.

---

## 2. Governing Records — Read First

Before editing source code, read these Records files:

```text
00_PROJECT_CONTROL/DECISIONS/Chat47_Day20_Node7_Phase1c_Rejected_Applicant_Current_Evidence_Onboarding_Governance_Decision.md
01_BRAIN_HANDOFFS/Grok/Chat47_Day20_Node7_Phase1c_Rejected_Applicant_Current_Evidence_Onboarding_Grok_Independent_Review.md
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Driver_Company_Database_System_Truth_Audit_Investigation_Report.md
03_IMPLEMENTATION/implementation_reports/Chat47_Day20_Node7_Phase1b_Reviewer_Current_Evidence_Selection_Implementation_Report.md
02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md
02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md
02_ARCHITECTURE/locked_blueprints/Company_Integrated_Upgraded_Blueprint.md
```

Then inspect the current source implementation in the source repository.

**Source repository:**
```text
ayush22cp008/freight_hackathon
```

**Required source files:**
```text
src/app/(authenticated)/onboarding/page.tsx
src/app/(authenticated)/onboarding/OnboardingForm.tsx
src/app/api/onboarding/submit/route.ts
src/app/(authenticated)/reviewer/verify/[id]/page.tsx
```

The source repository must be treated as the implementation target. Records are the governance/architecture source of truth.

---

## 3. Verified Problem

A rejected applicant can successfully re-upload evidence.

The submit operation already behaves correctly:

```text
old evidence: REJECTED
new evidence: PENDING
identity.verification_status: PENDING
reviewer decision count: unchanged
```

The problem occurs when the recovered applicant returns to `/onboarding`.

The onboarding page currently performs an applicant-wide single-row evidence lookup equivalent to:

```ts
.eq('auth_id', identity.auth_id)
.single()
```

Recovery legitimately creates multiple evidence rows for the same applicant. Therefore the query can no longer return exactly one row.

The query failure leaves `evidence` unavailable. The page currently gates the Pending Verification UI on a truthy `evidence` object. As a result, a valid `PENDING` identity can fall through to the fallback UI and display:

```text
Complete Onboarding
```

This is a cardinality mismatch between the database history model and the page's one-row assumption.

`onboarding_evidence` must be treated as a multi-row evidence/history table, not a one-row-per-applicant table.

---

## 4. Required Fix

Modify only the authenticated applicant onboarding page so that it no longer depends on an applicant-wide `.single()` evidence query.

### Required current-evidence selection

Resolve the latest relevant `PENDING` evidence deterministically using the same safe pattern already accepted and implemented in Reviewer Verify:

```text
.eq('auth_id', identity.auth_id)
.eq('status', 'PENDING')
.order('created_at', { ascending: false })
.limit(1)
```

Use the resulting current pending evidence row only as supporting display data.

### Required state gating

Use the identity lifecycle state as the primary gate:

```text
identity.verification_status === 'PENDING'
```

When the authenticated identity is `PENDING`, render the existing **Pending Verification** state.

Do not require the evidence object to exist merely to decide whether the applicant is in Pending Verification.

The evidence record should provide any supporting fields the existing UI needs, but evidence lookup must not cause a valid PENDING applicant to fall through to the onboarding form.

### Error handling clarification

Distinguish these conditions:

1. Evidence query succeeded and no PENDING evidence exists.
2. Evidence query failed.

Do not silently convert a query failure into an onboarding-form fallback.

A safe Pending Verification state is preferable to incorrectly showing `Complete Onboarding` for a `PENDING` identity.

No new lifecycle state is permitted.

---

## 5. Historical Preservation Requirements

The fix must preserve the existing recovery model completely.

After re-upload:

```text
historical REJECTED evidence → preserved
new PENDING evidence        → preserved
original reviewer decision  → preserved
identity status             → PENDING
```

Do not delete, overwrite, collapse, or mutate historical evidence rows.

Do not create a new reviewer decision during applicant re-upload.

Do not change the submit route unless source inspection proves a strictly necessary correction inside the approved scope.

---

## 6. Role Compatibility

The fix must remain compatible with both existing onboarding roles:

```text
DRIVER  → DRIVING_LICENCE
COMPANY → GST
```

Do not introduce new evidence types, role rules, or lifecycle states.

Current-evidence selection is based on authenticated owner scope + `PENDING` status + newest timestamp. Do not add speculative document-type rules to the page unless the existing source contract demonstrably requires them.

---

## 7. Security Requirements

Preserve authenticated owner scoping.

The onboarding page must continue to query evidence using the currently authenticated identity's own `auth_id`.

Do not weaken RLS, authorization, ownership checks, or introduce service-role access into the applicant page.

Do not expose another applicant's evidence through any new query path.

No security model changes are authorized by this task.

---

## 8. Scope Boundary — DO NOT CHANGE

Do **not** modify or redesign:

```text
Reviewer Queue behavior
Reviewer History behavior
Reviewer Verify current-evidence selection (already fixed)
Database schema
onboarding_evidence table structure
RLS policies
Authentication
Rate limiting
Driver portal workflow
Company portal workflow
Trip lifecycle
Receiver handshake
AI verification/scoring
Evidence types
Lifecycle states
Broad UI/navigation architecture
```

Do not refactor unrelated code.

Do not create a broad audit/history subsystem.

Do not solve hypothetical future issues in this task.

---

## 9. Implementation Constraints

Keep the change narrow and minimal.

Preferred implementation characteristics:

- one focused source-page change
- reuse the already accepted Reviewer Verify current-PENDING selection pattern
- preserve existing UI for Pending Verification
- preserve existing submission behavior
- preserve existing data model
- no migration
- no new API contract unless absolutely required by the current source

Do not replace the existing lifecycle semantics with a new state machine.

Do not make the form submit again simply to display the Pending Verification state.

---

## 10. Required Validation After Implementation

Perform code/build validation in the source repository.

Then prepare the implementation report for Records.

Ayush will perform final manual verification against the live system.

### Manual acceptance scenario

Use a controlled applicant currently in the rejected/recovery flow:

```text
1. Applicant has historical REJECTED evidence.
2. Applicant re-uploads replacement evidence.
3. Submit request succeeds.
4. identity.verification_status = PENDING.
5. New evidence row = PENDING.
6. Old REJECTED evidence remains.
7. Original reviewer decision remains.
8. Reload /onboarding.
9. Page shows Pending Verification.
10. Page does NOT show Complete Onboarding.
11. Applicant does not need to submit a second time.
```

### Reviewer continuity check

Confirm that this change does not regress the already-fixed Reviewer Verify behavior:

```text
Reviewer Queue → applicant appears
Reviewer Verify → newest PENDING evidence loads
Historical REJECTED evidence remains preserved
No second reviewer rejection is created automatically
```

### Role coverage

Validate both:

```text
DRIVER
COMPANY
```

### Negative/error-path validation

Where practical, verify that an evidence-query failure cannot incorrectly send a `PENDING` identity back to the onboarding form.

---

## 11. Implementation Report Requirements

After implementation, create/update the corresponding implementation report in Records:

```text
03_IMPLEMENTATION/implementation_reports/Chat47_Day20_Node7_Phase1c_Rejected_Applicant_Current_Evidence_Onboarding_Implementation_Report.md
```

The report must contain:

```text
Implementation status
Files changed
What changed
Why it changed
Tests/build results
Security impact
Scope compliance
Manual verification required from Ayush
Commit hash / source revision
Any deviations or blockers
```

Clearly classify implementation observations as VERIFIED / INFERRED / UNKNOWN / BLOCKED where applicable.

Do not claim live verification unless it was actually performed.

---

## 12. Final Completion Gate

Implementation is not considered fully closed by Antigravity alone.

Completion sequence:

```text
Governance approved
    ↓
Independent review approved
    ↓
Antigravity implements narrow fix
    ↓
Build/test validation
    ↓
Implementation report written
    ↓
Ayush manually verifies live rejected-applicant recovery
    ↓
ChatGPT reconciles result and closes/advances the Node
```

Do not treat implementation completion as product acceptance until Ayush's live verification is successful.

---

## 13. Stop Conditions

Stop and report before making broader changes if any of the following is discovered:

- the current source implementation materially differs from the reviewed source and invalidates the approved fix;
- a schema change appears necessary;
- RLS or authorization changes appear necessary;
- the fix requires changes outside the approved Phase1c boundary;
- a second lifecycle state is required;
- historical evidence or reviewer decisions would need to be deleted or mutated;
- the existing Reviewer Verify fix would need to be rewritten;
- an unrelated regression is uncovered.

Do not silently expand scope.

---

## 14. Authorized Outcome

The authorized outcome is narrowly defined:

```text
Rejected applicant re-uploads evidence
        ↓
identity remains / becomes PENDING
        ↓
newest PENDING evidence is selected deterministically
        ↓
/onboarding recognizes PENDING from identity state
        ↓
existing Pending Verification UI renders
        ↓
old rejected evidence + original reviewer decision remain preserved
```

No new lifecycle semantics are introduced.

**Proceed with implementation only within this exact boundary.**
