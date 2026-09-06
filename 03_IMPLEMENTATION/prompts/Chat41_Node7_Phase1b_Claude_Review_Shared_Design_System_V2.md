# Claude Review Prompt — Freight Shared Cross-Portal Design System V2

**Review target:** `00_PROJECT_CONTROL/Chat41_Node7_Phase1b_Shared_Design_System_Decisions.md`

## Purpose

Independently review the **reconciled** Chat41 Shared Cross-Portal Design System candidate before any final lock decision.

This is a second review because Chat41 materially changed after the first Claude review. Do not review the original candidate; review the current reconciled version in Records.

## Source-of-truth rules

- Treat the Freight Records repository and its current authoritative records as the source of truth.
- Preserve all locked Driver, Company, and Reviewer decisions.
- Do not invent missing requirements.
- Distinguish SOURCE-SUPPORTED, DESIGN INFERENCE, PROPOSED, and UNKNOWN/OPEN claims.
- Do not lock the document.
- Do not redesign the system.
- Do not authorize implementation.

## Review objectives

Verify specifically whether the reconciled Chat41 document is now safe and complete enough to proceed to the final lock gate and then the Implementation-Boundary Review.

### 1. Warning / Attention consolidation

Check whether merging the UI semantic categories `Attention` and `Warning` into **Warning / Attention** is logically sound and consistent with existing Freight Records.

Also verify that this UI semantic decision does not alter or conflict with any existing project-monitor WARNING severity terminology.

### 2. Status communication rule

Verify the reconciled rule:
- meaningful operational status requires a visible text label;
- color is supplementary;
- icon is optional reinforcement;
- color-only meaningful status is not allowed;
- icon-only meaningful status is not allowed;
- semantic meaning must remain consistent across Driver, Company, and Reviewer.

Check whether this is sufficiently precise without over-constraining implementation.

### 3. Contrast conclusions

Review the stated contrast findings and whether the conclusions are technically and internally consistent:
- Primary `#1D4ED8`
- Success `#15803D`
- Error `#B91C1C`
- Information `#0369A1`
- Warning / Attention `#B45309`
- former Attention `#D97706`
- Disabled `#94A3B8`

Check whether the document appropriately distinguishes token-level checks from actual component foreground/background validation.

### 4. Reviewer layout cross-check

Verify the explicit statement that the shared presentation hierarchy:

**Context → Current State → Important Information → Required Action → Supporting Evidence/History**

has no conflict with the locked Reviewer Mental Model and Reviewer Interaction Mapping.

Check that the shared hierarchy remains a presentation principle and does not accidentally override the Reviewer verification-first workflow.

### 5. Phase 1b boundaries

Verify that the reconciled boundaries are clear and safe:
- Light theme only for Phase 1b;
- LTR/current product language only for Phase 1b;
- dark mode is future work;
- i18n/RTL is future work;
- motion is minimal, purposeful, functional, and not the sole state communication;
- exact motion timing remains implementation-level;
- shared spacing uses the 4px-based scale without prescribing every screen.

Check whether any of these should be tightened or clarified before lock.

### 6. Locked blueprint protection

Cross-check Chat41 against the locked Driver, Company, and Reviewer records.

Identify any direct conflict, accidental workflow override, unauthorized feature implication, or terminology mismatch.

Do not reopen locked blueprints merely because a shared design principle could be interpreted differently. Flag only genuine conflicts or material ambiguity.

### 7. Shared-system completeness

Check whether any important shared design-system decision is still missing that must be resolved before implementation-boundary review.

Focus on decisions that materially affect cross-portal consistency, accessibility, status semantics, component behavior, or implementation safety.

Do not add speculative requirements simply to make the document longer.

### 8. Implementation readiness

Determine whether Chat41 is:
- ready to LOCK;
- needs minor changes before LOCK;
- needs material redesign/review;
- or should remain open because of a source conflict or unresolved boundary.

Be conservative. A clean review should not invent blockers.

## Required output

Return exactly these sections:

1. **Overall Verdict**
2. **Reconciled Changes Confirmed**
3. **Warning / Attention Review**
4. **Status Communication Review**
5. **Contrast Review**
6. **Reviewer Cross-Check**
7. **Phase 1b Boundary Review**
8. **Locked Blueprint Conflict Check**
9. **Remaining Missing Decisions**
10. **Unsupported or Overreaching Claims**
11. **Implementation Readiness**
12. **Final Recommendation**

For each finding, classify it as one of:
- CONFIRMED
- REQUIRED CHANGE
- MINOR CHANGE
- NO CONFLICT
- UNKNOWN / NEEDS EVIDENCE

## Final question

Answer this directly:

> **Is the reconciled Chat41 Shared Cross-Portal Design System safe to LOCK, with no remaining material conflict against the locked Driver, Company, and Reviewer blueprints?**

Do not perform the lock. The final lock decision belongs to Ayush after reviewing this second Claude review.