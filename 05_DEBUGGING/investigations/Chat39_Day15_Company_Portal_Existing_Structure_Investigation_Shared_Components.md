# Chat39 Day15 — Company Portal Existing Structure Investigation: Shared vs Company-Specific Components

## Investigation Purpose

Establish the evidence-based current frontend component structure of the Company portal, specifically which components are shared across roles and which are Company-specific.

This is an investigation only. Do not redesign the Company portal, change source code, create an implementation prompt, or make final Company UX decisions.

## Scope

Inspect the current source-code repository and determine:

1. Shared frontend components used by Company and other roles.
2. Company-specific frontend components.
3. Where role-specific behavior is assembled or selected.
4. Whether shared components introduce Company-specific structural inconsistencies.
5. Relevant source paths and evidence for each finding.
6. Confidence for each finding: VERIFIED / INFERRED / UNKNOWN.

## Required Investigation Areas

### 1. Shared Components

Identify components used by Company and at least one other portal/role, including where applicable:

- authenticated layout/shell
- Navbar/navigation
- common cards
- buttons and form components
- status/presentation components
- timeline/history components
- shared data-display components
- other reusable frontend components materially used by Company

For each component, record:

- component name
- source path
- roles/pages using it
- whether behavior changes by role
- evidence
- confidence

### 2. Company-Specific Components

Identify Company-only components/pages, including where applicable:

- Company Dashboard
- Create Trip workflow
- Receiver Check-in
- Delivery Completion
- Public Share management
- Company-specific status/action UI
- other Company-only frontend components

For each component, record its source path, purpose, usage, evidence, and confidence.

### 3. Role-Specific Assembly

Trace how the frontend determines and renders Company versus Driver/Reviewer experiences.

Determine:

- where authenticated role is resolved
- where Company UI is selected
- whether shared components receive role-specific props/state
- whether Company and Driver use the same structural shell
- whether any Company page is assembled from Driver-oriented components

Do not infer architecture where source evidence is absent.

### 4. Structural Implications

Document only source-backed observations, such as:

- useful existing component reuse
- duplicated structures
- shared components that do not properly distinguish roles
- shared navigation exposing a route that Company cannot actually use
- other concrete structural inconsistencies

Do not propose fixes yet.

## Important Existing Finding to Validate

Previous investigation identified that the authenticated Navbar is shared and exposes a Timeline link, while the Company user cannot successfully access `/timeline` because the route resolves a Driver record and redirects a standard Company user to `/`.

Validate the component-level cause and document it as an investigation finding. Do not fix it during this investigation.

## Evidence Rules

- Use concrete source-code paths.
- Prefer direct source evidence over assumptions.
- Mark every substantive conclusion VERIFIED, INFERRED, or UNKNOWN.
- Do not claim something is shared merely because it looks reusable; establish actual usage from source.
- Do not claim a component is Company-specific unless usage supports it.

## Output

Update the existing Company frontend investigation report with a completed Section: **Shared vs Company-Specific Components**.

Include:

1. Executive finding for this section.
2. Shared component inventory.
3. Company-specific component inventory.
4. Role-specific assembly findings.
5. Structural implications/gaps observed from the current architecture.
6. Source paths/evidence.
7. VERIFIED / INFERRED / UNKNOWN confidence summary.

## Strict Boundaries

- No source-code changes.
- No UI redesign.
- No new functionality.
- No implementation prompt.
- No API/backend changes.
- No final Company Mental Model decisions.
- Do not reopen or modify the locked Driver blueprint.
- Keep this investigation separate from implementation.
