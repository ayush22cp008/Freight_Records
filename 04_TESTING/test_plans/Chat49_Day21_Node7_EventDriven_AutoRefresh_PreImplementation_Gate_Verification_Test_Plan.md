# Chat49 — Day 21 — Node 7

## Event-Driven Scoped Auto-Refresh — Pre-Implementation Gate Verification Test Plan

**Status:** OPEN — PRE-IMPLEMENTATION VERIFICATION ONLY
**Implementation authorization:** NOT GRANTED
**Scope:** Close the remaining evidence gaps identified by the independent compatibility reviews before any implementation prompt authorizes code changes.

---

## 1. Objective

Verify whether the proposed event-driven, resource-scoped auto-refresh architecture is safe and actually applicable to the current Freight system before implementation.

The target behavior remains:

```text
Relevant business event
→ scoped Realtime notification
→ only relevant safe read surface reacts
→ debounced router.refresh()
→ existing Server Component fetch becomes authoritative UI
```

The verification must prove that unrelated pages and in-progress capture state are not disturbed.

---

## 2. Mandatory Gates

### Gate A — Cloud Realtime Readiness

Determine from the current deployed/target Supabase environment whether:

- Realtime is enabled.
- `supabase_realtime` publication contains every table actually required by the final event/resource matrix.
- The target environment is the same environment used by the current deployed Freight application.
- No implementation is assumed merely because local `supabase/config.toml` enables Realtime.

**Evidence labels:** VERIFIED / UNKNOWN.

### Gate B — Realtime Security / RLS

Inspect the actual current migrations/policies and, where possible, verify live behavior for authenticated users.

Confirm:

- Which authenticated-role SELECT policies apply to `trips`, `events`, and any other subscribed tables.
- Whether Realtime delivery is appropriately constrained by those policies.
- Whether company, driver, and reviewer tenant/role boundaries prevent unauthorized resource visibility.
- Server-side `router.refresh()` remains authoritative and does not trust Realtime payload data.

Do not mark this gate VERIFIED from documentation alone.

### Gate C — Dependency / Browser Client Readiness

Verify the runtime dependency status of `@supabase/supabase-js` and identify the correct browser client pattern for Realtime without disturbing the existing SSR authentication/session model.

No dependency change should be made during this verification.

### Gate D — Safe-Surface / Capture Exclusion

Verify the route/component tree and establish the exact subscriber allowlist and hard exclusion list.

Hard exclusions include:

- `src/app/(authenticated)/events/*`
- onboarding multi-step/in-progress capture surfaces
- receiver-checkin and completion capture surfaces
- any surface whose primary purpose is active file/GPS/form submission

No subscriber may be placed in `src/app/(authenticated)/layout.tsx`.

### Gate E — Claim UI Safety

Verify the interaction between an event-driven `router.refresh()` and `ClaimTripButton.tsx`, especially its local `isClaimed` state.

Determine whether the proposed scoped refresh can cause the optimistic/success state to be lost, remounted unexpectedly, or conflict with claim completion.

Use source inspection plus a minimal behavioral test where available.

### Gate F — Reconnect / Visibility / Burst Safety

Verify the design requirements for:

- duplicate events
- burst events
- missed events
- reconnect/re-subscribe
- browser tab visibility returning to foreground
- component unmount cleanup

The architecture must converge by refetching authoritative server state; Realtime payloads must remain notification-only.

### Gate G — No Unrelated Refresh

Demonstrate the intended scoping for at least these representative cases:

1. Driver claims Trip A → relevant company surface for Trip A updates; unrelated driver/company/reviewer surfaces do not refresh merely because Trip A changed.
2. Lifecycle evidence event for Trip A → Trip A observers refresh; unrelated trip surfaces do not.
3. Event/capture work in `events/load` → no automatic subscriber-induced refresh interrupts local File/GPS state.

---

## 3. Required Evidence Output

Produce a verification report containing:

| Gate | Result | Evidence | Remaining uncertainty |
|------|--------|----------|-----------------------|
| A | PASS / FAIL / UNKNOWN | exact source/config/live evidence | explicit |
| B | PASS / FAIL / UNKNOWN | exact policy/query/runtime evidence | explicit |
| C | PASS / FAIL / UNKNOWN | package/client evidence | explicit |
| D | PASS / FAIL / UNKNOWN | exact routes/components | explicit |
| E | PASS / FAIL / UNKNOWN | source + behavioral evidence | explicit |
| F | PASS / FAIL / UNKNOWN | source/test evidence | explicit |
| G | PASS / FAIL / UNKNOWN | scope evidence/test evidence | explicit |

Every claim must be labeled VERIFIED, INFERRED, or UNKNOWN.

Do not convert inferred behavior into VERIFIED merely because it is consistent with framework or Supabase documentation.

---

## 4. Implementation Decision Rule

Implementation may be authorized only if:

- no mandatory gate is FAIL;
- all security/publication facts needed for implementation are VERIFIED or have an explicitly accepted, bounded uncertainty with a documented reason;
- capture exclusions are explicit and testable;
- ClaimTripButton safety is verified;
- reconnect/visibility behavior is defined;
- exact event → resource → audience → surface mapping is frozen for the implementation scope.

Otherwise return **BLOCKED — MORE EVIDENCE REQUIRED**.

---

## 5. Non-Goals

This plan does not authorize:

- implementing Realtime subscribers;
- changing business logic;
- changing claim atomicity;
- changing lifecycle semantics;
- changing RLS write paths;
- converting portals to Client Components;
- adding a polling timer;
- adding unrelated UX changes.
