# Chat49 — Day 21 — Node 7
## Global Auto-Refresh Risk Verification Test Plan

**Status:** READY FOR EXECUTION
**Purpose:** Verify Claude's identified risks before final architecture approval.

## Scope

Validate the proposed automatic-refresh behavior against the current implementation without modifying source code.

## Required Checks

1. Claim flow: confirm whether a periodic `router.refresh()` can reset or otherwise disturb the post-claim `ClaimTripButton` client state.
2. Event capture flow: confirm whether a periodic refresh can interrupt or lose selected photo/file state in the capture/submission components.
3. Reviewer evidence: confirm behavior of existing signed evidence URLs when a page remains open and refresh occurs after URL expiry.
4. Multiple tabs: observe request multiplication and any visible refresh/flicker/race behavior with two authenticated tabs.
5. Background tab: background a tab for at least five minutes, then foreground it and observe refresh behavior, request bursts, and resulting UI state.
6. Action refresh overlap: verify behavior when an existing action-triggered `router.refresh()` overlaps or occurs near a periodic refresh tick.
7. Route classification: identify the current authenticated routes that are safe read/list/status surfaces versus interaction/capture/submission surfaces that should not receive a blanket timer.
8. Load characterization: identify the relevant layout/page queries and provide a reasoned estimate of added request/query volume under representative concurrent-tab counts.

## Evidence Rules

For each check record:

- observation;
- evidence (logs, network output, screenshots, source inspection, or reproducible behavior);
- result: VERIFIED / INFERRED / UNKNOWN;
- impact on the proposed architecture.

Do not fix defects discovered during this test. Record them separately and return them for governance.

## Exit Condition

The test plan is complete only when the evidence is sufficient to decide whether the proposed auto-refresh architecture must be revised, and exactly where safe automatic refresh can be applied without disrupting existing behavior.

**Project:** Freight — AI Builders Hackathon
**Node:** Node 7
**Day:** 21
**Chat:** Chat49
