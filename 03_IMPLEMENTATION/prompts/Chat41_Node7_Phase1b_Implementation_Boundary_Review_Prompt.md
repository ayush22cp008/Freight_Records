# Chat41 — Node 7 Phase 1b — Implementation Boundary Review Prompt

**Status:** REVIEW PROMPT — IMPLEMENTATION NOT AUTHORIZED

## Objective

Perform the Implementation-Boundary Review for Node 7 Phase 1b before any implementation begins.

The implementation order already decided for Phase 1b is:

**Driver → Company → Reviewer**

This review must validate that order against the governing Records and establish the exact implementation boundary. It must not begin coding, modify locked blueprints, or create new product behavior.

## Governing Sources

Use the current Freight Records as the source of truth, especially:

- `00_PROJECT_CONTROL/Chat38_Day14_Node7_Phase1b_Frontend_Blueprint_Decisions.md`
- `00_PROJECT_CONTROL/Chat39_Day15_Node7_Phase1b_Company_Blueprint_Decisions.md`
- `00_PROJECT_CONTROL/Chat40_Day15_Node7_Phase1b_Reviewer_Mental_Model_Decisions.md`
- `00_PROJECT_CONTROL/Chat40_Day16_Node7_Phase1b_Reviewer_Interaction_Mapping_Decisions.md`
- `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
- `00_PROJECT_CONTROL/Chat41_Node7_Phase1b_Shared_Design_System_Decisions.md`
- Existing Node 7 Phase 1a completion and implementation records

If Records conflict, surface the conflict and stop. Do not silently reconcile it.

## Review Questions

### 1. Implementation Order

Validate:

1. Driver first
2. Company second
3. Reviewer third

Explain the dependency and validation rationale for this sequence.

### 2. Allowed Phase 1b Changes

Identify what may change during Phase 1b, including:

- page structure
- visual hierarchy
- navigation presentation
- component presentation
- spacing and responsive layout
- typography and visual language
- shared color/status treatment
- discoverability and interaction clarity
- verified frontend UI/UX defects

### 3. Protected Existing System

Explicitly identify what must remain unchanged unless separately investigated and approved:

- APIs
- existing data models
- business rules
- authorization rules
- delivery lifecycle/state model
- evidence model and evidence requirements
- marketplace behavior
- driver claim/claiming mechanisms
- security/evidence integrity mechanisms
- existing backend functionality
- AI behavior

### 4. Cross-Portal Shared Foundation

Verify that all three portals use the locked Chat41 shared design foundation while retaining their role-specific workflows.

Confirm that shared design consistency does not mean identical screens or identical actions.

### 5. Driver Boundary

Use the locked Driver blueprint as authoritative. Determine the implementation surface without inventing new Driver functionality.

### 6. Company Boundary

Use the locked Company blueprint as authoritative. Preserve the unified Company model and Sender/Receiver relationship-specific behavior.

### 7. Reviewer Boundary

Use the locked Reviewer mental model, interaction mapping, and blueprint as authoritative. Preserve the verification-first, evidence-centered workflow and narrow responsibility boundary.

Do not introduce scoring, AI verification, new automated verification mechanisms, new persistent review states, new evidence requirements, trip/delivery review responsibility, or general admin responsibility.

### 8. Shared Design Rules

Apply the locked Chat41 rules, including:

- light theme only for Phase 1b
- LTR/current product language only for Phase 1b
- 4px-based spacing scale
- semantic statuses: Neutral, Information, Warning / Attention, Success, Error
- meaningful status must have a visible text label
- color is supplementary, never the sole status signal
- icon is optional reinforcement, never a replacement for meaningful status text
- restrained, purposeful motion only
- responsive and accessible presentation

### 9. Implementation Sequence and Gates

Define the safe sequence:

**Boundary Review → Implementation Preparation → Explicit Implementation Authorization → Driver Build → Driver Test/Manual Verification → Company Build → Company Test/Manual Verification → Reviewer Build → Reviewer Test/Manual Verification → Cross-Portal E2E Verification**

No implementation authorization is implied by this review prompt.

## Required Output

Return:

1. Overall Boundary Verdict
2. Confirmed Implementation Order
3. Allowed Changes
4. Protected System Areas
5. Driver Implementation Boundary
6. Company Implementation Boundary
7. Reviewer Implementation Boundary
8. Shared Design-System Constraints
9. Cross-Portal Dependencies
10. Risks / Ambiguities / Conflicts
11. Required Decisions Before Build
12. Implementation Readiness Verdict
13. Explicit statement of whether implementation is authorized or still blocked

## Governance Rules

- Do not modify locked blueprints.
- Do not invent unsupported requirements.
- Do not implement fixes during this review.
- Keep investigation, decision, implementation, testing, and manual verification separate.
- Mark findings as VERIFIED, INFERRED, or UNKNOWN where appropriate.
- If an UNKNOWN or conflict could materially affect implementation, stop and surface it before build preparation.
- Antigravity executes implementation only after Ayush explicitly authorizes it.
