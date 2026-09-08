# Chat44 — Day 18 — Node 7 — Phase 1b — Company Existing State Implementation Inspection Instruction

## Purpose

Establish a fresh, evidence-based picture of the **actual current Company Portal implementation** immediately before Company implementation-boundary definition.

This is a **source inspection only**. Do not modify source code, redesign the portal, or implement fixes.

This inspection intentionally re-baselines the existing Company implementation for the Day 18 / Chat44 implementation phase. Earlier Company investigations may be used as historical context, but current source-code evidence is authoritative for the present-state map.

## Governing Context

- Project: Freight — AI Builders Hackathon
- Node: Node 7 — AI + Final Integration + Demo
- Phase: Phase 1b
- Day: Day 18
- Chat: Chat44
- Portal under inspection: Company
- Driver Portal: complete, accepted, locked; do not modify or reopen
- Company Blueprint: locked
- Implementation Preparation: finalized/approved
- Next reasoning step after this inspection: Company Implementation Gap Map → Company Implementation Boundary

## Primary Objective

Inspect the current Company frontend in the `freight` source repository and produce a source-backed map answering:

1. What Company functionality already exists?
2. Where does it exist in the source?
3. What is already usable and should be preserved?
4. What exists but requires frontend restructuring or correction to align with the locked Company Blueprint?
5. What is missing and would need to be built on the frontend?
6. What apparent gaps depend on protected backend/API/security/business behavior and therefore must not be assumed implementable?
7. Which existing shared components could be affected by Company changes?

Do not make the final implementation-boundary decision in this report. Provide evidence for the reasoning step that will make that decision.

## Governing Reference Records

Inspect the current source implementation against these Records repo references:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
3. `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`
4. Existing Company investigation records under `05_DEBUGGING/investigations/` as historical context only

Do not reopen locked Company product decisions unless current source evidence reveals a genuine contradiction. If a contradiction is found, record it explicitly with source evidence and stop rather than resolving it independently.

## Inspection Method

Inspect the actual current source tree, routes, components, API usage, and relevant configuration. Trace important Company workflows through their real frontend implementation.

For each finding, distinguish:

- **VERIFIED** — directly supported by current source evidence.
- **INFERRED** — reasonable conclusion from source evidence but not directly established.
- **UNKNOWN** — cannot be established from the inspected source; do not guess.

Use exact source paths and relevant route/component/function names wherever practical.

## Required Inspection Areas

### 1. Company route/page inventory

Identify every currently implemented Company-facing route/page relevant to the locked blueprint.

For each, document:
- route/path
- source file/component
- purpose
- Company-specific vs shared
- access/role handling visible in frontend
- primary data/API dependency
- current state of implementation

### 2. Current Company navigation

Inspect the actual navigation used by Company:
- desktop navigation
- mobile navigation
- links/destinations
- active states where relevant
- role-aware visibility
- inaccessible/dead-end navigation discovered in source
- shared navigation components

Determine which locked Company navigation surfaces already have a reachable entry point and which do not.

### 3. Dashboard / home

Map the actual Company dashboard:
- visible sections/cards
- created/sent trip information
- incoming/receiving information
- active trips
- completed information
- pending actions/attention states
- quick actions
- source data and APIs

Do not redesign it. State what exists and how it is currently assembled.

### 4. Created / sent trip visibility

Trace how a Company that created/published a trip can currently find and inspect that trip after publication.

Determine:
- whether sender-created trips are surfaced
- where they are surfaced
- what status/driver/claim/progress information is available
- whether existing APIs already provide information that the frontend does not surface
- whether the gap is frontend-only or appears to require protected behavior

### 5. Incoming / receiving delivery workflow

Inspect the existing receiving-company workflow:
- incoming delivery listing
- receiver action visibility
- check-in
- completion
- state presentation
- success/error handling
- navigation after actions

Pay particular attention to known response-shape inconsistencies. Document the current frontend expectation and actual API contract without changing either.

### 6. Trip Detail / unified trip view

Identify all existing Company trip-detail surfaces and determine:
- whether a unified detail page exists
- what trip identity/status information is shown
- visual progress
- next action
- driver/claim information
- trip details
- evidence
- timeline/history
- public sharing
- entry points from Dashboard, Created Trips, Incoming Deliveries, and History

Record separate surfaces if the current implementation has fragmented trip views.

### 7. Evidence and Public Share

Inspect existing Company evidence/delivery-proof and Public Share implementation.

Document:
- source locations
- route(s)
- authorization assumptions visible in frontend
- information exposed
- how Company reaches the capability
- whether the capability should be preserved as-is based on the locked blueprint

Do not alter authorization or public projection behavior.

### 8. Completed Trips / History / Timeline

Determine what Company currently has for:
- completed trips
- history
- event timeline
- read-only trip review
- navigation/accessibility of those surfaces

Distinguish between an existing capability that is merely hard to reach and a genuinely missing capability.

### 9. Profile / Account

Inspect whether Company has a dedicated Profile/Account surface.

Document existing routes, components, account/company data sources, and navigation access. If absent, state UNKNOWN or NOT FOUND based on source evidence rather than inventing a subsystem.

### 10. Responsive/mobile implementation

Inspect Company-specific and shared responsive behavior for:
- navbar/navigation
- dashboard
- trip lists
- trip detail
- Create Trip
- receiver actions
- public share
- history/profile if present

Record concrete implementation observations only. Do not prescribe the redesign here.

### 11. Shared vs Company-specific components

Map the important components used by Company and identify which are shared with Driver or Reviewer.

For each important shared component, document:
- source path
- Company usage
- other portal usage when evident
- whether a Company change could have cross-portal impact

Do not modify any shared component during this inspection.

### 12. Frontend → API/data dependencies

For each major Company workflow, identify the existing API/data source and relevant response fields where useful.

Focus on establishing current capability and dependency boundaries.

Do not:
- create new APIs
- change response shapes
- change database schema
- change RLS/security
- change authentication/authorization
- change lifecycle/business rules

### 13. Current implementation defects and structural gaps

Record concrete source-level observations under these categories where applicable:

- Already implemented / preserve
- Implemented but presentation/navigation needs change
- Implemented but frontend defect exists
- Backend/data capability exists but Company UI does not surface it
- Missing frontend surface
- Shared-component dependency/risk
- Protected-boundary dependency
- UNKNOWN / needs further evidence

Do not turn these categories into final implementation decisions.

## Required Blueprint Comparison Matrix

After the raw implementation inspection, produce a comparison matrix against the locked Company Blueprint with at least these columns:

| Locked Company Blueprint Requirement | Current Source Implementation | Evidence / Source Path | Classification | Likely Gap Type | Protected Dependency? |
|---|---|---|---|---|---|

Classification must use:
- PRESENT
- PRESENT — NEEDS RESTRUCTURE
- PRESENT — DEFECT
- PARTIALLY PRESENT
- NOT FOUND / MISSING
- UNKNOWN

This matrix is an **evidence map**, not the final implementation boundary.

## Protected Areas

During inspection, treat the following as protected and do not modify or reinterpret them:

- API contracts / response shapes
- database schema
- RLS/security policy
- authentication/authorization policy logic
- role assignment
- business rules
- lifecycle semantics
- claiming / marketplace behavior
- evidence requirements and integrity
- persistent workflow state
- AI behavior
- Reviewer authority
- backend behavior generally

Known Company issue **C-05 Receiver Completion response-shape mismatch** must be documented as observed current behavior and protected from implementation changes unless a later explicit decision changes the boundary.

## Stop / Escalation Conditions

Stop the inspection and report the issue clearly if:

- source cannot establish the behavior;
- a required blueprint capability appears to require new backend functionality;
- an API contract must change to satisfy a UI requirement;
- authorization/security behavior would need modification;
- a database/schema change appears necessary;
- a lifecycle/business rule would need to change;
- a shared component change creates an unclear Driver/Reviewer impact;
- a current source contradiction with a locked decision is discovered;
- the repository state is insufficient to establish the finding confidently.

Do not solve these conditions by assumption.

## Required Final Report Structure

1. Executive Summary
2. Repository / Source Inspection Scope
3. Current Company Route/Page Inventory
4. Current Company Navigation
5. Current Dashboard/Home
6. Current Created/Sent Trip Visibility
7. Current Incoming/Receiving Workflow
8. Current Trip Detail / Unified Trip Surfaces
9. Evidence & Public Share
10. Completed Trips / History / Timeline
11. Profile / Account
12. Responsive / Mobile Structure
13. Shared vs Company-Specific Components
14. Frontend → API/Data Dependencies
15. Concrete Current Defects / Structural Gaps
16. Locked Blueprint vs Current Implementation Comparison Matrix
17. Protected-Boundary Dependencies
18. VERIFIED / INFERRED / UNKNOWN Summary
19. Recommended Inputs for the Company Implementation Boundary — evidence-based observations only; no final boundary decision

## Completion Condition

The investigation is complete only when the report provides a sufficiently detailed, current, source-backed map of the Company Portal to allow ChatGPT and Ayush to determine:

**what already exists → what should be preserved → what must change → what must be built → what is protected → what remains UNKNOWN.**

No source code changes are permitted as part of this task.

## Required Output Location

Write the completed investigation report to:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`

Do not create or modify any implementation prompt during this inspection.
