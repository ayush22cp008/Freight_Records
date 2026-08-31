# Chat22 — Node 4 Subnode — Safe Automated Concurrency Test Infrastructure Plan

## Status
**APPROVED BY AYUSH — PLAN STAGE**

## Parent Node
Node 4 — Driver Marketplace + Atomic Claim

## Subnode purpose
Provide the smallest safe automated-testing capability required to satisfy the authoritative Node 4 acceptance criterion that concurrency/race-condition tests pass.

This Subnode is about **test infrastructure and evidence only**. It does not redesign the already-working marketplace or atomic claim implementation.

## Current evidence baseline
- Node 4 current-source investigation is complete.
- Driver marketplace behavior is implemented and manually verified by Ayush.
- Atomic claim implementation is verified in source/database behavior.
- Ayush manually tested two drivers competing for the same trip and observed one winner and one loser.
- No application-level automated test suite exists.
- No safe isolated Supabase test environment is configured.
- Chat19 attempted the automated test and correctly stopped because safe infrastructure was unavailable.
- The authoritative Node 4 roadmap explicitly requires concurrency/race-condition tests to pass.

## Objective
Establish the minimum safe test capability needed to execute a real concurrency test against the claim mechanism without using production data or weakening authentication/authorization.

## Scope

### 1. Test-runner assessment
Inspect the existing Next.js project and choose the smallest suitable test runner/framework already available or minimally introducible.

Do not add multiple overlapping frameworks.
Do not perform unrelated dependency upgrades.

### 2. Safe test environment
Determine the minimum safe isolated Supabase strategy supported by the current project/environment.

Requirements:
- Must not perform destructive concurrency testing against the production database.
- Must not expose production secrets.
- Must provide controlled test data and cleanup.
- Must support two distinct valid driver identities appropriate to the current auth architecture.

If a genuinely isolated Supabase environment cannot be established safely without major external infrastructure work, stop and report the blocker rather than weakening the test.

### 3. Minimal concurrency test
Implement one focused integration-level test of the existing claim path/database behavior.

Test setup:
```text
Trip = published
Trip.driver_id = NULL
Driver A = valid verified driver
Driver B = valid verified driver
```

Execute both claims concurrently for the same trip.

Required assertions:
```text
exactly 1 success
exactly 1 failure
final trip status = claimed
final driver assignment = exactly one driver
winner ∈ {Driver A, Driver B}
loser does not become assigned
```

The test must exercise the real claim behavior as realistically as the selected architecture permits. Do not replace the atomic operation with a mock that cannot prove the database race behavior.

### 4. Isolation and cleanup
Use dedicated test records and deterministic cleanup.

Never use an uncontrolled production trip or real user account as disposable test data.

### 5. Validation
Run:
- the new concurrency test;
- relevant existing checks if any;
- TypeScript/typecheck;
- build;
- lint if available, recording pre-existing failures separately.

Do not claim PASS without actual command output/evidence.

## Out of scope
Do not:
- redesign claim logic;
- change trip state machine;
- change driver identity model;
- change authorization architecture;
- change marketplace UI;
- change production database behavior solely to accommodate tests;
- create unrelated tests;
- introduce CI infrastructure beyond what is minimally required for this Subnode.

## Stop conditions
Stop and report rather than improvising if:
- safe test isolation requires production database access;
- service-role credentials would need to be exposed to the client;
- authentication cannot be safely controlled in tests;
- the selected test approach would only mock the critical atomic behavior;
- the work requires major architecture or infrastructure beyond this Subnode.

A stop condition may trigger roadmap reassessment if it becomes a major blocker.

## Deliverables
1. Minimal test infrastructure/configuration, if required.
2. Automated concurrency test.
3. Test execution evidence.
4. Implementation report under:
   `03_IMPLEMENTATION/implementation_reports/`
5. Exact source commit SHA and push status.

## Acceptance criteria for this Subnode
```text
[ ] Safe isolated test environment established
[ ] Test runner configured/minimally reused
[ ] Two distinct valid driver identities available for test
[ ] Same-trip claims execute concurrently
[ ] Exactly one claim succeeds
[ ] Exactly one claim fails
[ ] Final trip is claimed
[ ] Final assignment contains exactly one driver
[ ] Test passes reproducibly
[ ] Build/type/lint evidence recorded
[ ] No production data was used destructively
[ ] Implementation report recorded
```

## Relationship to Node 4
This Subnode does not close Node 4 by itself. After successful Subnode completion, Node 4 still requires the overall acceptance matrix and Ayush final verification before closure.

## Decision
**Proceed with the minimal safe automated concurrency-test infrastructure described above.**
