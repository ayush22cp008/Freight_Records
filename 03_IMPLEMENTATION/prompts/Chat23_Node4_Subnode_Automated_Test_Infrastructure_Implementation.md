# Chat23 — Node 4 Subnode — Automated Concurrency Test Infrastructure Implementation

## Role
You are Antigravity, the implementation/execution agent for Freight.

This prompt is authorized by the approved Node 4 Subnode plan. Implement **only** the minimum safe automated concurrency-test infrastructure required to satisfy the existing Node 4 acceptance criterion.

## Governing Records
Before changing anything, read:

1. `00_PROJECT_CONTROL/ROADMAP.md`
2. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
3. `00_PROJECT_CONTROL/PROJECT_STATE.md`
4. `02_ARCHITECTURE/Chat22_Node4_Subnode_Safe_Automated_Concurrency_Test_Infrastructure_Plan.md`
5. `00_PROJECT_CONTROL/Chat21_Node4_Roadmap_Reassessment.md`
6. `03_IMPLEMENTATION/implementation_reports/Chat18_Node4_Current_Source_Investigation_Report.md`
7. `03_IMPLEMENTATION/implementation_reports/Chat19_Node4_Concurrency_Test_Implementation_Report.md`

Do not silently override conflicts. If a newer authoritative record conflicts with this prompt, stop and report the conflict.

## Source repository
`ayush22cp008/freight_hackathon`

## Mandatory preflight
Before implementation:
- inspect current branch and commit SHA;
- inspect working-tree status;
- inspect `package.json`, lockfile, Next.js configuration, and existing test configuration;
- verify the existing claim route and atomic claim implementation remain intact;
- determine whether any test framework is already present;
- determine what safe Supabase/test environment options already exist.

Record the baseline before making changes.

## Core requirement
The authoritative Node 4 acceptance criterion requires concurrency/race-condition testing.

The test must prove:

```text
One published + unassigned trip
        ↓
Driver A claim ─┐
                ├── concurrent execution
Driver B claim ─┘
        ↓
Exactly ONE succeeds
Exactly ONE fails
        ↓
Final trip = claimed
Final assignment = exactly ONE driver
```

## Implementation principles

### 1. Minimal infrastructure
Use an existing test framework if one is already configured.

If none exists, select and configure the smallest appropriate test runner for this Next.js/Supabase project. Avoid large framework additions or unrelated dependency upgrades.

### 2. Safe isolation
The test MUST NOT perform destructive operations against the production Supabase database.

Establish the smallest safe isolated test environment supported by the project. Depending on what the source repository already supports, this may be a local/isolated Supabase environment or another explicitly safe test database.

Do not expose service-role credentials to browser/client code.
Do not commit real secrets.
Do not use real production users as disposable test identities.

If safe isolation cannot be achieved without significant external infrastructure work, STOP. Do not weaken the test and do not test production data. Report the blocker.

### 3. Real claim behavior
Prefer an integration-level test that exercises the actual claim API/database behavior.

Do NOT mock away the database atomicity being tested.

The test may use controlled test authentication/session setup appropriate to the current architecture, but it must preserve the server-side identity model.

### 4. Concurrent execution
The two claim attempts must be started concurrently. Do not serialize them by awaiting Driver A before starting Driver B.

Use the selected framework/runtime's actual concurrency mechanism.

Do not use arbitrary sleep timing as the proof of atomicity.

### 5. Assertions
The test MUST assert:

- exactly one claim succeeds;
- exactly one claim fails;
- the success response is the expected claim success;
- the failure response is the expected conflict/unavailable response;
- final trip state is `claimed`;
- final assigned driver is exactly one driver;
- final assigned driver is either Driver A or Driver B;
- both drivers cannot be assigned;
- the losing driver is not assigned.

### 6. Test data and cleanup
Create dedicated test trip and driver data.

The trip must begin in the claimable state required by the existing claim implementation.

Clean up safely after the test. Ensure cleanup cannot delete unrelated data.

## Production source protection
Do not redesign or modify the existing production claim algorithm merely to make it testable.

Do not modify:
- driver marketplace UI;
- trip detail UI;
- trip state machine;
- driver identity architecture;
- authorization model;
- IDOR protections;
- authentication product model;

unless a directly evidenced test integration issue makes a minimal production change unavoidable. If such a change appears necessary, STOP and report it before expanding scope.

## Validation
After implementation, run:

1. The new automated concurrency test.
2. Any relevant existing tests.
3. Typecheck/build/lint commands available in the project.

Separate newly introduced failures from pre-existing failures and provide command evidence.

A passing manual test does not substitute for the automated concurrency test.

## Required implementation report
Create/update:

`03_IMPLEMENTATION/implementation_reports/Chat23_Node4_Subnode_Automated_Test_Infrastructure_Implementation_Report.md`

The report must contain:

1. Status: PASS or BLOCKED
2. Source baseline before implementation
3. Test framework selected and rationale
4. Test-environment/isolation design
5. Files changed
6. Dependencies added, if any
7. How test identities are created/controlled
8. How the same-trip claim requests are executed concurrently
9. Exact assertions
10. Cleanup strategy
11. Commands executed
12. Actual test results
13. Build/type/lint results
14. Security/safety notes confirming production data was not destructively tested
15. Any blockers
16. Whether another Subnode is justified
17. VERIFIED / INFERRED / UNKNOWN summary
18. Commit SHA
19. Working-tree status
20. Push status

Do not claim automated concurrency PASS without actual execution evidence.

## Git/push rules
Do not assume local source changes are pushed. Follow the project's established source-repository push workflow and explicitly record:
- local commit SHA;
- whether pushed;
- pushed branch/ref if applicable.

## Scope stop conditions
STOP and report if:
- no safe isolated test environment can be created without significant external infrastructure;
- production data would need to be used destructively;
- secrets would need to be exposed or committed;
- only a mocked test can be produced instead of a meaningful atomicity integration test;
- implementation requires changing the production atomic claim logic;
- work expands materially beyond this Subnode.

## Final response to ChatGPT
Return only:
- PASS or BLOCKED;
- implementation report path;
- test framework/environment;
- test command and result;
- build/type/lint result;
- commit SHA;
- push status;
- blockers or Subnode recommendation.

Do not paste source files or the full report into chat.
