# Chat49 — Day 21 — Node 7
# Antigravity Execution Prompt — Event-Driven Scoped Auto-Refresh Investigation

## AUTHORIZATION

**Scope:** INVESTIGATION / DESIGN VERIFICATION ONLY

**Implementation authorized:** NO

**Production source-code modification:** NO

**Portal baseline modification:** NO

## OBJECTIVE

Execute the architecture investigation defined in:

`05_DEBUGGING/investigations/Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Architecture_Investigation.md`

Evaluate the proposed replacement for fixed global polling:

```text
BUSINESS EVENT
→ affected resource identified
→ scoped notification
→ only relevant connected page/component reacts
→ authorized data is re-fetched/updated
→ unrelated pages remain untouched
```

The goal is to determine whether this architecture can provide automatic synchronization for relevant connected surfaces without requiring a manual refresh and without introducing the risks identified in the prior global timer review.

## REQUIRED WORK

1. Inspect the current application source and existing Records evidence.
2. Inventory the meaningful business events and identify their authoritative mutation sources.
3. Map each event to the exact Company, Driver, and Reviewer routes/components that should react.
4. Explicitly identify routes/components that must not be automatically refreshed, especially `events/*` capture/file/photo/GPS/submission surfaces.
5. Inspect the current Supabase configuration and determine whether Postgres Changes, Broadcast, or another existing mechanism is appropriate.
6. Analyze authorization, RLS interaction, channel/resource isolation, and event payload safety.
7. Analyze missed, duplicated, delayed, reordered, burst, reconnect, background-tab, and multi-tab behavior.
8. Compare expected DB/server load with the previously proposed fixed 30-second global timer.
9. Identify any places where the proposed architecture would interfere with locked portal behavior.
10. Produce a concrete recommendation and the minimum verification test matrix required before implementation approval.

## EVIDENCE RULES

For every material finding use exactly one of:

```text
VERIFIED
INFERRED
UNKNOWN
```

Do not mark a behavior VERIFIED solely because of framework documentation or theoretical reasoning when it requires execution against this application.

Where execution is not possible, explicitly mark UNKNOWN and explain what test would resolve it.

## STRICT NON-GOALS

Do not:

- implement Realtime
- add polling/timers
- edit production UI
- edit database schema/functions
- edit RLS/security policies
- change authentication/authorization behavior
- change trip claiming behavior
- change evidence/lifecycle behavior
- modify locked portal baselines
- create a production implementation PR

## OUTPUT

Write the completed investigation report to the Records repo under:

`03_IMPLEMENTATION/implementation_reports/`

Use the filename:

`Chat49_Day21_Node7_EventDriven_Scoped_AutoRefresh_Investigation_Report.md`

The report must contain:

```text
Status
Scope
OBSERVATION
INVESTIGATION
EVIDENCE
Event → Resource → Portal/Route mapping
Route safety classification
Realtime mechanism analysis
Security / RLS analysis
Failure-mode analysis
Performance comparison
ROOT CAUSE / DESIGN CONSTRAINTS
OPTIONS CONSIDERED
RECOMMENDED ARCHITECTURE
RISKS / TRADE-OFFS
REQUIRED TESTS
DECISION STATUS
```

## FINAL DECISION BOUNDARY

Do not declare implementation approved in the report.

The final architecture decision remains with the project governance process after the report is reviewed.
