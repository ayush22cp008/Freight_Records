# Chat42 — Day 16 — Node 7 — Phase 1b
# Implementation-Boundary Investigation Instructions

## 1. Purpose

Perform a targeted factual investigation to close the remaining evidence gaps required for ChatGPT to complete the architectural Implementation-Boundary Analysis for Node 7 Phase 1b.

This is an investigation task only. Do not implement, refactor, redesign, or modify source code.

The governing architectural constraints are the locked Driver, Company, and Reviewer blueprints and the locked Shared Cross-Portal Design System. The existing Chat41 Implementation-Boundary Review Report is evidence, but it is not itself the final architectural boundary decision.

## 2. Current Gate

- Node: 7
- Phase: 1b
- Implementation order: Driver → Company → Reviewer
- Current gate: Implementation-Boundary Review
- Implementation authorization: NOT GRANTED
- Immediate objective: collect only the factual evidence needed to determine what can safely change in Phase 1b frontend implementation and what must remain protected.

## 3. Investigation Questions

### A. Driver Boundary

Determine, with source-code evidence:

1. Which Driver routes/pages/components are currently present and actively used.
2. Which existing APIs, data reads, and mutation paths those Driver surfaces depend on.
3. Which existing Driver interactions are purely presentational and therefore safe candidates for Phase 1b redesign.
4. Which existing Driver behaviors are coupled to business rules, lifecycle/state transitions, authentication, authorization/RLS, evidence integrity, or other protected behavior.
5. Whether the locked Driver blueprint can be implemented as a frontend/UI layer without changing backend contracts or product behavior.
6. Any concrete dependency or ambiguity that could block implementation.

### B. Company Boundary

Determine, with source-code evidence:

1. Which Company routes/pages/components are currently present and actively used.
2. Which existing APIs, data reads, and mutation paths those Company surfaces depend on.
3. Which existing Company interactions are purely presentational and therefore safe candidates for Phase 1b redesign.
4. Which existing behaviors are coupled to trip creation, incoming delivery handling, public sharing, relationship/state logic, authentication, authorization/RLS, or other protected behavior.
5. Whether the locked Company blueprint can be implemented as a frontend/UI layer without changing backend contracts or product behavior.
6. Any concrete dependency or ambiguity that could block implementation.

### C. Reviewer Boundary

Determine, with source-code evidence:

1. Which Reviewer routes/pages/components are currently present and actively used.
2. Confirm the exact frontend dependency on the existing review API and evidence-access path.
3. Separate safe frontend/UI corrections from backend/security changes.
4. Identify which Reviewer defects can be corrected within Phase 1b without changing the verification model or security architecture.
5. Identify any frontend changes that would necessarily require backend/API/data/security changes and therefore remain outside the Phase 1b boundary.
6. Determine whether the locked Reviewer blueprint can be implemented without introducing new verification mechanisms, review states, evidence requirements, scoring, trip review, or general administration behavior.

### D. Cross-Portal Dependencies

Investigate only factual dependencies that affect implementation sequencing:

1. Shared components/design primitives used by Driver, Company, and Reviewer.
2. Shared layouts/navigation/authentication boundaries that could cause cross-portal regressions.
3. Existing status/state presentation patterns that are shared across portals.
4. Whether Driver implementation can be completed and tested without changing Company or Reviewer behavior.
5. Whether Company implementation can be completed and tested without changing Reviewer behavior.
6. Any shared dependency that requires coordinated implementation rather than isolated portal work.

### E. Protected Boundary Verification

Explicitly verify whether the following are touched by the proposed Phase 1b frontend work:

- APIs and API contracts
- Database/data models
- Business rules
- Authentication
- Authorization/RLS
- Evidence model or evidence requirements
- Marketplace/claiming behavior
- Security/evidence integrity
- Backend behavior
- AI behavior
- Existing lifecycle/state model

For each, classify the evidence as:

- VERIFIED — existing source evidence proves the dependency/boundary.
- INFERRED — supported by evidence but not directly proven.
- UNKNOWN — insufficient evidence; do not assume.

## 4. Evidence Requirements

For every material finding, provide:

- Exact source path.
- Relevant route/component/API/database object.
- Observed behavior or dependency.
- Why it matters to the Phase 1b implementation boundary.
- Confidence: VERIFIED / INFERRED / UNKNOWN.

Prefer direct source-code evidence, route inventories, API traces, imports/usages, and existing implementation records. Do not rely on screenshots alone when source evidence is available.

## 5. Explicit Non-Goals

Do NOT:

- Modify source code.
- Create implementation prompts.
- Create implementation plans.
- Fix defects.
- Change APIs or database policies.
- Change authentication or authorization.
- Change lifecycle/state models.
- Introduce new backend behavior.
- Introduce AI behavior.
- Invent missing functionality merely because it appears in a blueprint.
- Treat a proposed blueprint feature as proof that an existing backend capability exists.

If a blueprint requirement has no existing implementation evidence, record it as UNKNOWN or as a frontend-only proposal requiring architectural approval rather than assuming it exists.

## 6. Required Output

Produce an investigation report containing:

1. Executive evidence summary.
2. Driver implementation-boundary findings.
3. Company implementation-boundary findings.
4. Reviewer implementation-boundary findings.
5. Cross-portal dependency findings.
6. Protected-boundary verification matrix.
7. Evidence/confidence register.
8. Blocking UNKNOWNs and ambiguities.
9. Recommended factual conclusion for ChatGPT's architectural boundary decision.
10. Clear statement that this report does NOT grant implementation authorization.

## 7. Governing Records

Use these records as the governing context and evidence sources:

- `00_PROJECT_CONTROL/CHECKPOINTS/Chat41_Node7_Phase1b_Master_Handoff_Checkpoint.md`
- `00_PROJECT_CONTROL/Chat41_Node7_Phase1b_Shared_Design_System_Decisions.md`
- `02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md`
- `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
- `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
- `03_IMPLEMENTATION/implementation_reports/Chat41_Node7_Phase1b_Implementation_Boundary_Review_Report.md`
- `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Completion_Report.md`

## 8. Final Instruction

Investigate only what is necessary to close the Implementation-Boundary evidence gap.

The result must enable ChatGPT to make a separate architectural decision on:

- what Phase 1b may change,
- what Phase 1b must preserve,
- what requires separate investigation/approval,
- whether implementation preparation is safe to begin,
- and whether any UNKNOWN remains that prevents implementation readiness.

Implementation remains NOT AUTHORIZED until ChatGPT completes the boundary decision and Ayush explicitly authorizes implementation.
