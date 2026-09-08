# Chat44 — Day 18 — Node 7 — Phase 1b — Company Trip Detail Evidence Implementation Prompt

## 0. Status

**FRONTEND-ONLY IMPLEMENTATION SPECIFICATION — READY FOR AYUSH AUTHORIZATION**

This prompt is narrowly scoped to the verified Company Trip Detail delivery-evidence presentation gap.

It does **not** authorize source-code execution by itself. Antigravity must wait for explicit Ayush authorization before modifying source code.

---

## 1. Evidence-Based Decision

The completed investigation established:

- Company Unified Trip Detail already retrieves the trip and related events through the existing query using `events (*)`.
- `events (*)` already includes the existing `photo_url` evidence field.
- Both sending and receiving Company relationships are already authorized through the existing Company relationship checks.
- The current Company Trip Detail Event Timeline renders event type and timestamp but does not render `evt.photo_url`.
- The established evidence model is `events.photo_url`.
- No API, database, RLS, authentication, authorization, evidence-model, storage, or business-rule change is required.
- The evidence gap is therefore **VERIFIED FRONTEND-ONLY**.

Governing investigation report:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Trip_Detail_Evidence_Visibility_Investigation_Report.md`

---

## 2. Implementation Objective

Add **Delivery Evidence** presentation to the existing Company Unified Trip Detail using the evidence data already available in the page.

The objective is only to make existing authorized evidence visible in the Company Portal.

### Core rule

> **Consume and present existing `events.photo_url` data. Do not change how evidence is created, stored, authorized, persisted, or interpreted.**

---

## 3. Exact Scope

### Allowed

Modify only the Company frontend presentation necessary to:

1. Detect an event with a non-empty `photo_url`.
2. Render that existing evidence in the Company Trip Detail.
3. Place the evidence within the locked **Delivery Evidence** portion of Trip Detail.
4. Keep evidence associated with its corresponding event where practical.
5. Provide an accessible visual presentation of the existing photo URL.
6. Present an appropriate empty state when no evidence photo exists.
7. Preserve the existing Event Timeline and all current trip/state information.
8. Maintain responsive presentation on phone, tablet, and desktop.

A small Company-specific presentation component may be created if that is the safest implementation.

### Not allowed

Do not:

- Create a new evidence upload flow.
- Create a new evidence table/model.
- Create a new API endpoint.
- Modify an API contract or response shape.
- Modify `events` schema.
- Modify `events.photo_url` semantics.
- Modify RLS.
- Modify authentication.
- Modify authorization policy logic.
- Modify trip lifecycle/state semantics.
- Modify claiming behavior.
- Modify Receiver Check-in/Completion behavior.
- Modify Public Share exposure.
- Modify Driver evidence behavior.
- Modify Reviewer behavior.
- Add new business rules.

---

## 4. Required Source Target

Primary target:

`src/app/(authenticated)/company/trips/[id]/page.tsx`

The investigation identified this page as the existing Company Unified Trip Detail implementation.

The existing data retrieval must remain functionally unchanged:

```tsx
.from('trips')
.select(`*, events (*)`)
```

Do not replace this with a new API or backend data path merely to implement the presentation.

If the source has changed since the investigation and the existing evidence field is no longer available, **STOP and report the discrepancy** rather than inventing a new data path.

---

## 5. Required UX Result

Within Company Trip Detail, the locked hierarchy already requires:

**Current Status → Visual Delivery Progress → Next Required Action → Driver / Claim Information → Trip Details → Delivery Evidence → Timeline / History**

Add the Delivery Evidence presentation without disrupting this hierarchy.

### Evidence with photo

When an event has a valid non-empty `photo_url`:

- Show that evidence as an image or equivalent accessible visual.
- Keep it clearly identified as delivery evidence.
- Associate it with the relevant event when the existing event context is available.
- Do not expose unrelated/private data.
- Do not change the event's meaning or state.

### No evidence

When no event has a usable `photo_url`:

- Show a concise, truthful empty state such as **No delivery evidence available**.
- Do not imply that evidence is required at a point where the existing system does not say so.
- Do not invent evidence records.

### Broken/unusable photo URL

If the existing URL fails to load:

- Present a safe frontend error/fallback state.
- Do not alter or regenerate the stored URL.
- Do not create a replacement upload mechanism.

---

## 6. Authorization Preservation

The Company Trip Detail already operates inside the authenticated Company route and uses the existing sender/receiver relationship checks.

Preserve this exact security boundary.

The implementation must not:

- Broaden Company trip access.
- Fetch evidence for trips outside the existing authorized trip context.
- Add public evidence access.
- Modify Public Share.
- Bypass server-side authorization.

The fact that `photo_url` exists does not authorize access to arbitrary trips. Only render evidence attached to the already authorized Trip Detail data.

---

## 7. Public Share Boundary

Do not modify Public Share.

Evidence rendering is for the authenticated Company Trip Detail only.

Do not add `photo_url` to any public verification/public-share response or projection.

The existing receiving-company-only Public Share behavior must remain unchanged.

---

## 8. Driver Protection

The Driver Portal already has evidence presentation using the established `events.photo_url` model.

Do not modify Driver code for this Company fix unless a shared component is absolutely required.

Prefer Company-specific composition when it avoids unnecessary Driver regression risk.

If a shared component must be changed:

1. Identify all affected portals.
2. Confirm the change is presentation-safe.
3. Preserve the locked Driver behavior.
4. Test the affected Driver surface sufficiently to establish non-regression.

If impact cannot be bounded, **STOP**.

---

## 9. Protected C-05 Boundary

Do not modify:

`src/app/api/completion/route.ts`

This evidence presentation task is independent of the protected Receiver Completion response-shape issue.

Do not combine the two fixes.

---

## 10. Implementation Procedure

### Step 1 — Preflight

Inspect the current Company Trip Detail source before editing.

Confirm:

- Existing trip query.
- Existing `events (*)` retrieval.
- Existing `sortedEvents` or equivalent event collection.
- Existing Event Timeline rendering.
- Existing Company authorization context.

### Step 2 — Add presentation

Use the already retrieved event data.

For each relevant event with a non-empty `photo_url`, render the evidence within the Delivery Evidence area.

Avoid duplicate evidence presentation if an existing Company component already provides equivalent rendering.

### Step 3 — Accessibility

Ensure the rendered evidence has meaningful accessible text/alternative text based on available event context.

Do not invent sensitive metadata solely for accessibility.

### Step 4 — Responsive behavior

Verify that evidence presentation:

- Fits phone width.
- Fits tablet width.
- Fits desktop width.
- Does not introduce ordinary horizontal page scrolling.
- Does not break the existing Trip Detail hierarchy.

### Step 5 — Preserve existing behavior

Confirm no changes to:

- Trip state.
- Event semantics.
- Receiver actions.
- Driver/claim behavior.
- Public Share.
- APIs/backend/security.

---

## 11. Stop Conditions

Stop immediately and report if:

1. `photo_url` is no longer available from the existing Company Trip Detail data.
2. A new API is required.
3. The API response must change.
4. RLS/security must change.
5. Authorization behavior must change.
6. Evidence storage must change.
7. A new evidence model is required.
8. Public Share must change.
9. Driver behavior must change in an unbounded way.
10. A lifecycle/business-rule change appears necessary.
11. The locked Company Blueprint would need reinterpretation.
12. The source behavior contradicts the investigation report.

Never work around these conditions by assumption.

---

## 12. Build / Test Requirements

After the source change, run the project's appropriate existing frontend validation commands.

At minimum, record the applicable:

- Build result.
- Type-check/lint result if configured.
- Relevant test result if configured.

Also verify the Company Trip Detail behavior for:

### Case A — Evidence exists

A trip with an event containing `photo_url` displays the evidence.

### Case B — No evidence

A trip without `photo_url` displays the truthful empty state and does not fail.

### Case C — Multiple evidence photos

If multiple events contain `photo_url`, the UI presents them without incorrectly collapsing them into a single unrelated item.

### Case D — Sender Company

An authorized sending Company sees relevant evidence attached to its authorized Trip Detail.

### Case E — Receiving Company

An authorized receiving Company sees relevant evidence attached to its authorized Trip Detail.

### Case F — Responsive

Evidence presentation remains usable on phone, tablet, and desktop.

### Case G — Existing Trip Detail

Current status, progress, next action, driver/claim information, trip details, and timeline remain intact.

---

## 13. Required Implementation Report

Create/update:

`03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Trip_Detail_Evidence_Implementation_Report.md`

The report must state:

- Exact files changed.
- Exact evidence presentation added.
- Existing data path reused.
- Whether `events.photo_url` was used directly.
- Build/test results.
- Responsive evidence.
- Sender/receiver verification evidence where available.
- Public Share unchanged.
- Backend/API/DB/RLS/auth unchanged.
- Driver impact/non-regression evidence.
- Any UNKNOWNs or blockers.

Use explicit:

- **VERIFIED**
- **INFERRED**
- **UNKNOWN**

Do not claim browser/manual verification unless actually performed.

---

## 14. Final Implementation State

After source implementation and automated/build validation:

**FRONTEND FIX COMPLETE — AWAITING AYUSH MANUAL VERIFICATION**

Do not declare the Company Portal accepted or locked.

Do not begin Reviewer implementation because of this fix.

Wait for Ayush's manual browser verification and explicit acceptance.

---

## 15. Authorization

**Current state: READY FOR AYUSH IMPLEMENTATION AUTHORIZATION.**

This is a narrowly scoped implementation prompt derived from the completed investigation.

Source execution must begin only after Ayush explicitly authorizes implementation.
