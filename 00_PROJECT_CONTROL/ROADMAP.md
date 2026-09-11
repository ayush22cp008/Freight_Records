# ROADMAP.md

**Project:** Freight — AI Builders Hackathon  
**Hackathon window:** Aug 21 – Sep 15, 2026  
**Roadmap status:** ACTIVE EXECUTION ROADMAP — Driver, Company, and Reviewer accepted/locked; Chat50 NEW-Trip sender/receiver rule verified; Cross-Portal E2E next.  
**Current execution day:** Day 21  
**Current chat:** Chat50

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
Company Implementation              → COMPLETE / ACCEPTED / LOCKED + Chat50 NEW-Trip exception VERIFIED
Reviewer Implementation             → COMPLETE / VERIFIED / LOCKED
Reviewer Decision Atomicity         → VERIFIED
Reviewer Rollback Safety            → VERIFIED
Day 19                              → CLOSED
Day 20                              → CLOSED
Day 21 Auto-Refresh                 → DROPPED FROM CURRENT SCOPE
Day 21 Chat50 Same-Company Rule     → IMPLEMENTED / AYUSH VERIFIED
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
Company Portal → 🔒 LOCKED + Chat50 NEW-Trip governance exception VERIFIED
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

### Day 21 Chat50 — Same-Company Sender/Receiver Governance

The previous same-company Sender/Receiver behavior was formally superseded for NEW Trips by an approved governance decision.

Required invariant:

```text
Trip.company_id != Trip.receiving_company_id
```

Manual verification using the deployed `testc2` account confirmed:

```text
Own company in Receiving Company selector → NOT AVAILABLE → PASS
Direct authenticated testc2 → testc2 API request → HTTP 400 → PASS
Rejected CHAT50 TEST SAME COMPANY Trip visible → NO → PASS
```

Existing same-company Trips were preserved; no migration, deletion, reassignment, or historical retrofit was performed.

Authoritative records:

- `02_ARCHITECTURE/Chat50_Day21_Node7_SameCompany_Sender_Receiver_Governance_Decision.md`
- `03_IMPLEMENTATION/prompts/Chat50_Day21_Node7_SameCompany_Sender_Receiver_Governance_Implementation_Prompt.md`
- `03_IMPLEMENTATION/implementation_reports/Chat50_Day21_Node7_Report_SameCompany_Sender_Receiver_Governance.md`
- `04_TESTING/test_results/Chat50_Day21_Node7_SameCompany_Sender_Receiver_Governance_Manual_Verification_Test_Result.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat50_Day21_Node7_SameCompany_Sender_Receiver_Governance_Manual_Verification_Checkpoint.md`

### Next Action

**Proceed with Cross-Portal End-to-End validation and demo-readiness using the locked Driver, Company, and Reviewer baselines plus the verified Chat50 NEW-Trip sender/receiver invariant. Do not implement the dropped auto-refresh feature.**

## Governance

Day 21 / Chat50 is the current continuation point. Historical Day 20 / Chat48 records remain unchanged as historical records.

Locked portal behavior must not be changed silently. Any new defect follows:

```text
Observation → Investigation → Evidence → Root Cause → Decision → Fix → Build/Test → Ayush Manual Verification
```
