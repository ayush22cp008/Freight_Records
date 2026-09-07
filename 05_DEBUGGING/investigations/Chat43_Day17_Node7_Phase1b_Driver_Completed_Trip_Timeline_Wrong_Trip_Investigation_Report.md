# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Completed Trip Timeline Wrong-Trip Investigation Report

## 1. Investigation Status

**STATUS: INVESTIGATION OPEN — NO FIX AUTHORIZED YET**

**Portal:** Driver

**Phase:** Node 7 — Phase 1b — Driver UI/UX

**Scope:** Historical completed-trip navigation from `Completed Trips` → `View Timeline`.

**Purpose:** Determine why selecting `View Timeline` for a completed trip appears to open a Timeline for a different/current trip instead of the exact completed trip selected by the Driver.

---

## 2. New Manual Observation

Ayush manually inspected the Driver `Completed Trips` page and observed multiple completed-trip cards, each containing a `View Timeline` action.

When selecting a completed trip's `View Timeline`, the resulting Timeline displayed a trip identified as **`navsari`** rather than providing clear evidence that the exact completed trip selected from the history list was being loaded.

This indicates a potential historical-trip selection/navigation regression.

The issue is functionally different from the recently resolved responsive photo-layout issue and must be investigated separately.

---

## 3. Why This Requires Investigation

The existing Driver architecture explicitly requires completed historical trips to navigate using:

```text
/timeline?tripId=<selected-completed-trip-id>
```

The repository contains an earlier investigation establishing that the Timeline must consume the requested `tripId` and constrain the lookup to the authenticated Driver's ownership. fileciteturn280file0

The repository also contains an implementation report stating that exact historical-trip Timeline selection had previously been manually verified as functional. fileciteturn282file0

Therefore, the current observation may represent a regression, overwrite, stale deployment/source mismatch, or another navigation/data-selection problem.

We must determine the actual mechanism before changing code.

---

## 4. Expected Behavior

The required historical navigation flow is:

```text
Completed Trips
      ↓
View Timeline on selected trip A
      ↓
/timeline?tripId=<trip-A-id>
      ↓
Timeline reads requested tripId
      ↓
Exact trip A + authenticated driver ownership verified
      ↓
Trip A events displayed
```

For another completed trip B:

```text
View Timeline on trip B
      ↓
/timeline?tripId=<trip-B-id>
      ↓
Exact trip B events displayed
```

The Timeline must never silently substitute another trip belonging to the same Driver.

---

## 5. Current Evidence Classification

### VERIFIED

- Completed Trips page displays multiple completed trips.
- Completed-trip cards expose `View Timeline` actions.
- The current manual observation shows a Timeline for `navsari` after selecting a completed-trip Timeline action.
- Historical-trip Timeline navigation has previously been specified as `/timeline?tripId=<id>`. fileciteturn280file0
- Previous project records state that exact historical-trip Timeline selection had worked after implementation. fileciteturn282file0

### UNKNOWN — requires source/runtime verification

- Which completed-trip card was clicked for the current screenshot.
- The exact `tripId` in the resulting browser URL.
- Whether the `View Timeline` href currently contains the clicked trip's ID.
- Whether the deployed Timeline source currently reads `searchParams.tripId`.
- Whether the current application source has reverted to driver-only `.single()` trip lookup.
- Whether `navsari` is actually the selected completed trip or a substituted trip.
- Whether localhost and deployed/Vercel source are synchronized.

Do not infer these points solely from the screenshots.

---

## 6. Source-Level Investigation Required

Inspect the actual application source and runtime path for both sides of the navigation.

### A. Completed Trips source

Inspect:

```text
src/app/(authenticated)/driver/history/page.tsx
```

Verify:

- completed trips are queried correctly;
- each card uses its own `trip.id`;
- each `View Timeline` action generates exactly `/timeline?tripId=<that-card-id>`;
- no hard-coded trip ID exists;
- no fallback redirects to the current/active trip exist.

### B. Timeline source

Inspect:

```text
src/app/(authenticated)/timeline/page.tsx
```

Verify:

- `searchParams` is accepted/read;
- `tripId` is extracted;
- the exact requested trip is selected;
- selection is constrained by authenticated Driver ownership;
- completed trips remain eligible for historical Timeline display;
- there is no driver-only `.single()` fallback that can select an arbitrary trip.

### C. Deployment/source consistency

Verify that the application being manually tested is running the same source/commit represented by the current implementation records.

The current screenshots include both a deployed Vercel URL in earlier testing and localhost in the latest laptop screenshots. This must be distinguished before concluding that source behavior is broken.

---

## 7. Security Boundary

Historical Timeline selection must remain tenant-safe.

The requested `tripId` must never be treated as sufficient authorization.

Required conceptual lookup:

```text
trip.id = requested tripId
AND
trip.driver_id = authenticated driver.id
```

Do not weaken or bypass authenticated ownership checks to make historical navigation work.

The earlier investigation explicitly requires this ownership constraint. fileciteturn280file0

---

## 8. Possible Root-Cause Classes

These are hypotheses only and must be verified:

1. **Dashboard/history link regression** — wrong trip ID is attached to a card.
2. **Timeline selection regression** — Timeline ignores `tripId` and selects another driver trip.
3. **Fallback behavior** — missing/invalid `tripId` causes a current-trip fallback.
4. **Deployment mismatch** — local source differs from deployed source being tested.
5. **Stale build/cache** — browser/deployment is serving older code.
6. **Data identification confusion** — `navsari` may actually correspond to the selected historical trip.

No root cause is declared until evidence identifies one.

---

## 9. Non-Goals / Protected Areas

Do **NOT** modify:

- database schema;
- RLS policies;
- authentication model;
- Node 4 claim behavior;
- Node 5 lifecycle semantics;
- event vocabulary;
- event insertion APIs;
- evidence/photo storage;
- AI Evidence Summary logic;
- responsive CSS;
- Timeline visual redesign;
- Completed Trips visual redesign;
- Company portal;
- Reviewer portal.

Do not create a workaround by changing the Dashboard to point every completed trip at the current trip.

---

## 10. Required Diagnostic Test Matrix

| Test | Required evidence |
|---|---|
| Completed trip A → View Timeline | URL contains A's ID; Timeline shows A |
| Completed trip B → View Timeline | URL contains B's ID; Timeline shows B |
| Refresh historical Timeline | Same selected trip remains loaded |
| Direct `/timeline?tripId=A` | A loads when owned by authenticated Driver |
| Direct `/timeline?tripId=B` | B loads when owned by authenticated Driver |
| Invalid/non-owned tripId | Must not expose another Driver's trip |
| Active/current Timeline | Existing behavior remains functional |
| Localhost vs deployed | Source/commit consistency established |

---

## 11. Decision Rule

### If the source correctly uses exact `tripId` and ownership

Do not change the source yet. Investigate deployment/runtime/data mismatch and provide evidence.

### If history links generate the wrong ID

Create a narrowly scoped history-link implementation fix.

### If Timeline ignores `tripId`

Create a narrowly scoped Timeline historical-selection fix using exact `tripId` + authenticated Driver ownership.

### If an unauthorized fallback exists

Remove only the fallback responsible for substituting another trip, while preserving the ownership boundary.

### If multiple causes exist

Separate them into independently scoped fixes; do not combine unrelated changes without a new decision.

---

## 12. Implementation Authorization

**NO IMPLEMENTATION IS AUTHORIZED BY THIS INVESTIGATION.**

First establish:

1. the clicked completed trip ID;
2. the URL generated by its `View Timeline` button;
3. the Timeline's actual trip-selection query;
4. the authenticated ownership constraint;
5. the deployed/local source version.

Only after those are verified should a separate implementation prompt be created.

---

## 13. Investigation Conclusion

**Current observation:** A completed-trip `View Timeline` action appears to show the wrong/current trip (`navsari`).

**Known architecture requirement:** Historical Timeline navigation is trip-specific and must use the selected `tripId` plus authenticated Driver ownership. fileciteturn280file0

**Historical project evidence:** Exact completed-trip Timeline selection was previously reported as implemented and manually verified. fileciteturn282file0

**Current root cause:** UNKNOWN.

**Decision:** Investigate the history link, Timeline `tripId` consumption, ownership query, and source/deployment consistency before making any fix.

**Next state:** `INVESTIGATION → ROOT CAUSE DETERMINATION → DECISION → IMPLEMENTATION (if required)`
