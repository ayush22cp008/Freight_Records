# CURRENT_STATUS.md

**Last updated:** Aug 26, 2026

## Where we are

**Historical Core MVP — IMPLEMENTED / VERIFIED.**

The original fixed single-facility, 3-event Core MVP remains completed and preserved:

- Login → Trip Hub → Arrival → Check-in → Departure → Timeline → AI Evidence Summary.
- Trip Hub remains the workflow state source of truth for the original Core MVP.
- Arrival, Check-in, and Departure are immutable evidence events with GPS + server timestamp.
- Arrival and Departure require photo evidence; Check-in remains optional-photo under the original Core MVP scope.
- Timeline displays recorded events chronologically with evidence.
- AI Evidence Summary interprets deterministic Arrival + Check-in + Departure evidence.
- AI summary truncation fix was implemented and browser-verified.
- `npm run build` passes.

The Core MVP is **not being discarded**. It is the verified foundation being extended into the broader product model defined by the active 7-Node roadmap.

## Current Product Direction

The active product model remains:

```text
Company creates / publishes trip
        ↓
Eligible drivers see opportunity
        ↓
Driver evaluates trip economics/details
        ↓
Driver accepts
        ↓
Atomic first-valid acceptance wins
        ↓
Trip locks to winning driver
        ↓
Pickup
        ↓
Arrival / Check-in / Load / Depart
        ↓
In transit
        ↓
Destination / receiving company
        ↓
Arrival / Check-in / Unload / Delivery confirmation
        ↓
Delivery completed
        ↓
Immutable evidence timeline
        ↓
AI evidence-grounded summary
```

The product rules and authorization model are governed by the Node 1 final lock:

`01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`

## Node 1 — Product + Authorization Rework

```text
Status → 🔒 FINAL LOCKED / COMPLETE
```

The Node 1 final-lock record explicitly states that Node 1 is formally locked and complete, with Claude independent final review:

```text
APPROVE — NO BLOCKING FINDINGS
```

The locked model includes:

- exactly 1 application identity per Auth User;
- exactly 1 application role per Auth User;
- role = Company OR Driver;
- contextual trip participant relationships;
- trip lifecycle and delivery sequence;
- server-side authorization / IDOR rules;
- concurrency rules;
- authentication requirements derived from the locked model.

This status is based on the existing Node 1 FINAL LOCK record; it does not claim that every later implementation acceptance criterion has been independently re-verified in this checkpoint.

## Node 2 — Authentication + Identity

```text
Status → 🔵 ACTIVE DESIGN / NOT LOCKED
```

Node 2 broad authentication/identity investigations are complete. The current contract is:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

Status:

```text
DRAFT / NOT LOCKED
```

Claude independently reviewed the Node 2 contract and found it not ready for full lock because several load-bearing decisions remain unresolved.

### Resolved evidence / decisions

```text
Broad Node 2 investigation              ✅ COMPLETE
Remaining auth evidence investigation  ✅ COMPLETE
Signup/onboarding investigation         ✅ COMPLETE
Claude Q1/Q4 independent review         ✅ COMPLETE
Q1 Signup consistency                   🔒 LOCKED DECISION
Q2 Email-confirmation policy            🔒 LOCKED DECISION
Q3 Session lifecycle / refresh          🔒 LOCKED DECISION
Q4 One-user/one-identity                🔒 LOCKED DECISION
Q5 Authentication rate limiting         🔒 LOCKED DECISION
```

### Current Node 2 decisions still requiring resolution

1. RLS / service-role boundary
2. Final acceptance-test matrix

### Q2 — Email-confirmation policy

**STATUS → 🔒 DECIDED / LOCKED FOR NODE 2**

Final policy:

```text
Email confirmation is required before the normal authenticated
user onboarding/evidence-submission flow and before Active/Usable access.
```

The verification stages are explicitly separated:

```text
1. Evidence submission
   → user action; requires confirmed authenticated session

2. Verifier review
   → authorized verifier action; does not require applicant session

3. Trusted-role assignment
   → server-controlled result of approved verification
```

Active/usable access requires all three:

```text
email_confirmed = true
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

`UNCONFIRMED + VERIFIED + trusted_role` is an allowed defensive state after an email-change/reconfirmation event, but it is always inactive and cannot access protected business operations.

The final decision report is:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Email_Confirmation_Policy.md`

Claude's independent review is:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Claude_Review_Q2_Email_Confirmation_Policy.md`

Q2 was approved after the Claude-identified precision corrections were incorporated.

### Q3 — Session lifecycle / refresh

**STATUS → 🔒 DECIDED / LOCKED FOR NODE 2**

Final policy:

```text
Policy B — Middleware-centered session refresh
+
live Freight DB Active gate
```

The session/authentication boundary is explicitly separated from Freight authorization:

```text
Valid Session
    = authenticated identity

Freight DB Active Gate
    = current account / verification state

Node 1 Authorization
    = operation-level authorization
```

Five approved Q3 decisions:

```text
1. Active status
   → live Freight DB state on every protected business request.

2. JWT
   → authentication/identity information only; never the sole Freight Active decision.

3. CSRF
   → SameSite cookie protection + Origin validation for cookie-authenticated state-changing requests.

4. PENDING / REJECTED
   → session may remain valid, but protected business access is immediately denied by the live DB Active gate.

5. Middleware
   → scoped to the authenticated/protected application surface; public/static routes bypass unnecessary auth processing.
```

Q3 security invariant:

```text
Valid Session
    ≠ Active Freight Account
    ≠ Authorized for every operation
```

Q3 also explicitly requires the live Freight Identity state to prevent stale JWT/session claims from preserving business access after verification or email-confirmation changes.

The refined Q3 decision report is:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q3_Session_Lifecycle_Refresh.md`

Claude's final independent approval is:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q3_Session_Lifecycle_Refresh_Claude_approved.md`

Claude verdict:

```text
APPROVE
```

The minor forward notes from Claude are preserved for contract/implementation work: keep the `getUser()` refresh caveat explicit and make Origin-header validation explicit in the eventual acceptance test/implementation wording.

Q3 is now locked at the architecture/policy level. Implementation remains paused.

### Q5 — Authentication rate limiting

**STATUS → 🔒 DECIDED / LOCKED FOR NODE 2**

Final MVP policy:

```text
Supabase-native Auth rate limiting
+
no custom Redis/Upstash limiter initially
+
no hard account lockout
+
generic authentication/recovery responses
+
secure client-IP forwarding where required
+
correct 429 handling
```

Supabase's Auth controls remain the primary authentication abuse-control mechanism for the MVP. Exact platform defaults are configuration details and must be re-verified at implementation time; they are not hard-coded Freight architecture constants.

The locked Q5 record is:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q5_Authentication_Rate_Limiting.md`

Claude independently reviewed Q5 and approved the underlying policy after two factual corrections were incorporated:

```text
1. Correct the inaccurate sign-in/sign-up rate-limit number.
2. Explicitly acknowledge separate MFA challenge/verify rate limits; MFA remains outside the current MVP scope.
```

Q5 does not replace Q2 Active enforcement, Q3 session lifecycle, or Node 1 authorization.

## Signup / onboarding evidence

The targeted investigation established that the current signup flow performs Auth User creation and application identity creation as separate operations rather than one database transaction.

A verified current failure state is:

```text
Auth User EXISTS
Application identity MISSING
```

The investigation also identified the reverse orphan risk associated with the current `ON DELETE SET NULL` relationship.

No implementation fix has been authorized from this evidence.

## Security / Authentication State

```text
Q5 authentication rate limiting       → 🔒 LOCKED
Q6 RLS / service-role boundary        → 🔵 OPEN / INVESTIGATION NEXT
IDOR / API authorization              → 🔒 LOCKED AS NODE 1 CONTRACT
authentication implementation          → PAUSED
```

The existing historical RLS investigation evidence is not itself a Q6 lock. Q6 must determine the final **RLS / service-role boundary policy** and reconcile it with the current architecture before implementation resumes.

## Active Roadmap Position

The active execution roadmap remains the existing 7-Node roadmap in `ROADMAP.md`.

```text
Historical Core MVP                  → IMPLEMENTED / VERIFIED
Node 1 Product + Authorization       → 🔒 COMPLETE / LOCKED
Node 2 Authentication + Identity    → 🔵 ACTIVE DESIGN / NOT LOCKED
Node 3 Company Trip Creation         → FUTURE
Node 4 Driver Marketplace            → FUTURE
Node 5 Whole Delivery Tracking       → FUTURE
Node 6 Security + Evidence           → FUTURE
Node 7 AI + Final Integration + Demo → FUTURE
```

**No roadmap rewrite is made by this update.**

## Checkpoint

Chat12 Day 5 checkpoint remains the historical continuation record:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md`

The Q2 lock and transition to Q3 are recorded in:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Q2_Lock_Checkpoint.md`

Q3 and Q5 locks are recorded by their finalized decision reports and Claude approval/review records. Dedicated lock checkpoints may be created as required by the project-control workflow.

## Subnode Rule

A Subnode is used only for significant unexpected work inside a Node.

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode

Major blocker / architecture change
→ stop and reassess roadmap
```

If one Node accumulates 3 or more Subnodes, perform an explicit roadmap reassessment.

## Record Routing

ChatGPT ↔ Antigravity bridge:

```text
GitHub Records repository
```

Implementation handoffs:

```text
03_IMPLEMENTATION/prompts/
```

Antigravity implementation reports:

```text
03_IMPLEMENTATION/implementation_reports/
```

Investigations:

```text
05_DEBUGGING/investigations/
```

Architecture records:

```text
02_ARCHITECTURE/
```

Project-control records:

```text
00_PROJECT_CONTROL/
```

## Next Action

**NEXT: Q6 — RLS / service-role boundary.**

Do not resume authentication implementation.

Q6 must be investigated and decided through the same workflow:

```text
Read authoritative records
        ↓
Investigate Q6
        ↓
Evidence
        ↓
Decision
        ↓
Independent review where appropriate
        ↓
Ayush approval
        ↓
Record Q6 decision
        ↓
Continue to the next unresolved Node 2 question
```

Q1, Q2, Q3, Q4, and Q5 are now locked decisions and must not be reopened unless new evidence creates a genuine conflict.
