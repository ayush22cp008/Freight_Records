# Chat42 Day16 Node7 Phase1b — Master Implementation Prompt

Status: EXECUTION GATE CLOSED — IMPLEMENTATION NOT AUTHORIZED

## 1. Execution Gate
Do not begin implementation, modify source code, create application files, or alter backend behavior until Ayush explicitly authorizes Phase 1b implementation.

When authorization is given, execute only the approved Phase 1b scope in this prompt and the finalized Implementation Preparation Master Scope.

## 2. Authority and Roles
- ChatGPT: reasoning, architecture, boundary decisions, implementation scope.
- Antigravity: implementation and execution only; do not independently expand scope or redesign architecture.
- Ayush: final authority and manual tester; authorization and acceptance are explicit.
- Freight_Records: persistent coordination/source-of-truth bridge.

## 3. Governing Records
Use these records as the implementation contract:
- 00_PROJECT_CONTROL/CHECKPOINTS/Chat41_Node7_Phase1b_Master_Handoff_Checkpoint.md
- 00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md
- 03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md
- 02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md
- 02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md
- 02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md

## 4. Global Implementation Boundary
Phase 1b is frontend-only.

Allowed:
- page and route presentation
- navigation presentation and role-aware shared navigation
- component presentation/refactoring
- responsive layout and accessibility improvements
- typography, spacing, visual hierarchy and shared status treatment
- presentation of existing data
- verified frontend defect correction
- Reviewer rejection interaction replacement with an in-app UI

Protected — do not change unless separately investigated and explicitly approved:
- APIs and API contracts
- database/schema/data model
- RLS and security architecture
- authentication and role rules
- business rules
- trip lifecycle/state semantics
- claiming/marketplace behavior
- evidence requirements/types/integrity
- persistent review state
- backend behavior
- AI behavior
- Reviewer authority/responsibility expansion

Do not invent missing backend data, endpoints, states, evidence requirements, or business rules.

## 5. Shared Foundation
Implement the shared authenticated navigation/layout carefully because Driver, Company, and Reviewer depend on it.

Requirements:
- role-aware navigation
- no invalid cross-role destinations
- preserve existing auth/role protection
- consistent design language across portals
- light theme, LTR, 4px spacing system
- responsive behavior
- accessible controls and visible text status
- restrained, purposeful motion only

Do not use a frontend change to bypass or weaken backend security/RLS.

## 6. Driver Implementation
Target structure:
- Dashboard
- Available Trips
- Trip Detail
- My Active Trip
- Completed Trips / History
- Profile

Implement the locked Driver blueprint and preserve existing Driver capabilities:
- existing trip retrieval
- claiming
- active-trip progression
- event recording
- completed-trip history/timeline
- authentication and authorization

Address verified Phase 1b gaps including navigation/structure, Available Trips accessibility, strict one-next-step lifecycle presentation, missing/inconsistent states, and the locked blueprint's evidence/status presentation expectations only where supported by existing data.

Do not invent an evidence model or change trip lifecycle semantics.

## 7. Company Implementation
Target structure:
- Dashboard
- My Created Trips
- Incoming Deliveries
- History / Timeline
- Profile / Account

Implement the locked Company blueprint and preserve existing Company workflows:
- trip creation/public sharing
- sender/receiver relationship semantics
- incoming delivery workflows
- receiver check-in/completion
- existing authentication/security

Address verified Phase 1b gaps including sender visibility, history/navigation, Company profile/account, mobile navigation, responsive Create Trip presentation, and workflow discoverability.

C-05 — receiver completion API response-shape issue — remains protected/out of scope for this frontend Phase 1b implementation. Do not change that API contract unless separately authorized.

## 8. Reviewer Implementation
Before Reviewer implementation, perform the narrow R-05 data-source readiness check required by the Boundary Decision.

Reviewer target flow:
Verification Queue → Applicant Verification → Evidence Examination → Decision Result → Verification History → read-only Verification Record → submitted evidence viewer.

Reviewer responsibility remains strictly identity and evidence verification.

Allowed Reviewer work:
- queue/workflow presentation
- applicant/evidence examination presentation using existing supported data
- explicit verification action presentation
- approve/reject presentation
- custom in-app rejection modal instead of native window.prompt
- history/record presentation only if existing data/query capability is verified during R-05 readiness check
- role-aware navigation correction

Protected:
- Reviewer API contract
- RLS/security architecture
- new persistent review state
- new evidence requirements
- scoring
- AI verification
- automated verification mechanism
- trip/delivery review
- general admin capabilities

R-03 remains protected/out of scope.

## 9. Execution Sequence — Mandatory
Never implement all three portals in one batch.

### Stage 1 — Driver
1. After explicit authorization, implement Driver only.
2. Build and run relevant tests/checks.
3. Inspect for runtime/build/console/navigation issues.
4. Record implementation evidence.
5. Ayush manually tests Driver.
6. Stop and fix verified problems if needed.
7. Do not start Company until Driver is accepted.

### Stage 2 — Company
1. Start only after Driver acceptance.
2. Implement Company only.
3. Build and run relevant tests/checks.
4. Inspect for regressions.
5. Record implementation evidence.
6. Ayush manually tests Company.
7. Stop and fix verified problems if needed.
8. Do not start Reviewer until Company is accepted.

### Stage 3 — Reviewer
1. Run the R-05 readiness check first.
2. Implement Reviewer only after readiness is verified.
3. Build and run relevant tests/checks.
4. Record evidence.
5. Ayush manually tests the complete Reviewer flow.
6. Stop and fix verified problems if needed.
7. Do not declare Phase 1b complete until Reviewer is accepted.

### Stage 4 — Cross-Portal E2E
Only after all three portals are individually accepted:
- test role separation
- shared navigation/layout
- Driver ↔ Company workflows
- trip lifecycle continuity
- evidence/timeline visibility
- responsive behavior
- regression surface
- final demo path

## 10. Evidence and Handoff Requirements
For every implementation stage, record:
- files changed
- reason for each meaningful change
- tests/checks run
- build result
- runtime/console findings
- screenshots or other evidence where useful
- known limitations
- anything that could not be verified
- explicit status: VERIFIED / INFERRED / UNKNOWN

Do not claim a feature works without evidence.

Antigravity must hand implementation results back through the Records repo. GitHub source pushes remain manually triggered by Ayush according to project governance.

## 11. Stop Conditions
Stop immediately and report back if:
- a requested change requires backend/API/schema/RLS/auth changes outside this scope
- locked blueprint requirements conflict with actual source behavior
- required data does not exist or cannot be safely read using existing supported mechanisms
- a change would alter business rules or lifecycle semantics
- a security/evidence-integrity concern appears
- implementation would require inventing data or behavior
- build/runtime behavior reveals an unexpected architectural dependency
- a cross-portal change has unclear ownership or impact

Do not silently solve a protected-boundary problem.

## 12. Final Handoff
After each portal, provide an implementation report and wait for Ayush's manual verification/acceptance before proceeding.

At the end, provide the final Phase 1b implementation/test/E2E evidence package. Phase 1b completion is not declared until Driver, Company, Reviewer, and cross-portal E2E have passed their respective gates.

## 13. Current State
This prompt is prepared as the execution contract, but its execution gate is intentionally CLOSED.

Current authorization: NOT AUTHORIZED.

No implementation should start until Ayush explicitly authorizes it.