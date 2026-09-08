# Chat44 — Day 18 — Node 7 — Phase 1b — Company Implementation Boundary

## Status

**Company Implementation Boundary — PROPOSED / READY FOR LOCK**

This record defines the evidence-based implementation boundary for the Company Portal following the Day 18 / Chat44 existing-state implementation inspection.

It is derived from:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
3. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`
4. `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`

The current Company implementation gap is primarily **frontend restructuring, frontend surface creation, navigation, and responsive presentation**. Existing backend/data capabilities are to be reused rather than expanded.

---

## 1. Boundary Objective

Implement the locked Company Portal frontend blueprint using the **existing application APIs, data model, authorization, business rules, lifecycle semantics, and backend behavior**.

The Company implementation must close the verified frontend gaps without expanding into backend or product-rule work.

### Core rule

> **Company implementation is frontend-only. Existing backend/data capabilities may be consumed and presented, but protected backend behavior must not be changed.**

If a blueprint requirement cannot be implemented using existing supported capabilities, implementation must stop and escalate rather than inventing or adding backend functionality.

---

## 2. Allowed to Change — COMPANY FRONTEND SCOPE

The following areas are inside the Company implementation boundary.

### 2.1 Company navigation

Allowed:
- Restructure Company-facing navigation.
- Add clear frontend navigation access to the locked Company surfaces.
- Implement role-aware presentation using the existing authentication/role foundation.
- Correct navigation traps or inaccessible Company UI paths.
- Improve mobile navigation presentation.

Required target structure:
- Dashboard
- My Created Trips
- Incoming Deliveries
- History / Timeline
- Profile / Account

Do not create a second authentication or authorization system.

### 2.2 Company Dashboard

Allowed:
- Restructure the existing Company Dashboard.
- Surface existing created/sent trip data already available through existing data capabilities.
- Add Needs Attention presentation.
- Present Active Created Trips.
- Provide quick access to relevant Company workflows.
- Avoid duplicating the dedicated Incoming Deliveries task list.
- Normalize frontend state/status presentation using existing state data.

The Dashboard must remain a frontend presentation layer; it must not introduce new business rules.

### 2.3 My Created Trips

Allowed:
- Build the Company frontend surface for trips created/sent by the Company.
- Use the existing `company_id` data relationship.
- Present existing trip status, delivery progress, driver/claim information, receiver-action indicators, and next-action information where those data already exist.
- Provide navigation into the unified Company Trip Detail surface.

No new sender business logic or claiming behavior may be created.

### 2.4 Incoming Deliveries

Allowed:
- Extract/restructure the existing receiving-company workflow into a dedicated frontend surface.
- Present pending receiver tasks clearly.
- Show current state and required action using existing state/data.
- Provide existing Receiver Check-in and Completion actions.
- After successful existing action handling, present the returned result appropriately and navigate to the appropriate existing/new Company Trip Detail surface.
- Implement frontend empty states and attention presentation.

The underlying receiver workflow, authorization, lifecycle rules, and API contracts remain protected.

### 2.5 Unified Company Trip Detail

Allowed:
- Build a Company-facing unified Trip Detail frontend surface using existing data/API capabilities.
- Present:
  - Current Delivery State
  - Visual Delivery Progress
  - Next Required Action
  - Driver / Claim information
  - Trip Details
  - Delivery Evidence
  - Timeline / History
  - permitted Public Share entry point
- Provide consistent entry into the same Trip Detail surface from Dashboard, My Created Trips, Incoming Deliveries, and History where applicable.

Do not create new workflow states or alter lifecycle semantics.

### 2.6 History / Timeline

Allowed:
- Build or restructure a dedicated Company History / Timeline frontend surface.
- Surface existing Company-participated trips using existing data.
- Provide read-only access to the unified Trip Detail.
- Present detailed event history inside Trip Detail using existing events/data.
- Remove Company dependence on an inappropriate shared/driver-focused timeline entry point.

No new historical data model may be created.

### 2.7 Profile / Account

Allowed:
- Build a dedicated Company Profile / Account frontend surface using existing company/account data and existing authentication context.
- Provide presentation of existing information.

Do not create a new account-management subsystem or alter identity/authentication behavior.

### 2.8 Existing Public Share

Allowed:
- Preserve and reposition the existing Company Public Share capability within the locked Company Trip Detail experience.
- Improve frontend discoverability and presentation.

Required constraint:
- Public Share remains available only within the existing authorized receiving-company context.
- Do not change public projection or authorization behavior.

### 2.9 Responsive / mobile frontend

Allowed:
- Improve Company responsive layout for phone, tablet, and desktop.
- Correct mobile navigation.
- Correct small-screen Create Trip presentation.
- Prevent normal horizontal scrolling caused by Company UI layout.
- Improve responsive trip lists, dashboard, Trip Detail, receiver actions, History, Profile, and Public Share presentation.

### 2.10 Frontend component architecture

Allowed:
- Refactor Company frontend components for clarity and maintainability.
- Separate Company-specific dashboard presentation from interleaved Driver logic where required.
- Reuse existing shared components where safe.
- Create Company-specific presentation components where needed.
- Modify shared presentation components only when the change is demonstrably compatible with the locked Driver Portal and shared design system.

---

## 3. Must Preserve — Existing Working Capabilities

The following existing capabilities should be preserved unless a verified frontend restructuring is required:

- Company identity/authentication context.
- Existing Company trip creation/publishing workflow.
- Existing Receiver Check-in workflow.
- Existing Receiver Completion workflow.
- Existing Public Share capability and its receiving-company restriction.
- Existing trip/event data sources.
- Existing server-side authorization behavior.
- Existing lifecycle/state semantics.
- Existing evidence behavior and integrity requirements.
- Existing backend contracts.
- Existing Driver Portal behavior and locked Driver implementation.
- Shared design-system decisions already locked.

Preservation means **do not rewrite working backend/product behavior merely to make the frontend easier to implement**.

---

## 4. Protected — OUTSIDE COMPANY IMPLEMENTATION

The following are explicitly outside the Company implementation boundary:

### Backend / API
- API contract changes.
- API response-shape changes.
- New backend business functionality.
- Server-side workflow changes.
- New backend endpoints solely to support the Company redesign.

### Database
- Database schema changes.
- New tables/columns solely for Company UI.
- Data migration.
- Changes to existing data relationships.

### Security / Authorization
- RLS policy changes.
- Authentication implementation changes.
- Authorization policy logic changes.
- Role assignment changes.
- IDOR/security behavior changes.

### Product / Business Rules
- Lifecycle semantics.
- Claiming / marketplace behavior.
- Business rules.
- Receiver authority.
- Evidence requirements or integrity rules.
- Persistent workflow state semantics.
- AI behavior.
- Reviewer authority expansion.

### Known C-05 boundary

The Receiver Completion response-shape mismatch remains protected. The Company frontend must not modify the completion API contract or backend response solely to make the UI convenient.

If the existing frontend can be safely adapted to the established contract without changing protected behavior, that is a frontend implementation concern. If not, stop and escalate.

---

## 5. Stop & Escalate Conditions

Antigravity must stop Company implementation and record the blocker if any of the following occurs:

1. A required Company Blueprint capability cannot be implemented using existing APIs/data.
2. A new API endpoint appears necessary.
3. An existing API response contract appears to require modification.
4. Database/schema changes appear necessary.
5. RLS/security policy changes appear necessary.
6. Authentication or authorization policy changes appear necessary.
7. A new business rule or lifecycle state appears necessary.
8. Claiming/marketplace behavior must change.
9. Evidence integrity/requirements must change.
10. AI behavior must change.
11. Reviewer authority or Reviewer workflow must be changed.
12. A shared component change could regress the locked Driver Portal and the impact cannot be safely bounded.
13. A locked Company Blueprint decision is contradicted by current source evidence.
14. Required source/data behavior is UNKNOWN and cannot be established safely.
15. Implementation would require inventing product behavior not present in the locked Blueprint.

**Do not work around a protected boundary by assumption.**

---

## 6. Company Scope by Blueprint Gap

| Company Blueprint Area | Current State | Boundary Treatment |
|---|---|---|
| Primary Navigation | Partially present | Frontend change allowed |
| Unified Dashboard | Partially present | Frontend restructuring allowed |
| My Created Trips | Missing | Frontend surface build allowed using existing data |
| Incoming Deliveries | Present but inline | Frontend restructure/extraction allowed |
| Unified Trip Detail | Missing | Frontend surface build allowed |
| History / Timeline | Partially present | Frontend surface/restructure allowed |
| Profile / Account | Missing | Frontend surface build allowed using existing data |
| Public Share | Present | Preserve; frontend repositioning allowed |
| Responsive / Mobile | Partially present | Frontend responsive work allowed |

These classifications originate from the Chat44 existing-state inspection and are implementation-planning inputs, not permission to modify protected system behavior.

---

## 7. Expected Implementation Result

At Company implementation completion, the frontend should expose the locked Company information architecture:

**Dashboard → My Created Trips → Incoming Deliveries → History / Timeline → Profile / Account**

with a consistent **Company Trip Detail** surface connecting the relevant workflows.

The implementation should reuse existing backend/data capabilities and preserve all protected system behavior.

---

## 8. Verification Requirements

Before Company can be considered implementation-complete, Antigravity must provide evidence covering:

- Files/components changed.
- Company routes/surfaces implemented or restructured.
- Existing APIs/data sources reused.
- Build/test result.
- Company role-aware navigation.
- Dashboard states.
- Created Trips visibility.
- Incoming Deliveries workflow.
- Unified Trip Detail.
- History/Timeline.
- Profile/Account.
- Public Share preservation and receiver-only behavior.
- Responsive/mobile behavior.
- Protected backend/API/database/security areas untouched.
- No Driver Portal regression introduced.

After implementation evidence is complete, **STOP for Ayush manual browser verification**. Do not declare Company accepted or locked without Ayush's manual verification and explicit acceptance.

---

## 9. Authorization Boundary

This document defines the scope; it does **not** by itself authorize source-code execution.

Execution authorization remains under Ayush's control according to the project workflow.

Antigravity must not begin Company source-code modification until Ayush explicitly authorizes implementation after reviewing the applicable implementation prompt.

---

## 10. Decision State

**Current state: PROPOSED / READY FOR LOCK**

This boundary should be treated as the governing Company implementation boundary only after the required checkpoint/lock decision is explicitly recorded.

Until locked, do not treat this document as authorization to expand scope or modify protected areas.
