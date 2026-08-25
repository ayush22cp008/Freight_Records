# Chat12 Node 2 — Q6 Investigation: RLS / Service-Role Boundary

## Task

Investigate Q6: **What should the exact RLS vs server/service-role boundary be for Freight?**

Do not implement anything.
Do not reopen Q1–Q5 unless new evidence creates a genuine conflict.

## Read First

Read the authoritative project-control and Node 2 records, especially:

- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/PROJECT_STATE.md`
- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md`
- `02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`
- `03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q5_Authentication_Rate_Limiting.md`
- relevant existing RLS/service-role investigation records in `03_IMPLEMENTATION/implementation_reports/`

## Investigate

Determine, with evidence:

1. Which Freight tables/operations should use normal authenticated access + RLS?
2. Which operations genuinely require a server-side/service-role path?
3. How should RLS identify the authenticated user safely?
4. How should RLS interact with the locked Node 1 authorization model?
5. How should pending/unverified/verified/trusted-role states affect database access?
6. Which verification/identity operations require elevated server authority?
7. How do we prevent `service_role` from reaching browser/client code?
8. How do we prevent a service-role endpoint from bypassing Node 1 authorization?
9. Should server-side code prefer user-scoped authenticated clients where possible instead of service-role access?
10. What happens if an RLS policy is missing, too broad, or incorrectly written?
11. What cross-user / IDOR attacks must the RLS boundary prevent?
12. What acceptance tests prove the boundary is actually enforced?

## Important Distinction

Keep these separate:

```text
RLS
    = database row-access boundary

Authentication
    = who is the user?

Node 1 authorization
    = is this user allowed to perform this business operation?

service_role
    = privileged server-side database capability that bypasses RLS
```

Do not claim that service-role access is authorization by itself.

## Existing Evidence to Reconcile

The repository contains earlier RLS experiments/reports. Treat them as evidence, not as automatically locked decisions. Reconcile them with the current Node 1 and Node 2 architecture.

Relevant earlier records include:

- `03_IMPLEMENTATION/prompts/Chat8_Node3_Instruction_Experiment_RLS_Ownership_Policy.md`
- `03_IMPLEMENTATION/implementation_reports/Chat8_Node3_Report_Experiment_RLS_Ownership_Policy.md`
- `03_IMPLEMENTATION/prompts/Chat8_Node3_Instruction_Verify_Real_Authenticated_RLS_Path.md`
- `03_IMPLEMENTATION/implementation_reports/Chat8_Node3_Report_Verify_Real_Authenticated_RLS_Path.md`
- `03_IMPLEMENTATION/prompts/Chat11_Node2_Investigation_Round3_DriverID_RateLimit_RLS_Role_Evidence.md`

## Required Output

Produce a factual investigation report containing:

1. **Executive conclusion**
2. **Current architecture/evidence**
3. **Threat model**
4. **RLS responsibility**
5. **Service-role responsibility**
6. **Authenticated-user vs service-role access matrix**
7. **Node 1 authorization interaction**
8. **Identity/verification interaction**
9. **IDOR/cross-user attack analysis**
10. **Recommended MVP policy**
11. **Rejected alternatives and why**
12. **Acceptance-test matrix**
13. **Remaining unknowns**
14. **Proposed Q6 decision**

## Decision Discipline

Do not jump from an observed bug directly to implementation.

Use:

```text
Evidence
  ↓
Threat / architecture analysis
  ↓
Candidate policies
  ↓
Recommended policy
  ↓
Acceptance criteria
```

Q6 remains **OPEN / NOT LOCKED** until the investigation is reviewed, independently checked where appropriate, and explicitly approved by Ayush.

## Final Status Requirement

End the report with:

```text
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🔒 LOCKED
Q6 = 🟡 INVESTIGATED / NOT LOCKED
Implementation = ⏸️ PAUSED
```

**No code changes or database-policy changes are authorized by this investigation.**