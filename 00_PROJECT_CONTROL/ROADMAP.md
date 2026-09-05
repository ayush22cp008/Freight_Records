# ROADMAP.md

**Project:** Freight — AI Builders Hackathon  
**Hackathon window:** Aug 21 – Sep 15, 2026  
**Roadmap status:** ACTIVE EXECUTION ROADMAP — Node 7 is materially rephased by the approved Chat29 reassessment.  
**Current execution day:** Day 15  
**Current chat:** Chat39

## Active Roadmap — 7 Nodes

| Node | Work | Status |
|---|---|---|
| Node 1 | Product + Authorization Rework | 🔒 COMPLETE / LOCKED |
| Node 2 | Authentication + Identity | 🔒 COMPLETE / ACCEPTED |
| Node 3 | Company Trip Creation + Publishing | 🔒 COMPLETE / ACCEPTED |
| Node 4 | Driver Marketplace + Atomic Claim | 🔒 COMPLETE / ACCEPTED |
| Node 5 | Whole Delivery Tracking | 🔒 COMPLETE / ACCEPTED |
| Node 6 | Security + Evidence | 🔒 COMPLETE / ACCEPTED |
| Node 7 | AI + Final Integration + Demo | 🔵 ACTIVE |

Do not reopen Nodes 1–6 unless new evidence identifies a regression or a specific reviewer requirement.

## Node 7 — AI + Final Integration + Demo

**Status: 🔵 ACTIVE**

### Execution sequence

```text
Phase 1a
   ↓
Phase 1b
   ↓
Phase 3 (conditional)
   ↓
Final E2E + bugfix + demo + presentation
```

### Phase 1a — COMPLETE / ACCEPTED

Baseline AI evidence-grounded summary, timeline integration, and public shareable read-only evidence were completed and manually accepted.

### Phase 1b — Full 3-Portal UI/UX Redesign

**Status: 🔵 ACTIVE**

Scope:

1. Driver portal
2. Company portal
3. Reviewer portal

Phase 1b redesigns frontend structure, presentation, navigation, hierarchy, discoverability, responsiveness, and demo experience around existing capabilities. It does not introduce new product functionality.

## Driver Portal — Blueprint Complete / Locked

**Day 14 / Chat38 → 🟢 COMPLETE / LOCKED**

Authoritative record:

`00_PROJECT_CONTROL/Chat38_Day14_Node7_Phase1b_Frontend_Blueprint_Decisions.md`

## Company Portal — Blueprint Complete / Locked

**Day 15 / Chat39 → 🟢 COMPLETE / LOCKED**

The Company Portal blueprint was completed through:

```text
Existing Company Frontend Structure investigation
→ Company Mental Model
→ Company Interaction Mapping
→ Company Final Blueprint
→ Implementation-Boundary Review
```

Authoritative record:

`00_PROJECT_CONTROL/Chat39_Day15_Node7_Phase1b_Company_Blueprint_Decisions.md`

Locked decision counts:

```text
Company Mental Model        → 23
Interaction Mapping         → 20
Final Blueprint             → 10
Implementation Boundary    → 5
```

### Company final structure

```text
Company Portal
├── Dashboard
│   ├── Needs Attention
│   ├── Active Created Trips
│   └── Quick Access → My Created Trips / Incoming Deliveries
├── My Created Trips
├── Incoming Deliveries
│   └── Receiver Action Inbox
├── History / Timeline
└── Profile / Account
```

Core Company model:

```text
One Company
→ multiple trips
→ trip-specific Sender/Receiver relationship
→ shared core delivery visibility
→ relationship/state-based actions
```

The Company uses one unified portal. Sender and Receiver share core delivery-progress visibility, while available actions differ by relationship and trip state. Public Share remains Receiving Company-only.

Core interaction rule:

```text
My Created Trips
→ delivery-progress monitoring

Incoming Deliveries
→ pending Receiver-specific tasks

Receiver task completion
→ underlying delivery state advances
→ relevant Company views update consistently
```

Company Trip Detail uses:

```text
Current Status
→ Visual Delivery Progress
→ Next Required Action
→ Driver / Claim Information
→ Trip Details
→ Delivery Evidence
→ Timeline / History
```

### Company scope boundary

The Company redesign is frontend-focused: structure, presentation, navigation, hierarchy, discoverability, responsiveness, and verified UI/UX defect correction. Existing APIs/data, business rules, trip lifecycle, evidence rules, and authorization remain the source of truth.

No new backend business functionality, invented data, new authorization rules, new delivery stages, new evidence types, new marketplace behavior, new claim mechanisms, or new AI behavior may be introduced without separate verification and approval. Missing information must be treated as UNKNOWN and verified before scope expansion.

## Remaining Phase 1b Work

```text
Driver Portal blueprint       → 🟢 COMPLETE / LOCKED
Company Portal blueprint      → 🟢 COMPLETE / LOCKED
Reviewer Portal blueprint     → 🔵 NEXT
```

The next working step is **Reviewer Portal Blueprint**. Preserve the locked Driver and Company blueprints.

## Phase 3 — Conditional Add-On Features

**Status: ⏳ PENDING / CONDITIONAL**

Potential add-ons remain:

- AI inconsistency detection
- Confidence/completeness scoring
- Multi-signal evidence cross-check
- Natural-language Q&A over deterministic evidence

Do not begin Phase 3 before Phase 1b is complete and stable.

## Final Step — E2E + Demo + Presentation

**Status: ⏳ PENDING**

The final cycle happens once at the end:

```text
Full E2E across Driver / Company / Reviewer
→ Critical bug-fixing buffer
→ Realistic demo data/scenario
→ Presentation/demo flow
→ Final rehearsal
→ Node 7 final acceptance
```

## AI Boundary

AI may summarize, organize, or cross-check deterministic evidence but must not invent GPS, timestamps, event types, or unsupported blame/causality.

## Node 7 Acceptance Criteria

```text
[ ] Complete delivery scenario works from company creation to completion
[ ] Evidence timeline visible
[ ] AI summary generated from recorded evidence
[ ] Security regression passes
[ ] Critical bugs resolved
[ ] Demo can be repeated reliably
[ ] Presentation story is coherent
[ ] Ayush verification complete
```

## Subnode / Roadmap Governance

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode under current Node

Major blocker / architecture change
→ stop and reassess roadmap

3+ Subnodes under one Node
→ explicit roadmap reassessment
```

Do not silently deviate from the approved roadmap.

## Current Active Position

```text
Historical Core MVP                → IMPLEMENTED / VERIFIED
Node 1                              → COMPLETE / LOCKED
Node 2                              → COMPLETE / ACCEPTED
Node 3                              → COMPLETE / ACCEPTED
Node 4                              → COMPLETE / ACCEPTED
Node 5                              → COMPLETE / ACCEPTED
Dashboard follow-up                → CLOSED / VERIFIED
Historical AI follow-up            → CLOSED / VERIFIED
Node 6                              → COMPLETE / ACCEPTED
Node 7                              → ACTIVE
Phase 1a                            → COMPLETE / ACCEPTED
Phase 1b Driver Portal              → BLUEPRINT COMPLETE / LOCKED
Phase 1b Company Portal             → BLUEPRINT COMPLETE / LOCKED
Phase 1b Reviewer Portal            → NEXT
Phase 3                             → CONDITIONAL
Final E2E / Demo                    → PENDING
Day 15 / Chat39                     → CLOSED
```

## Working Method

```text
Observe
→ Investigate
→ Collect evidence
→ Determine root cause
→ Decide
→ Implement
→ Build/Test
→ Ayush manual verification
→ Record implementation report
→ Mark Node complete
```

Implementation prompts: `03_IMPLEMENTATION/prompts/`  
Implementation reports: `03_IMPLEMENTATION/implementation_reports/`  
Investigations: `05_DEBUGGING/investigations/`  
Architecture records: `02_ARCHITECTURE/`  
Project control: `00_PROJECT_CONTROL/`  
Checkpoints: `00_PROJECT_CONTROL/CHECKPOINTS/`

## Next Action

**Continue Node 7 Phase 1b with the Reviewer Portal Blueprint. Preserve the locked Driver and Company blueprints. Do not begin implementation until the required 3-portal blueprint/investigation sequence reaches implementation preparation.**
