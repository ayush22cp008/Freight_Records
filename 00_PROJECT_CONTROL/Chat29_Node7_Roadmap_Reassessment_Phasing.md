# Chat29 — Node 7 Roadmap Reassessment & Phasing

**Status:** APPROVED / LOCKED FOR EXECUTION  
**Date:** Sep 2, 2026 — Day 12  
**Reasoning brain:** Claude  
**Approved by:** Ayush  

## Context

Node 6 Security + Evidence was closed and accepted. Node 7 is the next active Node. The Node 7 execution strategy is materially expanded and reprioritized to support the final hackathon submission.

Two additional priorities were explicitly identified:

1. A full UI/UX redesign across the Driver, Company, and Reviewer portals because the current UI is functionally complete but confusing and difficult to navigate.
2. Additional high-value product/demo features beyond the original Node 7 baseline, subject to time and reliability.

The official Node 7 acceptance criteria remain unchanged. This record changes the **execution phasing, priorities, and scope protection**, not the acceptance gate.

## Node 7 Baseline — Preserved

The baseline remains:

- AI evidence-grounded summary
- Timeline integration
- Final API/UI integration
- End-to-end test
- Realistic demo data/scenario
- Security regression
- Critical bug fixing
- UX/demo polish
- Hackathon presentation/demo flow

## High-Priority Stretch

- **Public shareable read-only evidence link**
- **AI inconsistency detection**, only after baseline AI/timeline stability

## Optional / Time-Permitting

- Confidence/completeness scoring
- Multi-signal evidence cross-check
- Natural-language Q&A over deterministic evidence
- Video capture was evaluated as lowest priority and is cut from the active scope

## Scope Cuts / Protection

- Video capture → **CUT**
- AI inconsistency detection → **CONDITIONAL**
- Optional AI-depth enhancements → **CUT unless all higher-priority work is already complete early**
- Public shareable read-only evidence link → **KEPT**

## Approved Phasing

### Phase 1a — Baseline AI + Shareable Evidence

Build functionality first because the later UI redesign touches the same screens.

1. AI evidence-grounded summary
2. Timeline integration
3. Public shareable read-only evidence link

### Phase 1b — Full 3-Portal UI/UX Redesign

Redesign the final user-facing experience across:

1. Driver portal
2. Company portal
3. Reviewer portal

Use a unified visual/design system. This phase absorbs the remaining Final API/UI integration and UX/demo polish work where those concerns are coupled to the redesigned screens.

### Phase 3 — New Add-On Features (Conditional)

Only begin additional features after Phase 1a and Phase 1b are complete and stable. If no meaningful time remains, skip Phase 3 rather than starting speculative work.

### Final Step — Exactly Once

The final step is separate from the feature phases and occurs once at the end:

- Full E2E across all three roles
- Critical bug-fixing buffer
- Realistic demo data/scenario
- Hackathon presentation/demo flow
- Final rehearsal

Do not repeat the final E2E/demo cycle after every feature.

## Explicit Execution Sequence

```text
Phase 1a
   ↓
Phase 1b
   ↓
Phase 3 (conditional)
   ↓
Final step — E2E + bugfix + demo + presentation
```

## Non-Changes

- Node 7 official acceptance criteria remain unchanged.
- Nodes 1–6 remain locked and accepted.
- No prior Node is reopened.
- Node 7 remains the active final Node.

## Governance

This Chat29 record is the approved execution clarification for Node 7 and must be used as the planning reference for Day 14 / Chat30 onward. The older generic Node 7 stretch ordering is superseded by this phased execution plan.
