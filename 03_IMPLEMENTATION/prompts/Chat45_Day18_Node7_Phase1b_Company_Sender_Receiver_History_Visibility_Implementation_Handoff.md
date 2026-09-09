# Chat45 Day18 — Node7 Phase1b Company Sender/Receiver History Visibility Implementation Handoff

## Status
READY FOR IMPLEMENTATION

## Purpose
Implement the remaining Company frontend visibility improvement so the existing sender/receiver participation already represented by current trip data is clearly visible to the Company user.

This is a **frontend-only Phase 1b change**. It must reuse existing data and existing Company capabilities. No backend, API, database, RLS, lifecycle, marketplace, completion, or security changes are authorized.

## Source Evidence / Context
The current system already tracks Company participation per trip:

- `company_id` identifies the creating/sending Company.
- `receiving_company_id` identifies the receiving Company.
- Company Recent Completed already includes trips where the current Company is either sender or receiver.
- Company History already includes completed trips where the current Company is either sender or receiver.
- Incoming Deliveries remains the receiver action inbox.
- My Created Trips remains the sender/creator surface.
- Unified Company Trip Detail already authorizes and displays the exact trip for either participating Company.

The current gap is **relationship visibility in the frontend**, not missing backend tracking.

## Required Scope

### 1. Recently Completed — relationship-aware wording
Update the Company dashboard's Recently Completed presentation so a completed trip clearly communicates whether the current Company participated as:

- **Sent** / sender
- **Received** / receiver

Use the existing `company_id` and `receiving_company_id` values already available from the trip query. If the IDs match the current Company, derive the appropriate relationship in the UI.

Do not introduce a new backend field or API.

The existing completion discovery and acknowledgement behavior must remain intact:

- existing recent-completion query behavior remains
- existing `View Completed Trip` CTA remains
- existing local acknowledgement behavior remains
- exact Trip Detail navigation remains unchanged

### 2. Company History — relationship labels
Update Company History so each completed trip visibly identifies the Company's relationship:

- **Sent** when current Company is the sender
- **Received** when current Company is the receiver

The label should be visible at the trip-card/list level without requiring the user to open Trip Detail.

### 3. Company History — relationship filter
Add a simple frontend filter with these states:

- **All**
- **Sent**
- **Received**

Default to **All**.

Filtering must operate on the already-fetched trip data in the UI. Do not add a new endpoint, query parameter, database field, or server-side lifecycle behavior.

### 4. Preserve existing navigation model
Do not change these responsibilities:

- **My Created Trips** = sender/creator active trip surface
- **Incoming Deliveries** = receiver action inbox
- **History** = company-wide completed participation history
- **Trip Detail** = unified exact-trip read/detail surface

Do not convert Incoming Deliveries into a history dashboard.

### 5. Preserve completion flow
Do not modify:

- Driver confirmation
- Receiver Company confirmation
- completion lifecycle semantics
- waiting/completion states
- local acknowledgement
- Recent Completion discovery logic
- History access rules
- exact trip-detail routing

## Protected Boundaries
The following remain protected and must not be changed for this task:

- APIs/contracts
- database/schema
- RLS/security
- auth/role rules
- trip lifecycle/state semantics
- claiming/marketplace behavior
- evidence requirements/types/integrity
- persistent review state
- backend behavior
- AI behavior
- Reviewer authority expansion
- Company completion API / C-05
- Reviewer R-03 protected boundary
- Reviewer R-05 dependency outside this narrow Company visibility task

## Implementation Guidance
Use the existing fetched fields rather than creating duplicate relationship logic in multiple places where practical.

Recommended frontend-derived semantics:

```text
if trip.company_id === currentCompanyId => Sent
else if trip.receiving_company_id === currentCompanyId => Received
else => Unknown / do not display an incorrect relationship
```

For this task, `Unknown` should not be silently converted into Sent or Received. If the required IDs are unexpectedly unavailable in a component, stop and report the gap instead of changing backend behavior.

## Required Data Selection Updates
The current Company dashboard Recent Completed query and Company History query must expose the existing relationship identifiers needed by the frontend:

- `company_id`
- `receiving_company_id`

This is only a change to the selected fields of the existing read query; it is **not** a schema/API change.

## UX Expectations
The result should make it immediately obvious that the same Company account can participate in different completed trips in different roles.

Example presentation concept:

```text
Recent Completed

Nagpur → Gandevi
Received
[View Completed Trip]

Mumbai → Pune
Sent
[View Completed Trip]
```

History concept:

```text
History    [All] [Sent] [Received]

Trip A     Sent
Trip B     Received
Trip C     Sent
```

Exact visual styling should continue using the existing locked Company/shared design system rather than introducing a new visual language.

## What Must NOT Change
Do not:

- add Accept/Reject delivery requests
- add new lifecycle states
- change marketplace visibility
- change driver claiming
- change completion authority
- change backend tables or migrations
- change RLS policies
- change APIs
- change Reviewer behavior
- change Company Trip Detail hierarchy
- change sender/receiver permissions

The separately approved Receiver Accept/Reject architecture review remains a future enhancement and is outside this implementation.

## Antigravity Preflight
Before editing source:

1. Read this handoff in full.
2. Inspect the current Company dashboard Recent Completed implementation.
3. Inspect `company/history` implementation.
4. Confirm exact existing field names for sender/receiver IDs.
5. Confirm no backend/API/DB/RLS change is required.
6. Confirm the change remains frontend-only within Phase 1b.

If any requirement cannot be satisfied with existing data/capabilities, stop and report it as UNKNOWN rather than inventing a new backend contract.

## Implementation
Make only the smallest source changes necessary to:

- expose `company_id` and `receiving_company_id` in the existing read selections
- derive/display Sent vs Received in Recently Completed
- derive/display Sent vs Received in History
- provide All/Sent/Received filtering in History

Keep TypeScript types correct and reuse existing Company/shared UI patterns.

## Validation Required
After implementation:

- run the project build/type validation used by the repository
- verify no unrelated routes/components are broken
- verify no backend/API/DB/RLS files were changed
- verify Recently Completed still opens the exact completed Trip Detail
- verify History still opens the exact completed Trip Detail
- verify filter states correctly show All, Sent-only, and Received-only
- verify a Company account acting as sender on one completed trip and receiver on another gets the correct labels
- verify the existing completion/acknowledgement behavior is unchanged

## Postflight Report
The implementation report must include:

- files changed
- exact UI behavior implemented
- validation/build result
- confirmation that no protected backend/data/security boundaries were modified
- any UNKNOWN or limitation discovered
- manual verification steps for Ayush

## Ayush Manual Verification Gate
Implementation is not accepted/locked until Ayush manually verifies in the browser:

1. A completed trip where the Company is the sender shows **Sent**.
2. A completed trip where the Company is the receiver shows **Received**.
3. Company History **All** shows both participation types.
4. Company History **Sent** shows only sender trips.
5. Company History **Received** shows only receiver trips.
6. Recently Completed relationship wording is correct.
7. Existing `View Completed Trip` navigation still reaches the exact Trip Detail.
8. No regression in Incoming Deliveries, My Created Trips, completion status, or acknowledgement flow.

Only after Ayush's manual verification should this Company visibility slice be considered accepted/locked.
