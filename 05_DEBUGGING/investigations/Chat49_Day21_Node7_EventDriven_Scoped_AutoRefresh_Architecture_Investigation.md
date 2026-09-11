# Chat49 — Day 21 — Node 7
# Event-Driven, Resource-Scoped Auto-Refresh Architecture Investigation

**Status:** 🔵 OPEN — INVESTIGATION / DESIGN ONLY
**Author:** ChatGPT
**Authority:** New investigation following independent review of the proposed global timer approach

## 1. PURPOSE

Evaluate whether Freight should replace fixed-interval global polling with an event-driven, resource-scoped automatic refresh architecture.

The intended product behavior is:

```text
Relevant business event occurs
→ only affected connected portal/page(s) update automatically
→ unrelated pages do not refresh
→ no fixed polling timer is required for normal synchronization
→ user should not need manual refresh for relevant state changes
```

This investigation does **not** authorize implementation and must not modify the locked Driver, Company, or Reviewer portal baseline.

## 2. GOVERNANCE CONTEXT

The Driver, Company, and Reviewer portals are locked baselines. The previously proposed global `GlobalAutoRefresh` timer in the authenticated layout was independently reviewed and found too blunt because it would repeatedly re-execute the current route and shared layout data for every active authenticated tab.

The existing 30-second timer verification result is therefore not an approval of this new architecture.

Authoritative prior review/report references:

- `01_BRAIN_HANDOFFS/Claude/Chat49_Day21_Node7_Claude_Independent_Review_GlobalAutoRefresh.md`
- `04_TESTING/test_plans/Chat49_Day21_Node7_Global_AutoRefresh_Risk_Verification_Test_Plan.md`
- `04_TESTING/test_results/Chat49_Day21_Node7_Global_AutoRefresh_Risk_Verification_Test_Result.md`

## 3. ARCHITECTURE HYPOTHESIS

Investigate an event-driven model using Supabase Realtime and/or an equivalent existing project mechanism, with explicit scoping by business event, resource, portal, route, and component.

The target pattern is:

```text
BUSINESS EVENT
      ↓
identify affected resource(s)
      ↓
publish/receive scoped event
      ↓
only interested connected client(s) react
      ↓
refresh/update only affected read/status surface(s)
```

Do **not** assume that every database mutation should trigger a UI refresh.

## 4. REQUIRED INVESTIGATION QUESTIONS

### A. Business Event Inventory

Identify the meaningful cross-portal events already present in the current system, especially:

- trip published / becomes available
- trip claimed by a driver
- trip claim state changes / claim loss
- pickup/load event recorded
- delivery-tracking event recorded
- trip status/lifecycle transition
- evidence/event added
- reviewer decision / review status change
- any other mutation that materially affects a connected portal surface

For each event, determine:

- authoritative source of truth
- event trigger location/mechanism
- resource identifier(s), such as `trip_id` or event id
- which portal(s) care
- which exact route(s) or component(s) care
- whether a UI refresh is actually required
- whether client-local state must remain untouched

### B. Current Route / UI Classification

Inspect the current application and classify relevant routes/components as:

```text
READ-ONLY / SAFE FOR EVENT-DRIVEN REFRESH
INTERACTIVE BUT REFRESH-SAFE
CAPTURE / FILE / PHOTO / GPS / SUBMISSION — EXCLUDE
LOCAL OPTIMISTIC / TRANSIENT STATE — EXCLUDE OR SPECIAL-HANDLE
```

At minimum, explicitly assess the known event-capture surfaces under `events/*`, plus dashboard, trip list/detail, and reviewer queue/detail surfaces.

### C. Realtime Mechanism

Determine whether the current Supabase setup can support the required event-driven behavior safely.

Compare, as appropriate:

- Postgres Changes
- Supabase Broadcast
- existing application/API event mechanisms

Do not select a mechanism solely because it is easy to add. Assess security, RLS/authorization interaction, filtering, tenancy/isolation, reliability, and operational complexity.

### D. Event Scope and Audience

Define a mapping model such as:

```text
TRIP_CLAIMED(trip_id)
→ Company trip list/status surfaces for trip_id
→ Company trip detail for trip_id
→ Driver surface for trip_id
→ Reviewer surfaces only when relevant
```

The investigation must explicitly avoid:

```text
Any event
→ refresh every authenticated page
```

### E. Client-State Safety

Determine how event-driven refresh interacts with:

- form inputs
- selected files / image previews
- open modals
- local status state
- optimistic state
- scroll position
- tab visibility
- multiple browser tabs
- reconnect after temporary network loss

Any state-loss risk must remain UNKNOWN unless actually verified.

### F. Security / Authorization

Verify that an event cannot expose cross-company or cross-driver information merely because a client is connected to a channel.

Explicitly assess:

- channel naming/isolation
- trip/resource filtering
- RLS interaction
- authorization checks on refresh/read
- whether event payloads contain sensitive data
- whether events should contain only identifiers and cause the client to re-fetch authorized data

### G. Reliability / Failure Modes

Investigate:

- duplicate events
- missed events
- out-of-order events
- reconnect behavior
- stale clients
- simultaneous mutation + refresh
- event storms / bursts
- multiple tabs listening to the same resource

The architecture should remain correct even if a refresh notification is missed, delayed, duplicated, or received after the underlying mutation has already changed again.

### H. Performance

Compare the event-driven approach against the rejected/revised global 30-second timer in terms of:

- database query volume
- active-tab scaling
- realtime connection overhead
- server component refresh frequency
- burst behavior
- hackathon/MVP suitability

No unsupported claim such as “safe at scale” may be made without evidence.

## 5. REQUIRED OUTPUT

Produce an investigation report that contains:

```text
OBSERVATION
→ INVESTIGATION
→ EVIDENCE
→ ROOT CAUSE / DESIGN CONSTRAINTS
→ OPTIONS CONSIDERED
→ RECOMMENDED ARCHITECTURE
→ RISKS / TRADE-OFFS
→ REQUIRED TESTS
→ DECISION STATUS
```

Every material finding must be marked:

```text
VERIFIED
INFERRED
UNKNOWN
```

## 6. IMPLEMENTATION BOUNDARY

This investigation is design/review only.

Do NOT:

- implement Supabase Realtime
- add timers or polling
- alter portal UI/UX
- alter database schemas/functions
- modify RLS/security rules
- change authentication
- change claim atomicity
- change evidence persistence
- change lifecycle semantics
- modify locked portal baselines

Any concrete implementation must be separately authorized through the project workflow after this investigation reaches a decision.

## 7. SUCCESS CRITERIA

The investigation succeeds only if it can answer, with evidence where possible:

1. Can Freight achieve automatic relevant-page synchronization without a fixed timer?
2. Which business events should cause refresh/update?
3. Which exact portal surfaces should react to each event?
4. Which surfaces must never be automatically refreshed during active capture/submission?
5. Which Realtime mechanism is appropriate, if any?
6. How is authorization preserved for every event-driven update?
7. What happens when events are missed, duplicated, delayed, or received after reconnect?
8. What tests are required before implementation approval?

## 8. DECISION STATE

```text
Architecture proposal: EVENT-DRIVEN / RESOURCE-SCOPED AUTO-REFRESH
Current status: UNDER INVESTIGATION
Implementation authorization: NOT GRANTED
Portal baseline changes: NOT AUTHORIZED
```
