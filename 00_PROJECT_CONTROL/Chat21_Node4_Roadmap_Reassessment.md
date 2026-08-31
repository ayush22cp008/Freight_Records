# Chat21 — Node 4 Roadmap Reassessment

## Status
**REASSESSMENT REQUIRED — Node 4 remains OPEN**

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
The previously recorded B+ decision to proceed without automated concurrency testing cannot override the authoritative Node 4 acceptance criteria. It is therefore **SUPERSEDED for purposes of Node 4 closure**.

The roadmap must now explicitly choose between:

### Option A — Build minimal safe automated test infrastructure
Scope:
1. Select the smallest suitable test runner for the existing Next.js/Supabase project.
2. Establish a safe isolated test database/environment.
3. Create controlled test identities/data.
4. Implement one focused integration concurrency test against the real claim behavior.
5. Add safe cleanup/teardown.
6. Run the concurrency test plus applicable project checks.
7. Record reproducible evidence.

**Benefit:** Satisfies the existing Node 4 acceptance criterion without weakening it.

**Cost/risk:** Adds testing infrastructure that was not present in the current project and may require additional setup/configuration work.

### Option B — Formally change the Node 4 acceptance criterion
This would require an explicit roadmap/product decision to replace or defer the automated concurrency-test requirement and document the resulting evidence standard.

**Benefit:** Avoids infrastructure work.

**Cost/risk:** Weakens/changes an already locked core acceptance criterion and would need explicit justification. It must not be treated as an implementation shortcut.

## Recommended direction
**Recommend Option A.**

The atomic first-valid acceptance requirement is a core Node 4 invariant, and automated concurrency evidence is explicitly required by the active roadmap. The smallest safe automated test infrastructure is therefore the technically correct route if the hackathon project can absorb the additional setup.

However, implementation must remain controlled:

```text
Reassessment → explicit approval/lock
       ↓
Minimal infrastructure design
       ↓
Implementation prompt through Records bridge
       ↓
Antigravity implementation
       ↓
Automated concurrency evidence
       ↓
Build/type/lint/test evidence
       ↓
Ayush manual verification
       ↓
Node 4 closure
```

Do not modify production claim logic merely to make testing easier.
Do not point destructive tests at the production database.
Do not call the existing manual race test an automated test.

## Subnode assessment
A testing-infrastructure Subnode is justified **only if Option A is explicitly approved**, because establishing a safe automated integration-testing environment is significant unexpected work inside Node 4.

Suggested name if approved:

`Node 4 Subnode — Safe Automated Concurrency Test Infrastructure`

This Subnode would not change the Node 4 atomic-claim requirement; it would provide the evidence mechanism required to satisfy it.

## Current closure status

```text
Node 4 implementation             → IMPLEMENTED / VERIFIED
Atomic claim mechanism             → VERIFIED
Manual race evidence               → VERIFIED
Automated concurrency evidence     → MISSING
Node 4 acceptance                  → OPEN
Ayush final Node 4 verification    → OPEN
Node 4 closure                     → BLOCKED pending decision/evidence
```

## Important historical-record rule
Do not rewrite or delete the earlier Chat20 decision. This reassessment supersedes its conclusion because the authoritative roadmap acceptance criterion was re-checked and found to require concurrency-test evidence.
