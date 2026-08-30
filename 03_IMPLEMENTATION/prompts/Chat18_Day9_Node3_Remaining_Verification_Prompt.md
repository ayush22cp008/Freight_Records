# Chat18 — Day 9 — Node 3 Remaining Verification Prompt

## Purpose

Complete the remaining **verification-only** gates for Node 3 — Company Trip Creation + Publishing.

Node 3 implementation is already deployed and Ayush has manually verified the previously failing **Claimed → Start Arrival → Arrival Recorded** transition. Do not redesign or extend Node 3 functionality unless a test exposes a real defect.

## Authoritative Records

Use these Records as the source of truth before execution:

```text
00_PROJECT_CONTROL/CURRENT_STATUS.md
03_IMPLEMENTATION/plans/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan.md
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Report.md
03_IMPLEMENTATION/implementation_reports/NODE_3_CLAIMED_TO_START_ARRIVAL_FIX_Report.md
```

Application repository:

```text
ayush22cp008/freight_hackathon
branch: main
```

## Critical Rule

This task is **verification first**.

Do not make speculative code changes merely to make a test pass. If a genuine defect is found:

```text
1. Stop.
2. Record the exact failure.
3. Identify whether it is a Node 3 defect or an unrelated pre-existing issue.
4. Fix only if the defect is clearly within the approved Node 3 scope.
5. Re-run the affected verification.
6. Record the change and evidence.
```

Do not reopen Node 1 or Node 2 decisions.

## Gate 1 — Inspect Existing Test/Build Configuration

Before running commands, inspect:

```text
package.json
existing test configuration
existing lint configuration
existing TypeScript configuration
existing CI/workflow configuration
```

Use the repository's actual configured commands. Do not invent a test framework or fabricate test results.

## Gate 2 — Targeted Node 3 Security/Behavior Verification

Verify the following behaviors against the actual implementation:

### Authentication / authorization

```text
[ ] Unauthenticated Company trip creation is rejected.
[ ] Non-Company users cannot create trips.
[ ] Unverified Company users cannot create/publish trips where the active Node 2 gate requires VERIFIED state.
[ ] Creator Company is derived from the authenticated server-side identity.
[ ] Client-supplied creator/company ownership values cannot override server-side identity.
```

### Trip creation

```text
[ ] Valid Company can create a trip.
[ ] Required fields are validated server-side.
[ ] Invalid/missing receiving_company_id is rejected.
[ ] Created trip is associated with the authenticated creator Company.
[ ] Created trip has driver_id = null before driver claim/assignment.
[ ] Created trip starts in draft/pre-publication state.
```

### Receiving Company lookup

```text
[ ] Lookup requires authenticated/authorized access.
[ ] Lookup requires the appropriate verified Company role.
[ ] Lookup returns only the minimum intended fields (id/name or the exact implemented minimal shape).
[ ] No broad Company profile fields are exposed by the endpoint.
[ ] Existing general Company RLS was not weakened merely to support lookup.
```

### Publishing

```text
[ ] Verified Company can publish its own draft trip.
[ ] Another Company cannot publish the first Company's trip.
[ ] Non-publishable trip state cannot be arbitrarily published.
[ ] Client-supplied owner/company identifiers are not trusted as authorization.
[ ] Publishing performs the intended draft → published transition only.
```

### Direct cross-company access / IDOR

Verify that Company B cannot use a direct trip ID to perform unauthorized protected operations against Company A's trip.

At minimum verify the relevant create/publish/read authorization paths that exist in the current Node 3 implementation.

Do not weaken RLS or server-side ownership checks to make this test pass.

### Compatibility

```text
[ ] Existing active trips remain legal/usable.
[ ] Driver dashboard queries remain compatible with trips whose driver_id is null.
[ ] Existing event/timeline consumers do not require every Node 3 trip to have a driver.
[ ] Claimed → Start Arrival → Arrival Recorded remains functional after verification changes.
```

## Gate 3 — Full Build / Lint / Test Evidence

Run the project's configured verification commands, including as applicable:

```text
- TypeScript check
- Production build
- Lint
- Automated test suite
```

Record the **exact command**, exit status, and concise result for each command.

Important:

```text
PASS = command actually executed successfully.
NOT CONFIGURED = repository does not define that check.
FAIL = command executed and failed.
BLOCKED = command could not be executed for a documented environmental reason.
```

Never report a check as PASS if it was not actually executed.

## Gate 4 — Targeted Test Evidence

If the repository already has suitable tests, execute them.

If suitable automated tests do not exist, do not fabricate tests or results. Instead:

```text
- document which targeted behaviors were manually/API tested;
- identify the exact automated coverage gap;
- determine whether adding a focused test is safe and within Node 3 scope;
- if adding tests, keep them narrowly scoped to the approved Node 3 behavior/security requirements;
- execute the new tests and record their results.
```

## Gate 5 — Do Not Overreach

Do NOT implement:

```text
- Driver marketplace redesign
- Driver claim redesign
- Atomic claim locking
- Full delivery lifecycle
- AI changes
- Automated routing/geocoding
- Automated pricing
- Node 4 functionality
- Node 1 authorization changes
- Node 2 identity changes
```

## Gate 6 — Update Application Implementation Report

After verification, update:

```text
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Report.md
```

Do not erase the historical implementation evidence.

Add a clearly dated **Verification** section containing:

```text
- exact commit verified;
- commands actually run;
- targeted security/behavior results;
- build result;
- lint result;
- test result;
- any NOT CONFIGURED/BLOCKED items;
- any defects found and disposition;
- final verification conclusion.
```

Preserve the distinction between VERIFIED, INFERRED, UNKNOWN, and NOT CONFIGURED where useful.

## Gate 7 — Records Checkpoint

Do not create a final Node 3 completion checkpoint unless the verification evidence supports closure.

If all required gates pass, prepare/update the Node 3 completion checkpoint in:

```text
00_PROJECT_CONTROL/CHECKPOINTS/
```

The checkpoint must explicitly state:

```text
Node 3 → COMPLETE / ACCEPTED
Day 9 → CLOSED
```

and list the evidence supporting that decision.

If any required gate remains open, leave Node 3 as **ACCEPTANCE PENDING** and document exactly what remains.

## Expected Final Report

Return a concise execution report with:

```text
1. Repository commit tested
2. Commands executed + results
3. Targeted security/behavior tests + results
4. Build/lint/test status
5. Defects found (if any)
6. Files changed (if any)
7. Whether Node 3 acceptance criteria are now fully satisfied
8. Whether a completion checkpoint was created
```

## Acceptance Boundary

Node 3 may be marked complete only when:

```text
Targeted security/behavior verification → PASS
Full build/lint/test evidence             → recorded and acceptable
Ayush manual verification                 → already recorded as PASS
Records implementation report             → updated
Node 3 completion checkpoint              → created
```

The Ayush manual verification already recorded in `NODE_3_CLAIMED_TO_START_ARRIVAL_FIX_Report.md` confirms the Claimed → Start Arrival → Arrival Recorded path. Do not repeat that claim as a substitute for the remaining security/build/test evidence.
