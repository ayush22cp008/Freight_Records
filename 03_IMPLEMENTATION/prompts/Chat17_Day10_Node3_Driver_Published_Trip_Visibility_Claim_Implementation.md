# Chat 17 — Day 10 — Node 3 Driver Published Trip Visibility + Claim Implementation

## Objective
Implement only the missing Node 3 flow:

**Published trip → eligible driver visibility → driver claim → driver assignment**

Do not redesign unrelated authentication, reviewer, company publishing, or password-recovery functionality.

## Confirmed Current State
- Company can create a trip as `draft`.
- Company can publish the trip successfully.
- The publish API changes the trip status to `published`.
- Node 3 trip schema supports `draft`, `published`, `claimed`, `in_progress`, and `completed`.
- The current driver dashboard only looks for trips where `driver_id = current_driver_id` and `status = active`.
- Therefore a newly published trip (`status = published`, `driver_id = NULL`) is not visible to drivers.
- Reviewer routing has already been fixed so reviewer authorization takes priority over normal Driver/Company identity routing. Preserve that behavior.

## Required Behavior

### 1. Driver Dashboard — Available Published Trips
For a verified Driver who does not currently have a claimed/active trip:
- Query published trips that are eligible for that driver according to existing Node 3 rules.
- Display the available trip details using the existing Freight UI style:
  - pickup/facility
  - destination
  - distance
  - duration
  - payout
  - receiving company when available
  - current status
- Provide a clear `Claim Trip` action.

Do not remove the existing active/claimed trip workflow.

### 2. Server-Side Claim Endpoint
Create or update a server-side endpoint for claiming a trip.

The server must:
- authenticate the current Supabase user;
- resolve the current driver using `auth_id`;
- verify the driver is verified and is a Driver;
- load the target trip server-side;
- verify the trip is currently `published`;
- verify eligibility according to existing Node 3 rules;
- never trust a browser-supplied `driver_id`;
- atomically assign the trip to the first valid claimant;
- set `driver_id` to the current driver;
- transition `status` from `published` to `claimed`;
- return clear success/error responses.

### 3. Atomic First-Valid Claim
This is a core Node 4-compatible requirement and must be safe under concurrent requests.

Do not implement an unsafe sequence such as:
`SELECT trip → client/server delay → UPDATE`.

Use an atomic conditional update or database RPC/function so only one driver can transition the trip from `published` to `claimed`.

A second concurrent claimant must receive a conflict/already-claimed result and must not overwrite the winning driver.

### 4. Post-Claim Driver Experience
After successful claim:
- refresh/revalidate the dashboard;
- the claimed trip must appear as belonging to that driver;
- the same trip must no longer appear in the available published list;
- preserve the existing arrival/check-in/departure workflow;
- do not regress historical `active` trip behavior if other existing code still relies on it.

Determine the smallest correct transition from `claimed` into the current driver workflow from the existing codebase. Do not blindly replace all `active` handling.

## Security Requirements
A Driver must not be able to:
- claim a `draft` trip;
- claim a non-published trip;
- claim an already claimed trip;
- claim a completed trip;
- assign another driver's ID;
- bypass verification/role checks;
- bypass server-side eligibility rules through manipulated browser payloads.

The Company and Reviewer authorization behavior must remain unchanged.

## Company Regression Requirements
Preserve the already-working company flow:

`Create New Trip → Review Trip (Draft) → Publish Trip → Trip Published Successfully`

Do not modify the publish authorization logic except where absolutely necessary to integrate the missing driver visibility/claim flow.

## Reviewer Regression Requirements
Preserve reviewer-first routing and reviewer authorization.

A reviewer authorized through `reviewer_authorizations` must still be routed to:

`/reviewer/queue`

even if that Auth user also has another Freight identity.

## Database Requirements
Inspect the existing Node 3 migration:

`src/db/migrations/006_node3_trip_schema.sql`

It already supports these lifecycle values:
- `draft`
- `published`
- `claimed`
- `in_progress`
- `completed`

Do not create duplicate schema.

If an RPC/database function is required for the atomic claim, create a separate properly documented migration under the existing migrations structure. Ensure RLS/security behavior is correct.

## Testing Requirements
Perform and report these tests:

### A. Company publishing
1. Company logs in.
2. Create a new trip.
3. Confirm DRAFT state.
4. Publish it.
5. Confirm the published-success page.

### B. Driver visibility
1. Log in as a verified Driver.
2. Open the dashboard.
3. Confirm the newly published trip is visible as an available trip.

### C. Driver claim
1. Click `Claim Trip`.
2. Confirm the claim succeeds.
3. Confirm the trip now belongs to the claiming driver.
4. Confirm it is no longer shown as available.

### D. Competing claim
Use a second eligible Driver account or an equivalent concurrent test.
- Confirm only the first valid claim succeeds.
- Confirm the second claimant cannot overwrite the winning driver.

### E. Post-claim lifecycle
Confirm the claimed driver can continue through the existing driver workflow without regression.

### F. Regression
Confirm:
- reviewer still routes to `/reviewer/queue`;
- company still reaches the company dashboard;
- existing authentication and password-recovery behavior remains unchanged;
- existing active-trip behavior still works where applicable.

## Validation
Run the available project checks after implementation:
- lint
- typecheck
- build
- tests

Fix implementation-caused errors before reporting completion.

## Scope Control
Do not modify unrelated Node 1/Node 2 behavior.
Do not modify password recovery.
Do not modify reviewer authorization architecture.
Do not redesign the company trip creation/publishing UI.
Do not implement Node 4 marketplace features beyond the minimum atomic claim behavior required to connect Node 3 publication to driver claiming.

## Required Implementation Report
After implementation, create an implementation report under:

`03_IMPLEMENTATION/implementation_reports/`

The report must include:
- exact files changed;
- database migration/RPC changes, if any;
- implementation summary;
- security considerations;
- lint/typecheck/build/test results;
- manual test results;
- commit SHA;
- pushed branch/repository confirmation.

Do not claim manual verification unless it was actually performed and evidenced.
