# Chat40 — Day 15 — Node 7 — Phase 1b
# Reviewer Mental Model — Investigation Instructions

## Purpose

Now that the Existing Reviewer System Investigation is complete, establish the **Reviewer Mental Model** from the evidence-backed current system.

This is a reasoning/architecture step. It must explain how the Reviewer should understand the product and work within it before any interaction mapping, visual redesign, implementation plan, or source-code change.

## Evidence Baseline

Use these Records as the authoritative evidence baseline:

- `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Report.md`
- `05_DEBUGGING/investigations/Chat40_Day15_Node7_Phase1b_Existing_Reviewer_System_Investigation_Completion_Report.md`
- Existing locked Node 7 / Phase 1b project-control records.

Do not invent current capabilities that are absent from the investigation evidence.

## Strict Scope

This task is **Reviewer Mental Model only**.

Do not:
- implement code;
- modify source application files;
- redesign UI;
- create the Reviewer Interaction Mapping;
- create the Final Reviewer Blueprint;
- create an implementation plan;
- change backend/API/database/security behavior;
- silently turn a product idea into a locked decision.

The Mental Model may identify problems, principles, priorities, and questions that later influence design, but it must not prematurely decide implementation details.

## Core Question

Answer:

**"Who is the Reviewer in the Freight product, what is the Reviewer responsible for, what information does the Reviewer need to understand, what decisions does the Reviewer make, what states does the Reviewer encounter, and what should the product mental model communicate to make those responsibilities clear?"**

## Required Mental Model Coverage

### 1. Reviewer Role Definition

Establish the Reviewer role from evidence.

Document:
- Who the Reviewer is in the current system.
- How Reviewer access is established.
- What responsibility the current system assigns to Reviewer.
- What Reviewer does NOT own.
- Boundary between Reviewer responsibility and Company/Driver responsibility.

Separate:
- `VERIFIED` current-system facts.
- `INFERRED` mental-model conclusions.
- `UNKNOWN` areas requiring later clarification.

Do not assume that the Reviewer should become an operational freight administrator merely because that could be useful.

### 2. Reviewer Primary Job-to-be-Done

Define the smallest evidence-grounded job the Reviewer is currently performing.

Cover:
- Trigger/input.
- Information needed.
- Evaluation/verification activity.
- Decision.
- Outcome.
- What happens after the decision.

Do not optimize the workflow yet. First describe the mental model of the responsibility.

### 3. Reviewer Information Model

Identify the conceptual information the Reviewer must understand.

At minimum consider the evidence-backed concepts:
- Applicant.
- Requested role.
- Identity verification state.
- Submitted onboarding evidence.
- Evidence status.
- Approval/rejection decision.
- Rejection reason.
- Provisioned business identity where applicable.

For each concept explain:
- Why it matters to Reviewer.
- What state/meaning the Reviewer needs to understand.
- Whether it is `VERIFIED`, `INFERRED`, or `UNKNOWN`.

Do not invent fields that the existing system does not support.

### 4. Reviewer Decision Model

Describe the Reviewer decision as a mental model, not UI implementation.

Establish:
- What Reviewer is deciding.
- What evidence informs the decision.
- Valid decision outcomes.
- What additional information is required for rejection.
- What changes in the system after approval.
- What changes after rejection.
- What the Reviewer needs to trust before committing a decision.

### 5. Reviewer State Model

Model the states relevant to Reviewer understanding.

At minimum investigate:
- Pending.
- Approved / Verified.
- Rejected.
- Evidence pending/available as applicable.
- Processing/action-in-progress.
- Error/failure.

Only include states supported by evidence. If a state is not actually represented or its behavior is unclear, mark it `UNKNOWN` or `NOT VERIFIED`.

### 6. Reviewer Mental Journey

Describe the conceptual journey:

`Enter → Understand pending work → Inspect applicant/evidence → Decide → Confirm outcome → Continue`

This is **not** Interaction Mapping. Do not specify exact screens, components, buttons, navigation mechanics, or visual layout unless needed only to explain the current mental model.

The purpose is to define the Reviewer’s cognitive model and responsibility flow.

### 7. Trust and Evidence Model

Establish what makes a Reviewer decision trustworthy.

Cover:
- Evidence visibility.
- Evidence status.
- Identity information.
- Decision consequences.
- Rejection reason.
- Authorization boundary.
- Any audit/history expectations that are supported or not supported by the current system.

Clearly distinguish what exists today from what the mental model suggests the Reviewer needs to understand.

### 8. Boundary Model

Explicitly define what is inside and outside the Reviewer responsibility.

Use evidence from the completed investigation to distinguish:

**Inside Reviewer responsibility**
- Current onboarding identity verification responsibilities.

**Outside current Reviewer responsibility**
- Company operational trip management.
- Driver operational workflow.
- Delivery lifecycle operations.
- Other capabilities not supported by the existing Reviewer system.

If a boundary is uncertain, mark it `UNKNOWN` rather than deciding it here.

### 9. Mental-Model Problems in the Current System

Identify cognitive/structural problems revealed by the existing system, such as:
- Reviewer being trapped by shared navigation.
- Role-confusion when identities overlap.
- Flat queue with limited context.
- Native prompt-based rejection reasoning.
- Silent post-decision refresh.
- Lack of historical visibility where verified.

Do not turn these into implementation fixes. Explain the mental-model problem they create for the Reviewer.

### 10. Mental Model Principles

Derive a concise set of principles that a future Reviewer design should respect.

Examples may include:
- Reviewer should understand **what requires attention now**.
- Reviewer should understand **who the applicant is and what role they requested**.
- Reviewer should understand **what evidence supports the decision**.
- Reviewer should understand **what decision they are making and its consequence**.
- Reviewer should never be confused about whether an action succeeded.
- Reviewer should not be presented with navigation/actions belonging to another persona.

These are mental-model principles, not final UI decisions.

### 11. KEEP / REDESIGN / FIX / MISSING / UNKNOWN Interpretation

Use the completed investigation baseline as input, but do not automatically convert every investigation classification into a final design decision.

Explain which findings affect:
- Mental model.
- Information architecture later.
- Interaction design later.
- Security boundary.
- Scope.

Keep verified facts separate from proposed mental-model implications.

### 12. Open Questions / Unknowns

Create an explicit list of unresolved questions.

For each:
- Question.
- Why it matters.
- Current evidence.
- Classification: `UNKNOWN` / `NOT VERIFIED`.
- Whether it must be resolved before Interaction Mapping.

Do not fill unknowns with assumptions.

## Required Final Record Structure

The completed Mental Model record must contain:

1. **Status**
2. **Evidence Baseline**
3. **Reviewer Role Definition**
4. **Primary Job-to-be-Done**
5. **Information Model**
6. **Decision Model**
7. **State Model**
8. **Mental Journey**
9. **Trust and Evidence Model**
10. **Boundary Model**
11. **Current Mental-Model Problems**
12. **Mental Model Principles**
13. **Relationship to Existing-System Findings**
14. **Open Questions / Unknowns**
15. **Scope Guardrails for Next Step**
16. **Conclusion / Readiness for Interaction Mapping**

## Completeness Rule

Do not declare the Mental Model complete until every required section above has been addressed.

If a required point cannot be established from the existing evidence, explicitly state `UNKNOWN`, `NOT VERIFIED`, or `NOT APPLICABLE` and explain why.

Do not silently omit requirements.

## Next-Step Gate

Only after this Mental Model is complete should the project proceed to:

**Reviewer Interaction Mapping**

Do not start Interaction Mapping inside this record.

## Authority / Project Position

- Chat: **Chat40**
- Day: **Day15**
- Node: **Node 7**
- Phase: **Phase 1b**
- Driver Blueprint: **COMPLETE / LOCKED**
- Company Blueprint: **COMPLETE / LOCKED**
- Reviewer Existing-System Investigation: **COMPLETE**
- Reviewer Mental Model: **CURRENT TASK**
- Reviewer Interaction Mapping: **NOT STARTED**
- Reviewer Blueprint: **NOT STARTED**
- Implementation: **NOT STARTED**

## Quality Gate

Before closing this task, verify:

- [ ] Existing Reviewer investigation was used as evidence baseline.
- [ ] Current-system facts are separated from inference.
- [ ] Reviewer role is explicitly defined.
- [ ] Reviewer job-to-be-done is defined.
- [ ] Information model is defined.
- [ ] Decision model is defined.
- [ ] State model is defined.
- [ ] Mental journey is defined without becoming Interaction Mapping.
- [ ] Trust/evidence model is defined.
- [ ] Responsibility boundaries are explicit.
- [ ] Current mental-model problems are identified.
- [ ] Mental-model principles are derived.
- [ ] Unknowns are explicitly recorded.
- [ ] No implementation/design/blueprint work was performed.
- [ ] Next step is clearly gated to Reviewer Interaction Mapping only.
