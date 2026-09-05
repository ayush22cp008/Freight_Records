# Chat39 Day15 — Node 7 Phase 1b Company Portal Blueprint Decisions

## Purpose

This record captures the Company Portal decisions locked during Chat39 / Day15 after the Existing Company Frontend Structure investigation, Company Mental Model, Company Interaction Mapping, Final Blueprint, and Implementation-Boundary Review.

## Evidence Basis

The Company blueprint is grounded in the Chat39 Company frontend/API/data-dependency investigations under:

`05_DEBUGGING/investigations/`

Key verified findings include:
- `companies` is a generic Company entity; Sender/Receiver relationship is trip-specific through `trips.company_id` and `trips.receiving_company_id`.
- The existing Company frontend has Dashboard, Create Trip, Incoming Deliveries, Receiver Check-in, Receiver Completion, Completed Deliveries, and Public Share management/viewing.
- Sender visibility has a current structural gap: the Company Dashboard primarily queries receiving-company trips, creating a post-publish Sender Black Hole.
- Shared Navbar exposes a Company-inaccessible `/timeline` path.
- Company Profile/Account is not currently a dedicated portal area.
- Company Timeline/History is not currently a dedicated portal area.
- Receiver Completion has a verified frontend response-shape defect: the client expects `data.state` while the API returns `{ success: true }`.
- Mobile navigation has a verified structural gap, and Create Trip has a verified small-screen layout weakness.
- Public Share is currently authorized for the Receiving Company only and exposes a controlled public projection.

These findings remain the source-of-truth boundary for the redesign. No unverified data, business rules, or authorization behavior is invented here.

# Part 1 — Company Mental Model — COMPLETE / LOCKED

1. **Company identity** — A Company is a business participant that can act as either the Sending Company or Receiving Company depending on the trip.
2. **Primary purpose** — The Company manages and monitors its trips and deliveries.
3. **Shared delivery visibility** — Sender and Receiver have the same core delivery-progress visibility; actions differ by relationship/state.
4. **Driver information** — When a driver claims a trip, both participating companies can see the claim state and appropriate basic driver information, subject to existing authorization/data support.
5. **Shared lifecycle** — Both Sender and Receiver see the same core trip lifecycle/status progression from the same underlying source of truth.
6. **Unified dashboard** — One Company Dashboard contains the Company's relevant work, clearly separating Created/Sent Trips from Incoming/Receiving responsibilities.
7. **Context-based actions** — Only actions authorized for the Company, its trip relationship, and current trip state are shown.
8. **Public Share** — Receiving Company only can create/revoke Public Share; both companies retain internal delivery-progress visibility.
9. **Completed history** — Both participating companies can access relevant completed trips/history.
10. **Evidence** — Both participating companies can see relevant delivery evidence/proof within existing authorization/data boundaries.
11. **Profile/Account** — Company has a dedicated Profile/Account area for existing company/account information and basic controls.
12. **History/Timeline** — Company has a dedicated History/Timeline area for past trips and delivery events.
13. **Sender/Receiver distinction** — Unified portal uses clear Created/Sent Trips and Incoming Deliveries sections rather than separate Sender/Receiver portals.
14. **Unified Trip Detail** — Each trip has one unified Trip Detail showing the complete delivery picture, with actions varying by relationship/state.
15. **Operational priority** — Dashboard prioritizes active/current work; completed work moves to History/Completed.
16. **Unclaimed state** — A published trip without a driver clearly shows a Waiting for Driver state.
17. **Claimed state** — Once claimed, the Company sees driver/basic driver information, current progress, and the next relevant stage.
18. **Next action** — Company sees a clear next required action whenever an action is actually required from that Company.
19. **Completed Trip Detail** — Completed trips remain available through full read-only Trip Detail.
20. **Attention state** — Company sees an attention state when it actually needs to act; normal progress is not treated as an alert.
21. **Visual progress** — Delivery progress is shown as a simple visual lifecycle; detailed history remains in Timeline.
22. **Global navigation** — Company has one stable navigation structure: Dashboard, My Created Trips, Incoming Deliveries, History/Timeline, Profile/Account.
23. **Company-first experience** — The portal is designed around the Company itself; each trip determines Sender/Receiver relationship, visibility, and actions.

## Part 2 — Company Interaction Mapping — COMPLETE / LOCKED

1. Dashboard first answers: **What needs my attention?**
2. Selecting an attention item opens the relevant Trip Detail directly.
3. Dashboard provides two clear entry points: **My Created Trips** and **Incoming Deliveries**.
4. My Created Trips shows current status, driver/claim status, and delivery progress as the primary trip snapshot.
5. Incoming Deliveries primarily shows delivery/task state and the next required Receiver action.
6. Trip Detail opens with Current Delivery State → Visual Progress → Next Required Action, followed by details.
7. Driver claim/basic driver information is shown in Trip Detail and summarized in relevant trip lists.
8. Receiver Check-in appears directly in Trip Detail when required.
9. Receiver Completion appears directly in Trip Detail when required.
10. Public Share management is accessed from the relevant Trip Detail and remains Receiving Company-only.
11. Completed trips are accessed through History/Timeline and open read-only Trip Detail.
12. Company can search and use basic operational filters across trips, such as status, date, and Sender/Receiver relationship.
13. Clicking a trip/card from Dashboard, Created Trips, Incoming Deliveries, or History opens the same unified Trip Detail.
14. Returning from Trip Detail preserves the originating list and applicable search/filter context where practical.
15. Receiver-action state is state-driven and reflected consistently in My Created Trips and Incoming Deliveries. My Created Trips communicates progress and provides a shortcut; Incoming Deliveries contains the actual pending task. Direct entry to Incoming Deliveries is always valid. Completion removes/updates the pending task and advances delivery progress.
16. Incoming Deliveries contains only pending Receiver-specific tasks; with none pending it shows a clear empty state such as **No actions required**.
17. Successful Receiver task completion immediately confirms success, reflects the resulting state, and provides a path to updated Trip Detail.
18. Sender and Receiver use the same core Trip Detail structure; relationship-specific information/actions vary by authorization/state.
19. Multiple pending Company actions are presented as a prioritized attention list using existing state/timing information where available.
20. Receiver task completion immediately updates the underlying delivery state and all relevant Company views consistently.

## Part 3 — Final Company Frontend Blueprint — COMPLETE / LOCKED

### 3.1 Primary Navigation

```text
Dashboard
My Created Trips
Incoming Deliveries
History / Timeline
Profile / Account
```

### 3.2 Dashboard

```text
1. Needs Attention
2. Active Created Trips
3. Quick Access → My Created Trips / Incoming Deliveries
```

- Needs Attention is a concise summary of Company actions that require attention.
- With no pending actions, it remains visible with a positive empty state such as **No actions needed**.
- Active Created Trips shows operational snapshots.
- Incoming Deliveries is not duplicated as a dashboard task list; it remains the Receiver Action Inbox.

### 3.3 My Created Trips

Each trip card presents an operational snapshot:

```text
Trip identity
→ Current status
→ Delivery progress
→ Driver / claim status
→ Receiver-action indicator when applicable
→ Next relevant action
```

If a Receiving Company has a pending manual action, the relevant trip can communicate that state and provide a direct shortcut to the specific Receiver task. This shortcut is navigation only; task state is not created by the shortcut.

### 3.4 Incoming Deliveries

Incoming Deliveries is a **Receiver Action Inbox**, not a second delivery-progress dashboard.

Each pending task card presents:

```text
Trip identity
→ Specific Receiver task
→ Why/action is needed
→ Relevant current delivery state
→ Perform Action
```

A Receiver may open this area directly or arrive through a My Created Trips shortcut. Both paths expose the same state-driven pending task.

After successful completion:
- the pending task disappears or updates because no outstanding Receiver action remains;
- the underlying delivery state advances;
- a success confirmation is shown;
- a path back to the updated Trip Detail is provided.

When no Receiver action is pending, show a clear **No actions required** empty state.

### 3.5 Unified Trip Detail

Final hierarchy:

```text
1. Current Status
2. Visual Delivery Progress
3. Next Required Action
4. Driver / Claim Information
5. Trip Details
6. Delivery Evidence
7. Timeline / History
```

Sender and Receiver use the same core structure. Relationship-specific actions appear only when authorized and appropriate for the current trip state.

### 3.6 History / Timeline

- Company-wide history lists past trips in which the Company participated.
- Opening a trip leads to read-only Trip Detail.
- The detailed event Timeline belongs inside the relevant Trip Detail.
- Completed trips are review-only; no operational actions are exposed.

### 3.7 Profile / Account

- Dedicated Company Profile/Account destination.
- Shows existing Company/account information and basic controls.
- No new Company-management subsystem is introduced by the blueprint.

### 3.8 Public Share

- Receiving Company only can create/revoke Public Share.
- Public Share is accessed from the relevant Trip Detail.
- Existing controlled public projection and authorization remain authoritative.

### 3.9 Responsive Behavior

- One responsive Company Portal across phone, tablet/intermediate, and laptop/desktop.
- Same information hierarchy, workflow, destinations, and existing capabilities across viewports.
- Navigation, cards, forms, Trip Detail, actions, and Timeline adapt to available width.
- No normal portal workflow requires horizontal scrolling.
- Existing mobile Navbar and Create Trip structural weaknesses are eligible for frontend correction.

## Part 4 — Implementation-Boundary Review — COMPLETE / LOCKED

1. **No new backend business functionality** — Company Phase 1b is a frontend redesign of existing capabilities; verified UI defects may be corrected.
2. **Reuse existing APIs/data** — Existing endpoints, database relationships, trip states, events, evidence, and authorization are reused wherever they already provide required information.
3. **Preserve authorization/security** — Existing server-side authorization remains the security boundary; the UI controls discoverability/presentation only.
4. **Verified UI defects only** — Fix verified UI/UX defects such as the Receiver Completion response-shape issue; do not expand into unrelated improvements or new business capability.
5. **Missing information requires verification** — If the UI needs information the current system does not expose, stop and verify it before scope expansion; treat it as UNKNOWN rather than inventing data/API/business rules.

## Final Company Blueprint Principles

```text
One Company
→ many trips
→ trip-specific Sender/Receiver relationship
→ shared core delivery visibility
→ relationship/state-based actions

My Created Trips
→ delivery-progress monitoring

Incoming Deliveries
→ Receiver-specific pending action inbox

Trip Detail
→ single complete delivery picture

History / Timeline
→ completed/past review

Profile / Account
→ existing Company/account information

Public Share
→ Receiving Company only
```

## Phase 1b Scope Boundary

This blueprint changes frontend structure, presentation, navigation, hierarchy, discoverability, responsiveness, and verified UI defects around existing capabilities. It does not introduce new backend business functionality, new authorization rules, new delivery stages, new evidence types, new marketplace behavior, new claim mechanisms, or new AI behavior.

## Status

**Company Portal Blueprint → COMPLETE / LOCKED**

The Company blueprint is ready for the next project-control step: formal blueprint closure/checkpoint and subsequent implementation preparation. No implementation should begin until the required implementation handoff is explicitly prepared and authorized.
