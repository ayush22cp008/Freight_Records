# Chat49 — Day 21 — Node 7
## Cross-Portal Auto-Refresh Investigation

**Status:** OPEN — investigation/design only

## Objective

Determine whether and how the locked Driver, Company, and Reviewer portal dashboards can reflect relevant backend state changes without requiring a manual browser refresh.

## Governance Boundary

The three portal baselines are locked. This record does not authorize implementation or change any locked portal behavior.

Any implementation must follow the project investigation pipeline and require an explicit decision/approval before an implementation instruction is created.

## Required Investigation

Inspect the current source implementation and determine:

1. Which dashboards/views currently fetch their data.
2. When those fetches/refetches occur.
3. Whether the application already has polling, realtime subscriptions, query invalidation, focus-based refetch, or another refresh mechanism.
4. Which cross-portal state changes should become visible automatically.
5. Whether automatic refresh can be added without changing business rules, API contracts, authorization/RLS, persistence, lifecycle semantics, evidence semantics, or locked portal product behavior.
6. The safest mechanism for this project, based on evidence from the current implementation.
7. Any performance, race-condition, stale-data, duplicate-request, or UX risks.

## Required Investigation Output

Use the project evidence discipline:

OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION

Do not propose or implement a fix until the current refresh architecture and its implications are evidenced.

## Expected Outcome

Produce an evidence-based recommendation stating whether automatic dashboard refresh should be introduced, where it should apply, and what mechanism is appropriate. If implementation is recommended, stop at the decision stage until Ayush explicitly approves the implementation direction.

**Chat:** Chat49
**Day:** Day21
**Node:** Node7
