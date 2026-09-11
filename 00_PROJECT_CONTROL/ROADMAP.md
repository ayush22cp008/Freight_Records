# ROADMAP.md

**Project:** Freight — AI Builders Hackathon  
**Hackathon window:** Aug 21 – Sep 15, 2026  
**Roadmap status:** ACTIVE EXECUTION ROADMAP — Driver, Company, and Reviewer accepted/locked; Cross-Portal E2E next.  
**Current execution day:** Day 21  
**Current chat:** Chat49

## Active Roadmap — 7 Nodes

| Node | Work | Status |
|---|---|---|
| Node 1 | Product + Authorization Rework | 🔒 COMPLETE / LOCKED |
| Node 2 | Authentication + Identity | 🔒 COMPLETE / ACCEPTED |
| Node 3 | Company Trip Creation + Publishing | 🔒 COMPLETE / ACCEPTED |
| Node 4 | Driver Marketplace + Atomic Claim | 🔒 COMPLETE / ACCEPTED |
| Node 5 | Whole Delivery Tracking | 🔒 COMPLETE / ACCEPTED |
| Node 6 | Security + Evidence | 🔒 COMPLETE / ACCEPTED |
| Node 7 | AI + Final Integration + Demo | 🔵 ACTIVE |

Do not reopen Nodes 1–6 unless new evidence identifies a regression or a specific reviewer requirement.

## Current Active Position

```text
Historical Core MVP                → IMPLEMENTED / VERIFIED
Node 1                              → COMPLETE / LOCKED
Node 2                              → COMPLETE / ACCEPTED
Node 3                              → COMPLETE / ACCEPTED
Node 4                              → COMPLETE / ACCEPTED
Node 5                              → COMPLETE / ACCEPTED
Dashboard follow-up                → CLOSED / VERIFIED
Historical AI follow-up             → CLOSED / VERIFIED
Node 6                              → COMPLETE / ACCEPTED
Node 7                              → ACTIVE
Phase 1a                            → COMPLETE / ACCEPTED
Phase 1b Driver Portal              → COMPLETE / ACCEPTED / LOCKED
Phase 1b Company Portal             → COMPLETE / ACCEPTED / LOCKED
Phase 1b Reviewer Investigation     → COMPLETE
Phase 1b Reviewer Mental Model      → COMPLETE / LOCKED
Phase 1b Reviewer Interaction Map   → COMPLETE / LOCKED
Phase 1b Reviewer Final Blueprint   → COMPLETE / LOCKED
Shared Cross-Portal Design System   → LOCKED
Implementation Boundary             → COMPLETE / GOVERNED
Implementation Preparation          → COMPLETE / APPROVED
Driver Implementation               → COMPLETE / ACCEPTED / LOCKED
Company Implementation              → COMPLETE / ACCEPTED / LOCKED
Reviewer Implementation             → COMPLETE / VERIFIED / LOCKED
Reviewer Decision Atomicity         → VERIFIED
Reviewer Rollback Safety            → VERIFIED
Day 19                              → CLOSED
Day 20                              → CLOSED
Day 21 Auto-Refresh                 → DROPPED FROM CURRENT SCOPE
Cross-Portal E2E / Demo              → NEXT
Phase 3                             → CONDITIONAL
```

## Node 7 — AI + Final Integration + Demo

**Status: 🔵 ACTIVE**

### Execution sequence

```text
Phase 1a
   ↓
Phase 1b / 1c
   ↓
Driver Portal → 🔒 LOCKED
   ↓
Company Portal → 🔒 LOCKED
   ↓
Reviewer Portal → 🔒 LOCKED
   ↓
Cross-Portal E2E
   ↓
Final integration bugfix / regression verification
   ↓
Demo readiness
   ↓
Final presentation
```

### Day 21 Auto-Refresh Decision

The proposed Cross-Portal auto-refresh enhancement was investigated and independently reviewed. Global fixed-timer polling was rejected. Event-driven, resource-scoped Realtime refresh was found architecturally compatible with conditions, but it was not required for the current completion target and was dropped from the current implementation scope.

Production verification established that the `supabase_realtime` publication exists, while no application tables were attached at the time of verification. No production Realtime publication changes were made.

The investigation/test records remain preserved as historical evidence. Auto-refresh must not be implemented unless its scope is explicitly reopened later.

### Next Action

**Proceed with Cross-Portal End-to-End validation and demo-readiness using the locked Driver, Company, and Reviewer baselines. Do not implement the dropped auto-refresh feature.**

## Governance

Day 21 / Chat49 is the current continuation point. Historical Day 20 / Chat48 records remain unchanged as historical records.

Locked portal behavior must not be changed silently. Any new defect follows:

```text
Observation → Investigation → Evidence → Root Cause → Decision → Fix → Build/Test → Ayush Manual Verification
```
