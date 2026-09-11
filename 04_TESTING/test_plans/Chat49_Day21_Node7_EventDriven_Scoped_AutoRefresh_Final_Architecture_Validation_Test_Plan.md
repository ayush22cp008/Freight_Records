# Chat49 — Day 21 — Node 7
# Event-Driven Scoped Auto-Refresh Final Architecture Validation Test Plan

**Status:** READY FOR EXECUTION
**Purpose:** Validate the remaining risks before final architecture approval
**Scope:** Investigation / validation only
**Implementation authorization:** NOT GRANTED

## 1. OBJECTIVE

Validate whether the proposed event-driven, resource-scoped auto-refresh architecture can satisfy the product requirement without fixed polling, global refreshes, cross-tenant leakage, missed-update persistence, or disruption of capture/local client state.

Target behavior:

```text
Relevant business event
→ identify resource
→ identify authorized audience
→ notify only interested connected client(s)
→ update/refresh only the relevant safe surface
```

## 2. AUTHORITATIVE INPUTS

Use the following Records as the investigation baseline:

- `05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Investigation.md`
- `05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Reinvestigation.md`
- `05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Reinvestigation_Report.md`
- `01_BRAIN_HANDOFFS/Claude/Chat49_Day21_Node7_Claude_Independent_Review_GlobalAutoRefresh.md`

## 3. TEST MATRIX

### T1 — Authorized Event Delivery / Tenant Isolation

Trigger a real relevant mutation, such as a trip claim, using authorized project flows.

Verify:

- intended connected client receives the event
- unrelated company/driver does not receive or react to it
- event payload does not expose unnecessary sensitive row data
- authoritative server fetch still enforces authorization/RLS

Record exact tested identities/scopes without exposing secrets.

Result must be VERIFIED / INFERRED / UNKNOWN.

### T2 — Exact Surface Scope

Open multiple connected surfaces, including relevant and unrelated pages.

Trigger an event affecting one resource.

Verify which exact route/component reacts.

Pass condition:

```text
Affected surface updates
Unrelated surfaces do not refresh/update
```

### T3 — Capture-State Protection

On representative `events/*` capture/submission pages:

- begin the real capture flow
- select a photo/file where applicable
- establish GPS/local state where applicable
- cause a relevant background event

Verify selected file, preview, input, GPS/local state, and in-progress submission are not lost or corrupted.

If the capture page is excluded from subscriptions, verify that no automatic refresh is triggered there.

### T4 — Duplicate / Burst Events

Cause or simulate repeated relevant mutations/events in a controlled test-safe manner.

Verify:

- refreshes are bounded/debounced where appropriate
- no runaway refresh loop occurs
- final UI converges to authoritative server state
- duplicate events do not create duplicate business mutations

Do not alter production business semantics merely to create this test.

### T5 — Missed Event / Reconnect Recovery

Test temporary network loss, subscription disconnect, or browser reconnect where practical.

Verify after reconnection/visibility regain:

```text
client reconnects
→ relevant surface becomes authoritative/current
```

A permanently stale UI requiring manual refresh is a failure against the target product requirement unless an explicit recovery mechanism is documented and accepted.

Mark unsupported claims UNKNOWN.

### T6 — Multiple Tabs

Use two tabs for the same resource and, where safe, an unrelated resource/page.

Verify:

- intended tabs react
- unrelated tabs do not react unnecessarily
- no race/flicker/inconsistent state appears
- subscription cleanup occurs after navigation/unmount

### T7 — Event Ordering / Rapid Mutations

Perform two or more valid state transitions close together where the current workflow permits it.

Verify the UI converges to the latest server-authoritative state even when notifications arrive close together.

### T8 — Security / RLS Reverification

Inspect the actual current project RLS policies, Realtime publication/subscription setup, and server authorization paths that would govern the proposed architecture.

Do not accept generic platform capability as proof.

Result must distinguish:

```text
VERIFIED — actual project configuration/evidence
INFERRED — reasoned but not directly verified
UNKNOWN — cannot establish safely
```

### T9 — Performance / Connection Characterization

Characterize, where practical, the difference between:

- no event / idle state
- event-driven active state
- burst of events
- multiple active tabs

Do not make unsupported claims of production-scale capacity. Mark extrapolations INFERRED.

## 4. TEST SAFETY

Do not change production schema, RLS, authentication, claim atomicity, evidence semantics, or locked portal UX merely to make a test pass.

Do not implement the proposed Realtime architecture as part of this validation unless a separate implementation authorization exists.

Use existing functionality, read-only inspection, logs, controlled test data, and reversible non-production-safe procedures as appropriate.

## 5. REQUIRED REPORT FORMAT

Produce a single test result report containing:

```text
Test ID
Preconditions
Execution
Observed result
Evidence
Status: PASS / FAIL / PARTIAL / NOT TESTABLE
Evidence quality: VERIFIED / INFERRED / UNKNOWN
Impact
```

Then provide:

```text
Overall architecture risk status
Remaining UNKNOWN items
Required mitigations
Final recommendation:
  APPROVE
  APPROVE WITH CONDITIONS
  REVISE
  REJECT
```

## 6. EXIT CRITERIA

Architecture validation is sufficient only when we have credible evidence for:

1. authorized event delivery and tenant isolation
2. exact surface scoping
3. capture/local-state protection
4. duplicate/burst behavior
5. reconnect/missed-event recovery
6. multiple-tab behavior
7. event-ordering convergence
8. actual RLS/security compatibility
9. practical MVP performance characteristics

Until these are sufficiently established, implementation remains unauthorized.
