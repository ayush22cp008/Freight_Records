# Chat49 — Day 21 — Node 7
# Event-Driven, Resource-Scoped Auto-Refresh Architecture Reinvestigation

**Status:** 🔵 OPEN — INVESTIGATION / DESIGN ONLY
**Author:** ChatGPT
**Authority:** Reopened after review of the first event-driven investigation and the clarified product requirement

## 1. REASON FOR REOPENING

The first event-driven investigation correctly identified Supabase Realtime as technically feasible but concluded that the timer-based approach was simpler. That conclusion does not fully address the clarified product requirement:

```text
When a relevant business event occurs
→ automatically update only the connected portal/page/component affected by that event
→ do not refresh unrelated pages
→ do not require a fixed timer for normal synchronization
→ do not disturb capture/submission/local client state
```

Therefore the architecture question is reopened around **event → resource → audience → page/component** rather than timer-versus-Realtime complexity alone.

## 2. GOVERNANCE BOUNDARY

Driver, Company, and Reviewer portal baselines remain locked.

This reinvestigation does not authorize implementation and must not modify:

- portal UI/UX
- database schema or functions
- RLS/security policies
- authentication
- trip claiming/atomicity
- evidence persistence
- lifecycle semantics
- existing locked behavior

The earlier global 30-second timer is not approved by this record.

## 3. PRIMARY ARCHITECTURE QUESTION

Determine whether Freight can safely implement:

```text
BUSINESS EVENT
    ↓
identify exact resource(s)
    ↓
identify authorized audience
    ↓
notify only interested connected client(s)
    ↓
refresh/update only the affected read/status surface
```

The desired system must not behave as:

```text
ANY EVENT
    ↓
EVERY AUTHENTICATED PAGE REFRESHES
```

## 4. REQUIRED REINVESTIGATION

### A. Existing Business Event Map

Inspect the current source and records and identify actual mutation/event mechanisms for at least:

- trip publication / availability
- trip claim
- claim state change or loss
- pickup/load event
- delivery tracking events
- lifecycle status changes
- evidence additions
- reviewer decision/status changes
- other cross-portal mutations that materially affect UI

For every event record:

```text
Event name
Authoritative source
Triggering mutation/action
Resource identifier(s)
Producer
Potential consumers
Required data freshness behavior
```

Do not invent events that are not present in the current system; mark gaps UNKNOWN.

### B. Exact Scope Matrix

Build a concrete matrix:

```text
EVENT
→ RESOURCE
→ PORTAL / USER SCOPE
→ ROUTE
→ COMPONENT / SURFACE
→ REACTION TYPE
```

Possible reaction types:

```text
NO ACTION
CLIENT-ONLY UPDATE
CURRENT-ROUTE router.refresh()
TARGETED DATA REFETCH
```

Determine whether the correct behavior can be achieved without refreshing an entire portal or authenticated layout.

### C. Capture / Local-State Exclusion Matrix

Explicitly inspect every known `events/*` capture/submission surface and classify:

```text
SAFE
EXCLUDE
SPECIAL-HANDLE
```

Assess file/photo selection, previews, GPS state, form inputs, local status, optimistic state, modals, and in-progress submissions.

The architecture must not allow a background synchronization event to corrupt or unexpectedly reset an active capture workflow.

### D. Realtime Options

Compare only mechanisms that fit the actual source architecture:

- Supabase Postgres Changes
- Supabase Broadcast
- existing application/API event path, if one exists

For each, assess:

```text
Authorization
RLS interaction
Tenant isolation
Filtering
Payload sensitivity
Connection lifecycle
Reconnect behavior
Implementation complexity
Required DB/config changes
```

A client must never receive privileged information merely because it subscribes to a resource channel.

### E. Security Model

Prefer event payloads that contain minimal identifiers and cause the client to perform an authorized read rather than trusting event payload data as authority.

Verify how authorization is preserved for:

- company-to-trip isolation
- driver-to-trip isolation
- reviewer access boundaries
- resource/channel naming
- refresh/read operations after an event

Any unverified security property must remain UNKNOWN.

### F. Reliability

Determine how the design remains correct when:

- events are duplicated
- events are missed
- events arrive out of order
- the browser reconnects
- multiple tabs are connected
- a mutation happens immediately before or during refresh
- events arrive in bursts

The UI should converge toward server-authoritative state rather than depending on delivery of every notification.

### G. Visibility / Lifecycle

Assess whether subscriptions should pause or change behavior when the document is hidden, when the user navigates away, or when the network disconnects.

Avoid a design that recreates continuous work when the user cannot see the page.

### H. Performance Comparison

Compare the proposed scoped event-driven model against the global timer model on:

- idle DB query load
- active-tab behavior
- websocket/subscription overhead
- server component refresh volume
- event burst behavior
- MVP suitability

Use evidence where possible and mark estimates INFERRED.

## 5. REQUIRED ARCHITECTURE OPTIONS

The investigation must compare at least these options:

```text
Option A — Global fixed timer
Option B — Event-driven scoped router.refresh()
Option C — Event-driven targeted client/data update
Option D — Hybrid, only where justified
```

The recommendation must be based on the product requirement and safety evidence, not simply implementation simplicity.

## 6. REQUIRED OUTPUT

Produce a report with:

```text
OBSERVATION
INVESTIGATION
EVIDENCE
EVENT → RESOURCE → AUDIENCE → SURFACE MATRIX
ROUTE / CAPTURE SAFETY MATRIX
SECURITY ANALYSIS
RELIABILITY ANALYSIS
PERFORMANCE ANALYSIS
OPTIONS CONSIDERED
RECOMMENDED ARCHITECTURE
RISKS / TRADE-OFFS
REQUIRED VALIDATION TESTS
DECISION STATUS
```

Mark each material finding:

```text
VERIFIED
INFERRED
UNKNOWN
```

## 7. IMPLEMENTATION PROHIBITION

During this reinvestigation:

- do not add Supabase Realtime
- do not add polling or timers
- do not change source behavior
- do not modify database configuration
- do not change RLS
- do not modify locked portals

Only inspect, reason, document, and test in a non-production/destructive-free manner where appropriate.

## 8. SUCCESS CRITERIA

The reinvestigation is complete only when it can answer:

1. Can the user's event-driven scoped-refresh goal be achieved safely?
2. What exact events matter?
3. Which exact connected surfaces react to each event?
4. Which surfaces are explicitly excluded?
5. Can authorization/RLS be preserved?
6. What happens on missed/duplicate/delayed events and reconnect?
7. Is targeted client update preferable to router.refresh() for any surface?
8. What minimum implementation and validation scope is required?

## 9. CURRENT DECISION STATE

```text
Global 30s timer: NOT APPROVED AS TARGET ARCHITECTURE
Event-driven scoped refresh: PREFERRED INVESTIGATION DIRECTION
Final architecture: NOT YET DECIDED
Implementation authorization: NOT GRANTED
Portal baseline changes: NOT AUTHORIZED
```
