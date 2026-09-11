# Chat49 — Day 21 — Node 7
# Antigravity Execution Prompt — Event-Driven Scoped Auto-Refresh Reinvestigation

## EXECUTION MODE

**INVESTIGATION ONLY — NO IMPLEMENTATION AUTHORIZED**

Execute the architecture reinvestigation referenced by:

`05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Reinvestigation.md`

Do not implement any proposed solution.

## OBJECTIVE

Determine whether Freight can safely achieve the following product behavior:

```text
Relevant business event occurs
→ only affected connected portal/page/component updates automatically
→ unrelated pages do not refresh
→ no fixed polling timer is required for normal synchronization
→ active capture/submission/local client state is protected
→ user does not need manual refresh for relevant state changes
```

## REQUIRED INVESTIGATION

Inspect the current application and produce evidence for:

1. Actual business event inventory and authoritative mutation sources.
2. Event → resource → authorized audience → exact route/component mapping.
3. Classification of dashboards, trip list/detail, reviewer surfaces, and every known `events/*` capture/submission route.
4. Client-state risks including files/photos, GPS, forms, modals, optimistic/local state, and in-progress submissions.
5. Comparison of Supabase Postgres Changes, Supabase Broadcast, and any existing application/API event mechanism that is actually present.
6. RLS/authorization/tenant isolation implications.
7. Event payload sensitivity and whether minimal identifiers plus authorized re-fetch are safer.
8. Missed, duplicate, delayed, out-of-order, reconnect, multiple-tab, and event-burst behavior.
9. Visibility/navigation lifecycle behavior.
10. Performance comparison against the global fixed timer proposal.
11. Whether targeted client/data updates are preferable to `router.refresh()` for any surfaces.

## REQUIRED OPTION COMPARISON

Compare:

```text
A — Global fixed timer
B — Event-driven scoped router.refresh()
C — Event-driven targeted client/data update
D — Hybrid, only if justified
```

Do not select an option merely because it is easiest to implement.

## REQUIRED EVIDENCE STANDARD

For every material finding use:

```text
VERIFIED
INFERRED
UNKNOWN
```

Do not claim a security, reliability, performance, or client-state property is VERIFIED without concrete evidence.

## STRICT IMPLEMENTATION PROHIBITION

Do NOT:

- add Supabase Realtime
- add polling/timers
- modify database publications
- modify schema/functions
- modify RLS/security
- modify authentication
- modify claim atomicity
- modify evidence persistence/lifecycle behavior
- modify locked Driver, Company, or Reviewer UI/UX
- create production code changes

If a temporary experiment is absolutely necessary, obtain/describe the minimum non-production-safe procedure in the report first; do not alter the locked production baseline.

## REQUIRED REPORT

Write the completed report to:

`05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Reinvestigation_Report.md`

The report must contain:

```text
OBSERVATION
INVESTIGATION
EVIDENCE
EVENT → RESOURCE → AUDIENCE → SURFACE MATRIX
ROUTE / CAPTURE SAFETY MATRIX
SECURITY ANALYSIS
RELIABILITY ANALYSIS
PERFORMANCE ANALYSIS
OPTIONS CONSIDERED
RECOMMENDED ARCHITECTURE
RISKS / TRADE-OFFS
REQUIRED VALIDATION TESTS
DECISION STATUS
```

## DECISION BOUNDARY

Your report is an investigation result only. Do not authorize implementation.

The final architecture decision and implementation authorization will be made after the report is reviewed through the project governance workflow.
