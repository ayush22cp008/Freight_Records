# Chat21 — Node 4 Roadmap Reassessment

## Status
**DECISION LOCKED — OPTION A APPROVED — NODE 4 REMAINS OPEN**

## Trigger
The active Node 4 roadmap explicitly includes concurrency/race-condition tests as a core task and its acceptance criteria require:

```text
[ ] Concurrency tests pass
[ ] Ayush verification complete
```

The current source investigation confirms that the production claim mechanism is atomic, but the source repository has no application-level automated test framework and no safe isolated Supabase test environment. The Chat19 automated-concurrency implementation attempt therefore stopped before destructive operations.

This is a scope/infrastructure conflict that cannot be silently waived.

## Evidence reviewed
- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/PROJECT_STATE.md`
- `03_IMPLEMENTATION/implementation_reports/Chat18_Node4_Current_Source_Investigation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat19_Node4_Concurrency_Test_Implementation_Report.md`
- `02_ARCHITECTURE/Chat20_Node4_Automated_Concurrency_Test_Decision.md`

## Confirmed current state

### Node 4 implementation
- Available published trips: IMPLEMENTED.
- Trip evaluation/details: IMPLEMENTED.
- Driver claim: IMPLEMENTED.
- Server-side authenticated driver identity: VERIFIED.
- Atomic conditional claim (`published` + unassigned): VERIFIED.
- Winner assignment persistence: VERIFIED by source inspection/manual behavior.
- Already-claimed rejection: VERIFIED by source inspection/manual behavior.
- Client-controlled driver identity manipulation: prevented by server-side identity resolution.
- Two-driver manual concurrent test: VERIFIED by Ayush's observed result (one winner, one loser).

### Automated evidence
- Application-level automated concurrency test: NOT AVAILABLE.
- Test runner: NOT configured.
- Safe isolated Supabase integration-test environment: NOT configured.

## Conflict / decision point
The earlier B+ decision to proceed without automated concurrency testing cannot override the authoritative Node 4 acceptance criteria. It is therefore **SUPERSEDED for purposes of Node 4 closure**.

## Decision
**Option A — Build the minimal safe automated test infrastructure — is APPROVED.**

This decision preserves the existing Node 4 acceptance criterion instead of weakening it.

## Approved Subnode
**Node 4 Subnode — Safe Automated Concurrency Test Infrastructure**

Approved purpose:
1. Select/configure the smallest suitable test runner.
2. Establish a safe isolated Supabase test environment.
3. Create controlled test identities/data.
4. Implement one focused integration concurrency test against the real claim behavior.
5. Add safe cleanup/teardown.
6. Run the concurrency test plus applicable project checks.
7. Record reproducible evidence.

## Constraints
- Do not modify the production claim algorithm merely to make it testable.
- Do not point destructive tests at the production database.
- Do not expose or commit secrets.
- Do not introduce multiple overlapping test frameworks.
- Do not redesign marketplace, authentication, identity, or authorization architecture.
- If major external infrastructure is unexpectedly required beyond this Subnode, stop and trigger explicit reassessment rather than silently expanding scope.

## Expected time/scope
This is an estimate only. Antigravity must establish the actual effort from the repository state before implementation.

Target:
- **Minimal setup and focused test:** preferably within approximately 0.5–1 day of project work if a local/isolation path is practical.
- If safe isolation requires materially more than this or introduces major infrastructure, report the variance before expanding scope.

The estimate is intentionally bounded because the project should not turn Node 4 into a general testing-infrastructure project.

## Required Subnode completion evidence
```text
[ ] Safe isolated test environment established
[ ] Test runner configured/minimally reused
[ ] Two distinct valid driver identities available
[ ] Same-trip claims execute concurrently
[ ] Exactly 1 claim succeeds
[ ] Exactly 1 claim fails
[ ] Final trip status = claimed
[ ] Final assignment = exactly 1 driver
[ ] Test passes reproducibly
[ ] Build/type/lint/test evidence recorded
[ ] No production data was destructively tested
[ ] Implementation report recorded
```

## Node 4 remains open after Subnode completion
The Subnode is an evidence-enablement milestone. It does not by itself close Node 4.

After successful Subnode completion:

```text
Automated concurrency evidence
        ↓
Remaining Node 4 acceptance verification
        ↓
Ayush final verification
        ↓
Node 4 closure
```

## Subnode count / roadmap escalation
This is the first formal Subnode under Node 4.

If Node 4 reaches 3 or more Subnodes, trigger an explicit roadmap reassessment as required by the project rules.

## Current closure status

```text
Node 4 implementation             → IMPLEMENTED / VERIFIED
Atomic claim mechanism             → VERIFIED
Manual race evidence               → VERIFIED
Automated concurrency evidence     → MISSING
Node 4 Subnode                     → APPROVED / ACTIVE
Ayush final Node 4 verification    → OPEN
Node 4 closure                     → OPEN
```

## Historical-record rule
Do not delete or rewrite the earlier Chat20 decision. This reassessment supersedes its conclusion for Node 4 closure because the authoritative roadmap acceptance criterion requires concurrency-test evidence.
