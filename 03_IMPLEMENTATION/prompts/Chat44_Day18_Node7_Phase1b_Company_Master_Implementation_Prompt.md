# Chat44 — Day 18 — Node 7 — Phase 1b — Company Master Implementation Prompt

## 0. Execution Status

**IMPLEMENTATION SPECIFICATION — READY FOR AYUSH AUTHORIZATION**

This document is the execution specification for the Company Portal frontend implementation.

It does **not** itself authorize source-code changes. Antigravity must not begin implementation until Ayush explicitly authorizes execution.

---

## 1. Mission

Implement the **locked Company Portal frontend blueprint** using the existing application's APIs, data, authorization context, business rules, lifecycle semantics, evidence behavior, and backend capabilities.

The objective is to close the verified Company frontend gaps while preserving all protected system behavior.

### Non-negotiable rule

> **Frontend-only implementation. Reuse existing backend/data capabilities. Do not modify backend, database, RLS, authentication/authorization policy, business rules, lifecycle semantics, claiming, evidence integrity, AI behavior, or Reviewer authority.**

If the requested Company experience cannot be implemented safely within this boundary, **STOP and escalate**.

---

## 2. Governing Records — Read Before Coding

Read and follow these records in this order:

1. `00_PROJECT_CONTROL/ROADMAP.md`
2. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
3. `00_PROJECT_CONTROL/PROJECT_STATE.md`
4. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
5. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
6. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
7. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`
8. `01_BRAIN_HANDOFFS/Claude/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary_Claude_Review_Report.md`

The **locked Company Blueprint** and **locked Company Implementation Boundary** are authoritative for this implementation scope.

Do not reinterpret them into new product behavior.

---

## 3. Preflight — INSPECT BEFORE MODIFYING

Before changing source code:

1. Inspect the current Company routes and components.
2. Locate the existing Company Dashboard implementation.
3. Locate existing Incoming Deliveries, Receiver Check-in, Receiver Completion, Completed Deliveries, Public Share, and Create Trip surfaces.
4. Locate the shared `Navbar.tsx` and authenticated layout.
5. Identify the current Company data/API consumption points.
6. Identify the current `company_id` sender/created-trip data path.
7. Identify existing trip/event data that can support History/Timeline and Trip Detail.
8. Identify existing company/account identity data available for Profile/Account.
9. Identify the existing Driver Portal dependencies before changing shared components.
10. Inspect responsive behavior of the affected Company surfaces.

Do not assume a capability is missing merely because it is not visible in the current Company UI. Reuse existing supported data/capabilities where available.

### Required preflight output

Record:
- Current relevant files/routes.
- Existing reusable components.
- Existing API/data sources.
- Shared components affected.
- Any uncertainty.

If a required capability cannot be established from source, mark it **UNKNOWN** and stop if implementation would require assumption.

---

## 4. Target Company Information Architecture

Implement the locked Company structure:

**Dashboard → My Created Trips → Incoming Deliveries → History / Timeline → Profile / Account**

with a unified **Company Trip Detail** surface connecting relevant workflows.

### Required Company-facing surfaces

1. Dashboard
2. My Created Trips
3. Incoming Deliveries
4. Company Trip Detail
5. History / Timeline
6. Profile / Account
7. Existing Public Share integration

The exact route naming may follow the existing application convention, but the resulting navigation and information architecture must match the locked Blueprint.

---

## 5. Implementation Scope

### 5.1 Company navigation

Implement:
- Clear Company-facing navigation to Dashboard, My Created Trips, Incoming Deliveries, History/Timeline, and Profile/Account.
- Role-aware presentation using the existing auth/role foundation.
- Correct mobile navigation.
- Removal/correction of Company exposure to the shared `/timeline` entry where it is inappropriate for Company.

Do not create a second auth/authorization mechanism.

### 5.2 Dashboard

Restructure the Company Dashboard to provide:
- Needs Attention.
- Active Created Trips.
- Useful quick access.
- Clear separation between created/sent and incoming/receiving work.
- No duplicated Incoming Deliveries task list.

Use existing data/state. Do not introduce new business rules.

### 5.3 My Created Trips

Create the missing Company frontend surface for trips created/sent by the Company.

Use the existing `company_id` relationship.

Where existing data is available, present:
- Trip identity.
- Current status.
- Delivery progress.
- Driver/claim information.
- Receiver-action indicator.
- Next action.

Provide entry to Company Trip Detail.

Do not create or alter sender-side business logic or claiming behavior.

### 5.4 Incoming Deliveries

Restructure the existing receiving-company workflow into the dedicated Incoming Deliveries surface.

Present:
- Pending receiver tasks.
- Why an action is required.
- Current state.
- Perform Action affordance.
- Existing Receiver Check-in.
- Existing Receiver Completion.
- Appropriate post-action confirmation and state presentation.
- Navigation to Company Trip Detail.

Do not change the underlying receiver workflow or API contracts.

### 5.5 Company Trip Detail

Build one reusable Company Trip Detail presentation surface using existing data.

Present, where supported by existing data:
- Current Status.
- Visual Delivery Progress.
- Next Required Action.
- Driver / Claim information.
- Trip Details.
- Delivery Evidence.
- Timeline / History.
- Permitted Public Share entry point.

Use the same conceptual surface when entered from Dashboard, My Created Trips, Incoming Deliveries, or History.

Do not create new lifecycle states.

### 5.6 History / Timeline

Build/restructure Company History as a dedicated Company-facing surface.

It should:
- Surface Company-participated trips using existing data.
- Be read-only at the history-list level.
- Link to Company Trip Detail.
- Show detailed existing event history within Trip Detail.
- Avoid dependence on an inappropriate shared/Driver-focused `/timeline` navigation surface.

Do not create a new history data model.

### 5.7 Profile / Account

Build a dedicated Company Profile/Account presentation surface using existing company/account data and authentication context.

Do not create a new account-management subsystem.

### 5.8 Public Share

Preserve the existing Public Share behavior.

Frontend work may improve discoverability and place the permitted entry point within Company Trip Detail.

The existing receiving-company-only authorization/projection behavior must remain unchanged.

### 5.9 Responsive/mobile

Ensure Company works appropriately on:
- Phone.
- Tablet.
- Desktop.

Specifically verify:
- Mobile navigation.
- Dashboard layout.
- Created Trips list.
- Incoming Deliveries actions.
- Trip Detail.
- History.
- Profile.
- Create Trip small-screen presentation.
- Public Share presentation.
- No ordinary horizontal scrolling caused by layout defects.

---

## 6. Data and API Rules

### Reuse only

Prefer existing:
- APIs.
- Fetch/query patterns.
- Data models.
- Components.
- State values.
- Event/history information.
- Authentication context.

Do not create new APIs merely to simplify frontend implementation.

### `company_id` distinction

Treat `company_id` as the existing relationship identifying trips created by the Company where applicable.

Do not confuse:
- Created/sent Company trips, and
- Incoming/receiving Company trips associated through the existing receiving-company relationship.

The frontend must present these relationship contexts separately while respecting the same underlying trip lifecycle.

---

## 7. Protected Boundaries — DO NOT MODIFY

The following are outside scope:

- API contract changes.
- API response-shape changes.
- New backend endpoints/business functionality.
- Database schema changes.
- Data migrations.
- RLS changes.
- Authentication implementation changes.
- Authorization policy changes.
- Role assignment changes.
- Lifecycle semantics.
- Business rules.
- Claiming / marketplace behavior.
- Evidence requirements/integrity.
- Persistent workflow-state semantics.
- AI behavior.
- Reviewer authority/workflow.

### C-05 — Explicit protected file

Do not modify:

`src/app/api/completion/route.ts`

The Receiver Completion response-shape mismatch is a known protected issue.

Adapt the frontend safely to the established contract if possible. If the contract itself must change, **STOP and escalate**.

### Driver protection

The Driver Portal is already accepted/locked.

Any shared component change must be evaluated for Driver regression. If safe compatibility cannot be established, **STOP and escalate**.

---

## 8. Stop Conditions

Immediately stop implementation and write an escalation/blocker record if:

1. A required Company capability cannot be implemented with existing APIs/data.
2. A new API endpoint appears necessary.
3. An API response contract appears to require modification.
4. Database/schema work appears necessary.
5. RLS/security policy work appears necessary.
6. Authentication/authorization logic appears necessary.
7. A new business rule or lifecycle state appears necessary.
8. Claiming/marketplace behavior must change.
9. Evidence integrity/requirements must change.
10. AI behavior must change.
11. Reviewer authority/workflow must change.
12. A shared-component change may regress Driver and cannot be safely bounded.
13. Source evidence contradicts a locked Blueprint decision.
14. Required source/data behavior is UNKNOWN and cannot be established safely.
15. Implementation would require inventing product behavior.

Never silently bypass a stop condition.

---

## 9. UX / State Presentation Rules

Use existing backend state values and lifecycle semantics.

The frontend may improve:
- Labels.
- Grouping.
- Visual hierarchy.
- Progress presentation.
- Attention indicators.
- Empty states.
- Loading states.
- Error presentation.
- Success confirmation.
- Navigation.

It must not redefine what a state means.

Receiver actions must remain tied to existing permitted actions.

---

## 10. Shared Component Safety

Before modifying a shared component:

1. Identify every known portal using it.
2. Determine whether the change is presentation-only or behavior-affecting.
3. Preserve Driver behavior.
4. Preserve the locked shared design-system decisions.
5. Prefer Company-specific composition when a shared change would introduce unnecessary risk.

If impact cannot be bounded, stop.

---

## 11. Build / Test / Evidence

After implementation:

### Build/test

Run the project's appropriate existing build, lint, type-check, and/or test commands without changing configuration solely to hide failures.

Record:
- Commands run.
- Results.
- Failures/warnings.
- Whether failures are pre-existing or introduced.

### Functional evidence

Capture evidence for:

- Company navigation.
- Dashboard.
- My Created Trips.
- Incoming Deliveries.
- Receiver Check-in.
- Receiver Completion frontend handling.
- Company Trip Detail.
- History/Timeline.
- Profile/Account.
- Public Share preservation.
- Responsive/mobile behavior.
- Driver Portal non-regression for affected shared components.

### Scope evidence

Report:
- Files changed.
- Routes added/changed.
- Components added/changed.
- Existing APIs/data reused.
- Protected files untouched.
- No DB/schema changes.
- No RLS/auth/business-rule changes.

Do not claim verification without evidence.

Use:
- **VERIFIED** when directly evidenced.
- **INFERRED** when reasoned from evidence but not directly tested.
- **UNKNOWN** when not established.

---

## 12. Required Implementation Report

Create the implementation report at:

`03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Implementation_Report.md`

The report must include:

1. Executive result.
2. Preflight findings.
3. Files changed.
4. Routes/surfaces implemented.
5. APIs/data sources reused.
6. Company Blueprint coverage matrix.
7. Responsive/mobile evidence.
8. Build/test evidence.
9. Driver regression/non-regression evidence.
10. Protected-boundary verification.
11. Any blockers/UNKNOWNs.
12. Final implementation status.

### Final status rule

If implementation is complete and evidence is available, report:

**IMPLEMENTATION COMPLETE — AWAITING AYUSH MANUAL VERIFICATION**

Do not report Company as accepted or locked.

---

## 13. Mandatory Ayush Gate

After implementation evidence is complete:

> **STOP. Do not continue into Reviewer implementation, cross-portal integration, or additional Company scope. Await Ayush's manual browser verification and explicit acceptance.**

The next project state is determined only after Ayush reviews the implemented Company Portal.

---

## 14. Execution Discipline

Follow the project workflow:

**OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION → FIX → BUILD/TEST → AYUSH MANUAL VERIFICATION**

Do not:
- Expand scope because a backend change appears convenient.
- Repair protected backend defects as part of Company frontend work.
- Invent missing APIs.
- Invent business rules.
- Modify security behavior.
- Modify Driver behavior without bounded compatibility evidence.
- Declare acceptance without Ayush verification.

If the implementation can be completed fully within this prompt and the locked boundary, proceed.

If not, stop and escalate with the exact protected boundary involved, evidence, and proposed decision required.

---

## 15. Authorization

**Current authorization state: NOT YET AUTHORIZED FOR SOURCE IMPLEMENTATION.**

This prompt is ready for execution only after Ayush explicitly authorizes Company implementation.

Once authorized, Antigravity should execute this specification against the source repository and record its implementation evidence in the required Records location.
