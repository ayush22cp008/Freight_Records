# Freight — Shared Cross-Portal Design System Decisions

**Record:** Chat41 / Node7 / Phase1b
**Status:** RECONCILED REVIEW CANDIDATE — NOT YET LOCKED
**Purpose:** Establish one shared design foundation for Driver, Company, and Reviewer portals before implementation. This record consolidates the design reasoning already established in the current project context, Claude review findings, and the reconciled decisions from review.

---

## 1. Document Purpose

This record defines the proposed shared design foundation that should apply across the Freight website/application before implementation of the remaining Phase 1b work.

The purpose is not to redesign or reopen the locked Driver, Company, or Reviewer workflows. It is to establish a common visual and UX language for unspecified shared details so implementation does not produce three visually unrelated portal experiences.

This document remains a review candidate until the final lock gate is explicitly completed.

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
- review questions and finalization criteria.

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

**Why:** The project research establishes the evidentiary gap around freight detention and facility interactions as the motivating problem. The current system has evolved beyond the original MVP concept, but its evidence, identity, state, and role boundaries remain central to the current product model.

### 3.2 Core problem

**Decision:** The foundational problem is the gap between real-world freight/facility events and the ability of relevant parties to understand, verify, and rely on the resulting evidence.

**Why:** The original problem statement and feasibility research consistently emphasize documentation, burden of proof, dispute resistance, timestamps, and evidence rather than simple location tracking.

### 3.3 Core purpose

**Decision:** Freight should make relevant freight activity and evidence understandable, structured, and usable for the role that must act on it.

**Why:** The product's differentiating premise is evidence-centered accountability rather than generic tracking. The current portals express this through role-specific actions and evidence/state visibility.

### 3.4 Core value

**Decision:** The core value is trustworthy structured evidence and clear role-specific action around that evidence.

**Why:** The research identifies the evidentiary gap as the central unmet need, while the current system establishes identity, evidence, state, and responsibility boundaries.

### 3.5 Product character

**Decision:** Freight should feel:
- professional;
- trustworthy;
- operational;
- evidence-centered;
- precise;
- clear rather than decorative.

**Why:** This character follows from the product's responsibility for evidence, verification, operational state, and potentially disputed events. A playful, flashy, or highly decorative interface would weaken that communication.

---

## 4. Shared UX Principles

### Principle 1 — Evidence first

**Decision:** Important information should be presented around the underlying evidence and system record rather than decorative presentation.

**Why:** Evidence is the product's central trust mechanism.

**UI implication:** Evidence, event details, timestamps, identity/context, and history should remain legible and discoverable wherever the existing workflow exposes them.

**Prevents:** Making Freight look like a generic dashboard where evidence is secondary.

### Principle 2 — State clarity

**Decision:** Users should be able to understand the current state of an important object or workflow quickly.

**Why:** Driver, Company, and Reviewer workflows all depend on state and transitions.

**UI implication:** Current status should have strong hierarchy and consistent semantics.

**Prevents:** Users having to infer state from scattered fields or actions.

### Principle 3 — Action clarity

**Decision:** When the existing workflow requires an action, the next required action should be visually obvious.

**Why:** The Company model explicitly includes Needs Attention and Receiver Action Inbox concepts, while Reviewer is decision-driven and Driver workflows are operational.

**UI implication:** Primary actions should be visually distinct and placed near the state/context that explains why they are needed.

**Prevents:** Action hunting and ambiguous next steps.

### Principle 4 — Role clarity

**Decision:** Driver, Company, and Reviewer should clearly understand what they are responsible for.

**Why:** The three portals have intentionally different responsibility boundaries.

**UI implication:** Role-specific navigation, content, and actions may differ while the shared design language remains consistent.

**Prevents:** Cross-role confusion and accidental implication of permissions.

### Principle 5 — Timeline clarity

**Decision:** Chronological information should be easy to follow when the underlying workflow exposes a timeline/history.

**Why:** The Freight concept and current Company model place meaningful importance on delivery history, evidence, and event progression.

**UI implication:** Events should have clear ordering, labels, timestamps where available, and evidence/context where supported by the system.

**Prevents:** Chronological ambiguity.

### Principle 6 — Trust through transparency

**Decision:** Where the existing system supports it, important information and decisions should expose the evidence/context that supports them.

**Why:** Freight is evidence-centered and the Reviewer role is explicitly verification-first.

**UI implication:** Do not hide the basis of a state or verification decision when the current product provides that basis.

**Prevents:** Black-box-looking verification or unexplained states.

### Principle 7 — Consistency across portals

**Decision:** Shared components, terminology, status semantics, and interaction patterns should remain consistent across all three portals.

**Why:** They are role-specific experiences of one Freight system.

**Prevents:** Three visually and behaviorally unrelated products.

### Principle 8 — Operational simplicity

**Decision:** Avoid unnecessary decoration, excessive animation, and dashboard noise.

**Why:** Freight is an operational evidence product, not a consumer entertainment or marketing experience.

**Prevents:** Visual complexity that competes with operational information.

### Principle 9 — Responsive by design

**Decision:** Shared components and layouts should remain usable across the device sizes relevant to each role.

**Why:** Different Freight roles may interact with the system from different device contexts.

**Prevents:** Desktop-first layouts that fail when the available viewport changes.

### Principle 10 — Accessible by default

**Decision:** Meaning must not depend on color alone, and interactive states must remain understandable and usable.

**Why:** Evidence/status-heavy interfaces can otherwise become ambiguous or inaccessible.

**Prevents:** Accessibility failures and state ambiguity.

---

## 5. Shared Visual Language

### 5.1 Overall visual direction

**Decision:** Freight should use a clean, professional, evidence-centered, operational, restrained visual language.

**Why:** This directly supports the product character established above: trust, evidence, accountability, and clarity.

### 5.2 Visual personality

| Area | Proposed direction | Reason |
|---|---|---|
| Overall style | Modern professional / operational | Matches a serious operational product |
| Visual density | Moderate | Supports information-rich workflows without crowding |
| Decoration | Minimal | Keeps attention on evidence and actions |
| Shapes | Clean and restrained | Supports professional consistency |
| Cards | Used for meaningful information grouping | Avoids card overload |
| Borders | Subtle and functional | Separates content without noise |
| Shadows | Light / limited | Preserves hierarchy without visual excess |
| Icons | Simple and consistent | Supports semantic recognition |
| Typography | Legibility and hierarchy first | Evidence and operational text must remain readable |
| Status treatment | Strongly distinguishable and consistent | State is operationally important |
| Animation | Minimal and purposeful | Avoids distraction |
| Whitespace | Deliberate | Separates decisions and major information |

### 5.3 Layout philosophy

**Decision:** Shared page hierarchy should generally communicate:

**Context → Current State → Important Information → Required Action → Supporting Evidence/History**

**Why:** This order mirrors the user's need to understand what they are looking at, what is happening, what matters, what they need to do, and why.

This is a shared presentation principle only. Individual portal blueprints remain authoritative for their actual information architecture and workflow.

### Reviewer cross-check

The shared hierarchy was explicitly checked against the locked Reviewer Mental Model and locked Reviewer Interaction Mapping.

**Result: NO CONFLICT.**

The hierarchy is compatible with the Reviewer sequence because:
- Context corresponds to Applicant + Claimed Role;
- Current State corresponds to Pending Verification / decision state;
- Important Information centers the submitted Evidence;
- Required Action corresponds to Evaluate → Identity/Role Verified → Approve/Reject;
- Supporting Evidence/History corresponds to evidence viewing and completed Verification History.

The shared hierarchy therefore remains a general presentation principle and does not override the Reviewer verification-first workflow.

### 5.4 Shared component language

The three portals should share the fundamental visual vocabulary for:
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

**Important:** Shared components do not mean identical screens. Role-specific workflows determine what content/actions are present; the shared design system determines how those elements look and behave.

---

## 6. Shared Color System — Proposed

### 6.1 Color strategy

**Decision:** Use a restrained deep-blue primary identity supported by neutral surfaces and standardized semantic colors.

**Why:** Blue is proposed because it communicates stability, trust, and professionalism and provides a strong foundation for an evidence-centered operational product. The restrained approach avoids making Freight look like a generic bright-blue SaaS template.

**Source classification:** DESIGN INFERENCE / PROPOSED DESIGN DECISION. The project source material does not prescribe an official Freight color palette.

### 6.2 Proposed color tokens

| Token | Proposed HEX | Purpose |
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

### 6.3 Why these token categories exist

The palette is intentionally divided into:
1. brand/interaction colors;
2. neutral structural colors;
3. semantic status colors.

This prevents every screen from becoming saturated with color and ensures that semantic colors retain meaning.

### 6.4 Color hierarchy

**Decision:** Primary blue should not appear everywhere.

Normal UI should primarily consist of neutral backgrounds/surfaces and dark readable text, with primary blue reserved for meaningful interaction and selected states. Semantic colors should be reserved for actual semantic conditions.

**Why:** If every component is strongly colored, status loses meaning and operational hierarchy becomes harder to read.

### 6.5 Semantic consistency

The same semantic color meaning must apply across Driver, Company, and Reviewer.

The shared UI semantic categories are:
- **Neutral** → inactive/unspecified/non-semantic state;
- **Information** → informational content;
- **Warning / Attention** → requires awareness or action but is not an error/rejection;
- **Success** → verified/completed/accepted;
- **Error** → failed/rejected/invalid.

**Important distinction:** The UI semantic category **Warning / Attention** is separate from any project-monitor severity terminology. Existing Monitor WARNING severity, where applicable, is not changed by this design-system decision.

### 6.6 Status communication rule

**Decision:** Every meaningful operational status must have a visible text label. Color is supplementary, never the sole indicator. An icon may reinforce a status when useful, but an icon is not a replacement for the text label.

Requirements:
- Text status label: **REQUIRED**;
- Color: **SUPPLEMENTARY**;
- Icon: **OPTIONAL REINFORCEMENT**;
- Color-only status: **NOT ALLOWED**;
- Icon-only status for a meaningful state: **NOT ALLOWED**;
- Same semantic meaning across portals: **REQUIRED**.

Examples include `Verified`, `Warning / Attention`, and `Rejected`.

### 6.7 Color contrast validation

The proposed palette was checked against white for common normal-text usage during reconciliation:
- Primary `#1D4ED8` → 6.70:1, suitable for normal text;
- Success `#15803D` → 5.02:1, suitable for normal text;
- Error `#B91C1C` → 6.47:1, suitable for normal text;
- Information `#0369A1` → 5.93:1, suitable for normal text;
- Warning / Attention `#B45309` → 5.02:1, suitable for normal text;
- The former Attention `#D97706` → 3.19:1 and is therefore not retained as the normal-text semantic token;
- Disabled `#94A3B8` → 2.56:1 and is therefore restricted to appropriate disabled-state presentation rather than normal informational text.

Exact component-level combinations still require implementation-level validation because contrast depends on the actual foreground/background pairing and text size.

---

## 7. Accessibility Requirements

The following are shared requirements for the design system, subject to implementation-level validation.

### 7.1 Contrast

Exact color tokens must be checked against the actual foreground/background combinations in which they will be used. The palette should not be approved based only on visual preference.

### 7.2 State communication

Do not rely solely on:
- color;
- a color-only dot;
- a color-only border;
- color-only chart/status encoding;
- an icon without a meaningful text label for an important operational state.

Use visible text labels plus supplementary color and/or icon cues as appropriate.

### 7.3 Focus and interaction

Interactive controls must have an identifiable focus/active state that remains understandable without relying exclusively on color.

### 7.4 Disabled states

Disabled controls should be visibly distinct but should not be the only way important information is communicated. The disabled token is not intended for normal body/status text where its contrast would be insufficient.

### 7.5 Typography readability

Typography decisions must prioritize legibility of operational data, evidence details, timestamps, labels, and actions over stylistic novelty.

### 7.6 Motion accessibility

Motion must not be required to understand important state or workflow information. Where supported by the implementation environment, reduced-motion preferences should be respected.

---

## 8. Shared Theme, Language, Motion, and Spacing Boundaries

### 8.1 Theme

**Decision:** Phase 1b supports the **Light theme only**.

Dark mode is outside Phase 1b scope and requires a separate future design/architecture decision if introduced.

This is a scope boundary, not a prohibition against future dark-theme work.

### 8.2 Internationalization / RTL

**Decision:** Phase 1b supports the current product language and **LTR layout only**.

Internationalization, localization, language switching, and RTL support are outside Phase 1b scope and require a separate future decision.

Layouts should avoid unnecessary hard-coded directional assumptions where practical so future support is not needlessly obstructed, without adding an i18n/RTL implementation layer now.

### 8.3 Motion

**Decision:** Motion should be **minimal, purposeful, and functional**.

Use short transitions for interactions such as opening/closing panels, navigation changes, loading feedback, and meaningful state transitions where useful.

Do not use:
- decorative animation;
- excessive movement;
- long transitions that slow operational workflows;
- motion as the sole communication of important information.

Exact timing values remain an implementation-level choice within this principle rather than an architecture-level fixed millisecond contract.

### 8.4 Spacing

**Decision:** Use a shared **4px-based spacing scale** across Driver, Company, and Reviewer.

| Token | Size | Typical use |
|---|---:|---|
| `space-1` | 4px | Very small gaps |
| `space-2` | 8px | Icon/text and compact gaps |
| `space-3` | 12px | Small component spacing |
| `space-4` | 16px | Standard component spacing |
| `space-5` | 20px | Medium spacing |
| `space-6` | 24px | Card/section spacing |
| `space-8` | 32px | Major section spacing |
| `space-10` | 40px | Large separation |
| `space-12` | 48px | Page-level separation |

The scale establishes consistency without prescribing exact spacing for every individual screen. Responsive layouts may adapt spacing contextually by viewport and workflow.

---

## 9. Cross-Portal Consistency Rules

### Must remain shared

The following should be shared across Driver, Company, and Reviewer unless a documented accessibility or workflow reason requires otherwise:
- brand identity;
- base color tokens;
- semantic color meanings;
- typography system;
- spacing principles;
- button language;
- form control language;
- status semantics;
- common interaction states;
- general accessibility behavior;
- common visual treatment for evidence/history where the same component concept exists.

### May differ by role

The following may differ because the locked portal blueprints define different responsibilities:
- navigation items;
- information architecture;
- page content;
- primary actions;
- workflow sequencing;
- role-specific terminology where explicitly established;
- density appropriate to a particular role/task;
- visibility of role-specific evidence or controls.

### Core principle

**Same Freight system, different role experience.**

Role identity should primarily come from workflow, content, permissions, and responsibility—not from giving each portal a different brand/theme color.

---

## 10. Locked Blueprint Protection

This shared design package must not be used to reinterpret or override locked decisions.

The following remain authoritative:
- locked Driver blueprint;
- locked Company blueprint;
- locked Reviewer Mental Model;
- locked Reviewer Interaction Mapping and any later confirmed Reviewer blueprint decisions;
- existing APIs/data/business rules/lifecycle/evidence/authorization behavior.

If a proposed shared visual rule conflicts with a locked blueprint or established system behavior:

**STOP → identify the conflict → investigate → obtain an explicit decision.**

Do not silently modify the locked boundary.

---

## 11. What We Are Deliberately Avoiding

Freight should not be visually repositioned as:
- a flashy consumer app;
- a cryptocurrency/blockchain product;
- a generic AI dashboard;
- a dense legacy TMS;
- an analytics-heavy BI platform;
- an overly futuristic AI interface;
- a decorative marketing website.

**Why:** These directions would weaken the evidence-centered, operational, professional character supported by the product foundation.

---

## 12. Decision Classification Summary

| Decision area | Classification | Current state |
|---|---|---|
| Product/problem foundation | SOURCE-SUPPORTED | Established |
| Evidence-centered product character | DESIGN INFERENCE | Established for design work |
| Shared UX principles | DESIGN DECISION | Reconciled |
| Shared visual language | DESIGN DECISION | Reconciled |
| Shared page hierarchy | DESIGN DECISION | Reconciled; Reviewer cross-check passed |
| Blue primary direction | DESIGN INFERENCE + PROPOSED | Reconciled; remains design-system choice |
| Exact HEX tokens | PROPOSED | Contrast-validated for common white-background text use; component validation remains required |
| Semantic color meanings | DESIGN DECISION | Reconciled; Warning / Attention consolidated |
| Status communication rule | DESIGN REQUIREMENT | Reconciled |
| Accessibility rules | DESIGN REQUIREMENT | Reconciled; implementation validation remains required |
| Theme | SCOPE DECISION | Light only for Phase 1b |
| i18n / RTL | SCOPE DECISION | Out of scope for Phase 1b; current LTR only |
| Motion | DESIGN DECISION | Minimal, purposeful; reduced-motion consideration |
| Spacing scale | DESIGN DECISION | Reconciled; shared 4px-based scale |
| Cross-portal consistency rules | DESIGN DECISION | Reconciled |
| Typography | OPEN / IMPLEMENTATION DETAIL | Exact typography tokens still to be selected during implementation preparation |
| Component specifications | OPEN / IMPLEMENTATION DETAIL | To follow after foundation lock |

---

## 13. Claude Independent Review and Reconciliation

Claude independently reviewed the candidate shared design system and returned **APPROVE WITH CHANGES**.

The review findings were reconciled individually rather than applied blindly.

### Reconciled findings

1. **Contrast:** The candidate palette requires actual contrast validation. Common normal-text combinations were checked; the former Attention token was removed from normal semantic use, while Information was confirmed to pass normal-text contrast on white.
2. **Disabled state:** Disabled `#94A3B8` has low contrast and is restricted to appropriate disabled-state presentation rather than normal text.
3. **Attention vs Warning:** No Freight-specific evidence required two separate UI semantic categories, so they are consolidated as **Warning / Attention**. This does not change any separate project-monitor WARNING severity terminology.
4. **Reviewer layout:** The shared page hierarchy was cross-checked against the locked Reviewer Mental Model and Interaction Mapping and found to have **no substantive conflict**.
5. **Status communication:** Meaningful statuses require visible text labels; color is supplementary and icons are optional reinforcement, never replacements for status labels.
6. **Theme:** Light theme only for Phase 1b.
7. **i18n / RTL:** Current product language and LTR only for Phase 1b; future support requires a separate decision.
8. **Motion:** Minimal and purposeful, with reduced-motion consideration; exact timing remains implementation-level.
9. **Spacing:** Shared 4px-based spacing scale established without prescribing every screen's exact spacing.

### Review conclusion

The Claude review identified no substantive contradiction with the locked portal boundaries after reconciliation. The remaining implementation-level details do not block the shared foundation, provided they do not override locked workflow decisions.

---

## 14. Finalization Gate

This record should become the final locked shared design-system decision only after:

**REVIEW → FINDINGS → RECONCILIATION → AYUSH DECISION → FINAL LOCK**

The following stages are now complete:

**REVIEW → FINDINGS → RECONCILIATION → AYUSH DECISION**

Final lock remains a separate explicit project-control decision.

At finalization, unresolved implementation-level details must remain explicitly marked UNKNOWN/OPEN rather than being silently filled by assumption.

Only after this gate should the shared visual system be treated as authoritative input for implementation preparation.

---

## 15. Current Status

**Status:** RECONCILED REVIEW CANDIDATE — READY FOR EXPLICIT FINAL LOCK.

**Implementation status:** No implementation authorization is created by this document.

**Next architectural gate:** Explicitly lock the Shared Cross-Portal Design System, then use the locked foundation together with the locked Driver, Company, and Reviewer blueprints for implementation-boundary review and implementation preparation.
