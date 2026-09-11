# Chat49 — Day 21 — Node 7
## Cross-Portal Global Auto-Refresh Investigation

**Status:** OPEN — investigation/design only
**Context:** Follow-up to the completed dashboard auto-refresh investigation.

## User Requirement

The target experience is not limited to dashboards. During normal use, the entire Freight website should update relevant visible application state automatically so the user does not need to perform a manual browser refresh at any point.

## Investigation Objective

Determine the safest project-wide data-refresh strategy for the Freight website while preserving all locked product, authorization, security, persistence, lifecycle, evidence, backend, and AI boundaries.

## Required Scope

Inspect all relevant dynamic application surfaces across the locked Driver, Company, and Reviewer portals, including but not limited to:

- dashboards and queues
- marketplace and trip lists
- trip detail/status views
- requests/applications and receiver flows
- tracking and delivery lifecycle views
- history views
- evidence/review state
- badges, counts, or other dynamic indicators
- any other authenticated page where backend state can change while the page remains open

Do not assume every route requires polling. Classify each relevant surface according to its existing data-fetch/navigation architecture and the refresh behavior actually required.

## Required Investigation Questions

1. What dynamic data surfaces exist across the website?
2. How does each surface currently obtain and refresh data?
3. Which surfaces remain stale while the page is open after a backend state change?
4. What refresh mechanisms already exist anywhere in the source?
5. Can a common refresh strategy preserve the existing Server Component architecture where applicable?
6. Where would `router.refresh()` be sufficient, and where would another mechanism be required?
7. Would periodic polling be appropriate globally, selectively, or not at all?
8. Are there places where event-driven/realtime updates are materially safer or more appropriate than polling?
9. What are the performance, duplicate-request, race-condition, navigation, loading-state, and background-tab risks?
10. Can the desired no-manual-refresh experience be achieved without changing API contracts, RLS/security, authorization, persistence, business rules, lifecycle semantics, evidence semantics, or locked portal product behavior?
11. What is the minimum-change architecture that provides the broadest correct automatic-update coverage?

## Governance

This record is investigation/design only. Do not create an implementation instruction from this record until the investigation reaches an evidence-backed decision and Ayush explicitly approves the resulting implementation direction.

Any defect or incompatibility discovered during this investigation must be recorded separately and must not be silently fixed as part of the investigation.

## Required Investigation Pipeline

OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION

Claims must be tagged VERIFIED / INFERRED / UNKNOWN according to the project evidence rules.

## Expected Outcome

Produce an evidence-backed project-wide recommendation defining:

- the target automatic-refresh behavior across the whole website;
- which routes/surfaces need automatic updates;
- which mechanism should be used for each class of surface;
- whether a shared refresh component/utility is appropriate;
- the default refresh cadence, where polling is justified;
- safeguards for inactive/background tabs and duplicate requests;
- and the exact protected boundaries that must remain unchanged.

**Chat:** Chat49
**Day:** Day21
**Node:** Node7
