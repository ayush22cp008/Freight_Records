# Claude Review Prompt — Freight Shared Design System

Review this file:

`00_PROJECT_CONTROL/Chat41_Node7_Phase1b_Shared_Design_System_Decisions.md`

Review it against the current Freight Records repository and the applicable locked Driver, Company, and Reviewer decisions.

## Your task

Act as an **independent reviewer**, not the implementation agent.

Determine whether the proposed shared design foundation is sufficiently grounded and safe to become the common design basis for Driver, Company, and Reviewer before implementation.

Focus on:

1. **Source grounding** — identify anything unsupported by the existing Records/evidence.
2. **Locked-boundary safety** — identify any proposal that conflicts with or unintentionally reopens a locked portal decision.
3. **Cross-portal consistency** — verify that shared design rules are appropriate for all three roles while allowing role-specific workflows to remain different.
4. **UX coherence** — assess the proposed principles, hierarchy, and visual language against the current Freight product model.
5. **Color system** — review the rationale and proposed tokens; identify anything that should change and any combinations that require accessibility/contrast validation.
6. **Accessibility** — identify missing or weak accessibility requirements.
7. **Missing decisions** — identify any important shared design requirement that must be resolved before implementation preparation.
8. **Overreach** — flag anything that could introduce functionality, business logic, authorization, lifecycle, evidence, marketplace, claim, or AI changes.
9. **Implementation readiness** — assess whether the document gives an implementation agent enough shared visual/UX direction without requiring it to invent the design system.

## Important rules

- Do **not** redesign the Freight product.
- Do **not** change locked workflows.
- Do **not** invent requirements when the Records do not support them.
- Clearly distinguish **SUPPORTED**, **INFERRED**, **PROPOSED**, **CONFLICT**, and **UNKNOWN**.
- Treat the exact HEX colors as proposed design choices, not existing Freight facts.
- Do not declare the document locked. Ayush makes the final decision after review and reconciliation.

## Return your review in this structure

### Overall Verdict
APPROVE / APPROVE WITH CHANGES / REJECT

### Confirmed Strengths

### Required Changes

### Conflicts or Boundary Risks

### Accessibility / Color Findings

### Missing Decisions

### Unsupported or Weakly Supported Claims

### Implementation Readiness

### Final Recommendation

Keep the review focused and evidence-based. Do not create implementation code.