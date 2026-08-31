# Chat20 — Node 4 — Automated Concurrency Test Decision

## Decision status
LOCKED FOR NODE 4 EXECUTION

## Context
Node 4 core marketplace and atomic claim behavior is already implemented and has been manually exercised with two drivers competing for the same trip. The current source investigation verified the server-side authenticated driver identity and the atomic database claim condition. The attempted automated concurrency-test implementation was blocked because the source repository has no configured test framework and no safe isolated Supabase test database/environment.

## Options considered

### Option A — Build full automated test infrastructure now
Would require, at minimum, selecting/configuring a test runner and establishing a safe isolated Supabase test environment with test data/auth setup and cleanup. This is significant additional infrastructure work for Node 4.

### Option B+ — Preserve implementation and use strong manual/non-destructive evidence
Keep the existing claim implementation unchanged. Preserve Ayush's two-driver concurrent manual test as evidence. Verify the atomic claim mechanism from the source/database contract without destructive production testing. Complete build/type/lint/test evidence that is safely available in the existing project, then perform final Ayush acceptance.

## Decision
**Choose Option B+ for Node 4.**

Do not introduce a full automated test framework and isolated Supabase environment solely to close Node 4 unless a later explicit project decision requires it.

## Rationale
1. The Node 4 claim implementation already exists and the source investigation verifies the atomic conditional database update.
2. Manual testing has demonstrated the intended observable race outcome: two drivers can see the same opportunity, one claim succeeds, and the other becomes unavailable.
3. Building a complete automated integration-testing environment is unexpected infrastructure scope relative to the already-working Node 4 feature.
4. The project must not run destructive concurrency tests against the live database merely to manufacture evidence.
5. Node 4 can proceed toward closure using the strongest safe evidence available, while explicitly documenting that no automated concurrency suite exists.
6. This decision does not weaken the requirement for atomic first-valid acceptance; it distinguishes implementation correctness from the absence of a reusable automated test harness.

## Required remaining Node 4 evidence
1. Preserve/document the Ayush manual two-driver concurrency test.
2. Verify the existing claim route and atomic database condition remain unchanged/correct.
3. Run available non-destructive project validation (build/type/lint and any existing applicable checks).
4. Complete final Ayush manual acceptance against the Node 4 acceptance matrix.
5. Record the Node 4 completion/closure evidence.

## Explicit limitation
Automated concurrency testing remains **NOT AVAILABLE** in the current project because no test runner or safe isolated test database is configured. Do not label the manual test as an automated test.

## Subnode decision
No testing-infrastructure Subnode is created at this time. The missing automated test is documented as a limitation rather than expanding Node 4 into a separate infrastructure project.

If a future requirement makes automated concurrency testing mandatory, reassess the roadmap explicitly before introducing that infrastructure.

## Classification
- Existing atomic claim implementation: VERIFIED
- Manual concurrent race behavior: VERIFIED by Ayush's test evidence
- Automated concurrency suite: NOT AVAILABLE
- Decision to proceed with B+: LOCKED
