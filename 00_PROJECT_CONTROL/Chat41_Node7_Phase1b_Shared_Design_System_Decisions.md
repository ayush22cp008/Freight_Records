# Freight — Shared Cross-Portal Design System Decisions

**Record:** Chat41 / Node7 / Phase1b
**Status:** **LOCKED**
**Lock authority:** Ayush (explicit final decision)
**Lock basis:** Reconciled candidate + Claude V2 independent review
**Purpose:** Establish one shared design foundation for Driver, Company, and Reviewer portals before implementation. This record consolidates the design reasoning already established in the current project context, Claude review findings, and the reconciled decisions from review.

---

## 1. Document Purpose

This record defines the shared design foundation that should apply across the Freight website/application for the remaining Phase 1b frontend work.

The purpose is not to redesign or reopen the locked Driver, Company, or Reviewer workflows. It establishes a common visual and UX language for shared details so implementation does not produce three visually unrelated portal experiences.

### Scope

This record covers:
- product/design foundation;
- shared UX principles;
- shared visual language;
- proposed color system;
- status semantics;
- accessibility principles;
- cross-portal consistency rules;
- shared theme/language/motion/spacing boundaries;
- boundaries protecting locked blueprints;
- implementation-level validation requirements that follow from the shared design system.

### Out of scope

This record does not authorize:
- new backend functionality;
- new authorization rules;
- new lifecycle states;
- new evidence requirements or evidence types;
- new marketplace behavior;
- new claim mechanisms;
- new AI behavior;
- changes to established business rules;
- changes to the locked Driver, Company, or Reviewer workflow models;
- dark theme implementation in Phase 1b;
- internationalization, localization, or RTL implementation in Phase 1b.

---

## 2. Evidence and Source Basis

The design foundation is derived from the existing Freight project material rather than invented independently.

### 2.1 Product/problem foundation

The original Freight project material establishes the central problem as the freight detention documentation and claim-generation/evidence problem: drivers and carriers can lack a sufficiently authoritative paper trail for arrival/departure and detention events, creating disputes around detention claims.

The market/feasibility research further identifies the core opportunity as a driver-centered evidence mechanism that creates a structured, shareable record of relevant events and evidence rather than merely providing another generic tracking or fleet-management product.

### 2.2 Current project evolution

The current Records repository is the source of truth for the evolved implemented Freight system. The original research explains why Freight exists and the evidence problem it addresses; current locked blueprints and project-control records govern the current portal workflows and system boundaries.

### 2.3 Current locked portal models

The current project state establishes three role-specific experiences:
- Driver;
- Company;
- Reviewer.

The locked Company model centers on one Company having multiple trips, trip-specific Sender/Receiver relationships, shared delivery visibility, and relationship/state-based actions.

The locked Reviewer Mental Model establishes Reviewer as an Identity & Evidence Verifier whose primary object is Evidence and whose verification model is Applicant + Role + Evidence → Evaluate → Verify → Approve/Reject.

The Driver blueprint is already locked and remains authoritative for its workflow and presentation decisions.

### Evidence classification

**SOURCE-SUPPORTED:** Product problem, evidence-centered purpose, role boundaries, and locked portal models described above.

**DESIGN INFERENCE:** The resulting shared visual character should communicate trust, evidence, operational clarity, and accountability.

**PROPOSED:** Exact visual tokens, color values, component language, and other shared visual implementation details in this document.

**UNKNOWN / OPEN:** Any requirement not explicitly established by project evidence and not necessary to define the shared design system.

---

## 3. Product Foundation

### 3.1 Product identity

**Decision:** Freight is a focused freight evidence/operations product centered on trustworthy structured records and role-specific workflows.

### 3.2 Core problem

**Decision:** The foundational problem is the gap between real-world freight/facility events and the ability of relevant parties to understand, verify, and rely on the resulting evidence.

### 3.3 Core purpose

**Decision:** Freight should make relevant freight activity and evidence understandable, structured, and usable for the role that must act on it.

### 3.4 Core value

**Decision:** The core value is trustworthy structured evidence and clear role-specific action around that evidence.

### 3.5 Product character

**Decision:** Freight should feel:
- professional;
- trustworthy;
- operational;
- evidence-centered;
- precise;
- clear rather than decorative.

---

## 4. Shared UX Principles

1. **Evidence first** — Important information should be presented around underlying evidence and system records rather than decoration.
2. **State clarity** — Users should understand the current state of important objects/workflows quickly.
3. **Action clarity** — When the existing workflow requires an action, the next required action should be visually obvious.
4. **Role clarity** — Each role should clearly understand its responsibility boundary.
5. **Timeline clarity** — Chronological information should be easy to follow wherever the workflow exposes history/timeline.
6. **Trust through transparency** — Where the existing system supports it, important information and decisions should expose the evidence/context supporting them.
7. **Consistency across portals** — Shared components, terminology, status semantics, and interaction patterns should remain consistent.
8. **Operational simplicity** — Avoid unnecessary decoration, excessive animation, and dashboard noise.
9. **Responsive by design** — Shared components/layouts should remain usable across relevant device sizes.
10. **Accessible by default** — Meaning must not depend on color alone; interactive states must remain understandable and usable.

---

## 5. Shared Visual Language

### 5.1 Overall direction

**Decision:** Freight uses a clean, professional, evidence-centered, operational, restrained visual language.

### 5.2 Visual personality

| Area | Locked direction |
|---|---|
| Overall style | Modern professional / operational |
| Visual density | Moderate |
| Decoration | Minimal |
| Shapes | Clean and restrained |
| Cards | Meaningful information grouping; avoid card overload |
| Borders | Subtle and functional |
| Shadows | Light / limited |
| Icons | Simple and consistent |
| Typography | Legibility and hierarchy first |
| Status treatment | Strongly distinguishable and consistent |
| Animation | Minimal and purposeful |
| Whitespace | Deliberate |

### 5.3 Shared hierarchy

**Decision:** Shared page presentation should generally communicate:

**Context → Current State → Important Information → Required Action → Supporting Evidence/History**

This is a shared presentation principle only. Individual portal blueprints remain authoritative for their actual information architecture and workflow.

### Reviewer cross-check

The hierarchy was explicitly checked against the locked Reviewer Mental Model and locked Reviewer Interaction Mapping.

**Result: NO CONFLICT.**

The hierarchy is compatible with the Reviewer sequence because:
- Context corresponds to Applicant + Claimed Role;
- Current State corresponds to Pending Verification / decision state;
- Important Information centers the submitted Evidence;
- Required Action corresponds to Evaluate → Identity/Role Verified → Approve/Reject;
- Supporting Evidence/History corresponds to evidence viewing and completed Verification History.

The shared hierarchy therefore remains a general presentation principle and does not override the Reviewer verification-first workflow.

### 5.4 Shared component language

The three portals share the fundamental visual vocabulary for:
- buttons;
- inputs;
- selects;
- cards/panels;
- tables/lists;
- badges;
- status indicators;
- alerts;
- dialogs/modals;
- tabs;
- navigation;
- timeline/event presentation;
- evidence presentation;
- empty states;
- loading states;
- error states;
- confirmation/success states.

Shared components do not mean identical screens. Role-specific workflows determine content/actions; the shared design system determines their visual and interaction language.

---

## 6. Shared Color System

### 6.1 Color strategy

**Decision:** Use a restrained deep-blue primary identity supported by neutral surfaces and standardized semantic colors.

**Source classification:** DESIGN INFERENCE / PROPOSED DESIGN DECISION. The project source material does not prescribe an official Freight color palette.

### 6.2 Locked color tokens

| Token | HEX | Purpose |
|---|---|---|
| Primary | `#1D4ED8` | Main Freight actions, active navigation, links |
| Primary Hover/Pressed | `#1E40AF` | Hover/pressed interaction |
| Primary Soft | `#EFF6FF` | Selected/soft primary surfaces |
| Background | `#F8FAFC` | Main application canvas |
| Surface | `#FFFFFF` | Cards, panels, forms |
| Text Primary | `#0F172A` | Main content |
| Text Secondary | `#475569` | Supporting information |
| Text Muted | `#64748B` | Muted supporting UI |
| Border | `#E2E8F0` | Dividers and component boundaries |
| Success | `#15803D` | Verified / completed / accepted |
| Success Soft | `#F0FDF4` | Success background |
| Warning / Attention | `#B45309` | Requires awareness/action; caution/important condition |
| Error | `#B91C1C` | Rejected / failed / invalid |
| Error Soft | `#FEF2F2` | Error background |
| Information | `#0369A1` | Informational state |
| Disabled | `#94A3B8` | Disabled controls |

### 6.3 Semantic consistency

The same semantic color meaning applies across Driver, Company, and Reviewer:
- **Neutral** → inactive/unspecified/non-semantic state;
- **Information** → informational content;
- **Warning / Attention** → requires awareness or action but is not an error/rejection;
- **Success** → verified/completed/accepted;
- **Error** → failed/rejected/invalid.

The UI semantic category **Warning / Attention** is separate from any project-monitor severity terminology. Existing Monitor WARNING severity is not changed by this design-system decision.

### 6.4 Status communication rule

**Decision:** Every meaningful operational status must have a visible text label. Color is supplementary, never the sole indicator. An icon may reinforce a status when useful, but an icon is not a replacement for the text label.

Requirements:
- Text status label: **REQUIRED**;
- Color: **SUPPLEMENTARY**;
- Icon: **OPTIONAL REINFORCEMENT**;
- Color-only status: **NOT ALLOWED**;
- Icon-only status for a meaningful state: **NOT ALLOWED**;
- Same semantic meaning across portals: **REQUIRED**.

Role-specific wording may be used where the underlying workflow/role terminology is explicitly established, but the underlying semantic meaning must remain consistent across portals.

### 6.5 Contrast validation

The proposed palette was checked against white for common normal-text usage during reconciliation:
- Primary `#1D4ED8` → 6.70:1;
- Success `#15803D` → 5.02:1;
- Error `#B91C1C` → 6.47:1;
- Information `#0369A1` → 5.93:1;
- Warning / Attention `#B45309` → 5.02:1;
- Former Attention `#D97706` → 3.19:1 and is not retained as the normal-text semantic token;
- Disabled `#94A3B8` → 2.56:1 and is restricted to appropriate disabled-state presentation rather than normal informational text.

Exact component-level combinations remain subject to implementation-level validation because contrast depends on actual foreground/background pairing and text size. In particular, text/icon use on `Primary Soft`, `Success Soft`, and `Error Soft` backgrounds must be checked when those component compositions are implemented.

---

## 7. Accessibility Requirements

### 7.1 Contrast

Actual foreground/background combinations must be checked during implementation. The palette is not approved solely by visual preference.

### 7.2 State communication

Do not rely solely on color, color-only dots/borders/charts, or an icon without a meaningful text label for an important operational state.

### 7.3 Focus and interaction

Interactive controls must have an identifiable focus/active state that remains understandable without relying exclusively on color.

### 7.4 Disabled states

Disabled controls should be visibly distinct but should not be the only way important information is communicated.

---

## 8. Shared Theme, Language, Motion, and Spacing Boundaries

### 8.1 Theme

**Phase 1b:** Light theme only.

Dark mode is future work and requires a separate decision. This does not prohibit a future dark theme; it prevents unplanned theme scope from entering Phase 1b.

### 8.2 Language and direction

**Phase 1b:** Current product language and LTR presentation only.

Avoid unnecessary hard-coded directional assumptions where practical, but do not introduce an i18n/RTL implementation layer in Phase 1b.

Internationalization/localization/RTL support is future work requiring a separate decision.

### 8.3 Motion

Motion must be:
- minimal;
- purposeful;
- functional;
- short rather than prolonged;
- never the sole communication of state;
- compatible with reduced-motion considerations where implementation supports them.

Exact millisecond timings remain implementation-level decisions rather than a fixed design-system contract.

### 8.4 Spacing

Use a shared 4px-based spacing scale:

| Token | Value |
|---|---:|
| space-1 | 4px |
| space-2 | 8px |
| space-3 | 12px |
| space-4 | 16px |
| space-5 | 20px |
| space-6 | 24px |
| space-8 | 32px |
| space-10 | 40px |
| space-12 | 48px |

The scale establishes consistency without prescribing exact spacing for every screen.

---

## 9. Cross-Portal Consistency

The Driver, Company, and Reviewer portals are three role-specific experiences of the same Freight system.

Shared consistency includes:
- visual language;
- color semantics;
- component vocabulary;
- status communication rules;
- accessibility principles;
- interaction conventions;
- evidence/history presentation conventions where those concepts exist in the workflow.

Role-specific navigation, content, actions, terminology, and workflow order remain governed by the corresponding locked blueprint. Role-specific terminology is allowed where explicitly established; it must not change the underlying semantic meaning of a shared status.

---

## 10. Locked Blueprint Protection

This shared design-system record does **not** override or reopen locked Driver, Company, or Reviewer decisions.

If implementation reveals a genuine conflict with a locked blueprint:
1. stop;
2. record the conflict as evidence;
3. investigate separately;
4. do not silently modify the locked blueprint;
5. obtain an explicit decision before changing the boundary.

Shared visual consistency must never become a reason to change established workflow, authorization, lifecycle, evidence, marketplace, claim, or AI behavior.

---

## 11. What We Avoid

Freight Phase 1b should avoid:
- decorative complexity;
- excessive gradients or visual effects;
- excessive animation;
- color-only status communication;
- icon-only meaningful statuses;
- inconsistent status semantics between portals;
- unnecessary card/dashboard overload;
- dark-theme scope in Phase 1b;
- i18n/RTL implementation scope in Phase 1b;
- introducing new product behavior under the label of UI redesign;
- reopening locked portal workflows without evidence of a genuine conflict.

---

## 12. Decision Classification Summary

| Area | Classification | Status |
|---|---|---|
| Product/problem foundation | SOURCE-SUPPORTED | Locked |
| Shared UX principles | DESIGN INFERENCE / DESIGN DECISION | Locked |
| Visual language | DESIGN INFERENCE / DESIGN DECISION | Locked |
| Color values | PROPOSED DESIGN DECISION | Locked for Phase 1b |
| Semantic status categories | DESIGN DECISION | Locked for Phase 1b |
| Status communication rule | DESIGN DECISION | Locked for Phase 1b |
| Accessibility principles | DESIGN DECISION | Locked for Phase 1b |
| Light theme boundary | SCOPE DECISION | Locked for Phase 1b |
| Current language / LTR boundary | SCOPE DECISION | Locked for Phase 1b |
| Motion principle | DESIGN DECISION | Locked for Phase 1b |
| 4px spacing scale | DESIGN DECISION | Locked for Phase 1b |
| Typography specifics | IMPLEMENTATION-LEVEL / OPEN | Not a blocker to shared-system lock |
| Detailed icon vocabulary | IMPLEMENTATION-LEVEL / OPEN | Not a blocker to shared-system lock |
| Component-level contrast validation | IMPLEMENTATION-LEVEL / REQUIRED | Must occur during implementation-boundary review/build |
| Dark theme | FUTURE DECISION | Out of Phase 1b |
| i18n / localization / RTL | FUTURE DECISION | Out of Phase 1b |

---

## 13. Claude Independent Review — V2

The reconciled version was independently reviewed by Claude after the V1 review and reconciliation.

**V2 verdict:** Minor changes were identified, but no material conflict or blocker was found.

Confirmed by V2:
- Warning / Attention consolidation is logically sound;
- status-label rule is precise and implementation-agnostic;
- stated contrast calculations are correct for the documented white-background checks;
- Reviewer layout cross-check has no conflict with the locked Reviewer mental model/workflow;
- light-theme/LTR/motion/spacing boundaries are clean;
- no conflict was found with locked Driver, Company, or Reviewer blueprints;
- no unsupported or overreaching claims were found.

V2 identified only two non-blocking refinements:
1. clarify status-label wording versus shared semantic meaning;
2. explicitly carry soft-background contrast validation into implementation-level validation.

Both refinements are incorporated into this locked record.

V2 final recommendation: **SAFE TO LOCK; no remaining material conflict.**

---

## 14. Final Lock Decision

**FINAL DECISION: LOCKED**

Ayush explicitly authorized the final lock after reviewing the V2 Claude review and the final frontend UI output summary.

This lock means the shared Cross-Portal Design System is the authoritative shared frontend foundation for Phase 1b implementation.

It does **not** authorize implementation by itself. Implementation remains subject to the project's existing implementation-boundary gate and authorization process.

The locked foundation must be used together with the separately locked Driver, Company, and Reviewer blueprints.

---

## 15. Current Status

**Chat41 / Node7 / Phase1b — SHARED CROSS-PORTAL DESIGN SYSTEM: LOCKED**

Next architectural gate:
**Implementation-Boundary Review → Implementation Preparation → Authorized Build/Test**

No backend/product-behavior changes are implied by this lock.
