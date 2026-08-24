# Chat8 — NEW UPDATE: Hackathon Node Roadmap Rework

**Project:** Freight — AI Builders Hackathon  
**Purpose:** Rebuild the remaining hackathon execution plan around the newly clarified product model, role model, authorization model, and full-delivery goal.

**Planning status:** BASELINE v1 — intentionally adjustable as implementation reveals new requirements.

---

## 1. Why the Roadmap Is Being Rebuilt

The earlier implementation sequence was based on a simpler product assumption. Chat8 clarified that the intended product is broader:

```text
Company creates/publishes trip
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

Therefore authentication should **not** be implemented against the old simplified model. The product, role, trip, lifecycle, and authorization decisions must be locked first.

The current Chat8 record explicitly states that authentication implementation is paused until this rework is complete. The same record identifies RLS as verified/closed, rate-limiting architecture as already decided, and IDOR/API authorization as the remaining security priority. 

---

## 2. Current Starting Point

### Security

```text
RLS investigation
→ CLOSED / VERIFIED

Rate-limiting architecture
→ DECIDED

IDOR / API authorization
→ OPEN
```

The authenticated Supabase path has already been verified. Do not reopen the RLS investigation unless new evidence contradicts it.

### Product

```text
Company creates/publishes trip
→ CLARIFIED

Driver chooses/accepts trip
→ CLARIFIED

Atomic first-winner acceptance
→ REQUIRED

Company can be creator/sender or receiver depending on trip context
→ CLARIFIED

Whole-delivery tracking
→ CORE GOAL
```

### Authentication

```text
Authentication implementation
→ PAUSED
```

Reason: the identity/role/authorization model must match the final product model.

---

# 3. New Execution Method — Node-Based Roadmap

Instead of treating the remaining work as one large day-by-day implementation sequence, the project will be tracked through **Nodes**.

Each Node has:

- Node ID
- Objective
- Planned days
- Tasks
- Dependencies
- Acceptance criteria
- Actual time/days
- Status
- Investigation/report requirements
- Manual verification

### Important

**Planned days are estimates, not permanent commitments.**

The roadmap can be changed later when:

- a requirement changes,
- an architecture decision changes,
- a task takes less/more time,
- a blocker appears,
- a security issue is discovered,
- the hackathon deadline requires reprioritization.

When this happens, update the roadmap rather than silently deviating from it.

---

# 4. 22-Day Baseline Plan

There are approximately **22 hackathon days remaining** according to the current Chat8 product/security record.

Baseline allocation:

| Node | Work | Planned Days | Priority |
|---|---|---:|---|
| Node 1 | Product + Authorization Rework | 2 | 🔴 Critical |
| Node 2 | Authentication + Identity | 3 | 🔴 Critical |
| Node 3 | Company Trip Creation + Publishing | 3 | 🔴 Critical |
| Node 4 | Driver Marketplace + Atomic Claim | 3 | 🔴 Critical |
| Node 5 | Whole Delivery Tracking | 5 | 🔴 Critical |
| Node 6 | Security + Evidence | 3 | 🔴 Critical |
| Node 7 | AI + Final Integration + Demo | 3 | 🔴 Critical |
| **Total** | | **22** | |

This is the **baseline roadmap**, not a locked promise that every Node must consume exactly its allocation.

---

# 5. NODE 1 — Product + Authorization Rework

**Planned duration:** 2 days  
**Status:** 🔵 NEXT  
**Dependency:** None  
**Blocks:** Authentication implementation and all later authorization-sensitive work

## Objective

Lock the product and authorization model before writing the remaining authentication implementation.

## Tasks

### 1. Role model

Lock the MVP identities:

- Company
- Driver

Then define contextual company relationships:

- creating/sending company for a trip
- receiving company for a trip

A company must not be permanently hard-coded as only sender or only receiver.

### 2. Identity mapping

Define exactly how authenticated users map to:

```text
Auth user
   ↓
Company record OR Driver record
```

Define the required identifiers and relationships.

### 3. Trip data model

Lock the minimum trip relationships, including the conceptual equivalent of:

```text
creator_company_id
receiver_company_id
assigned_driver_id
status
```

Also lock the commercial/trip information required by the MVP:

- pickup
- destination
- distance
- expected duration
- payment/offer
- shipment details

### 4. Trip state machine

Define legal states and transitions.

Candidate baseline:

```text
DRAFT
  ↓
AVAILABLE
  ↓
CLAIMED
  ↓
IN_PROGRESS
  ↓
DELIVERED
  ↓
COMPLETED
```

The exact final state names can change during this Node.

### 5. Driver eligibility

Define who can see available trips.

Do not assume every authenticated driver automatically has access unless explicitly decided.

### 6. Atomic acceptance

Define the exact backend/database operation for:

```text
AVAILABLE → CLAIMED
```

Requirements:

- two drivers may attempt acceptance simultaneously;
- exactly one valid driver wins;
- winner becomes assigned driver;
- losing request receives a clear failure;
- frontend controls are not trusted for exclusivity.

### 7. Authorization matrix

Define permissions for at least:

| Action | Creating Company | Receiving Company | Assigned Driver | Unassigned Driver | Unrelated User |
|---|---|---|---|---|---|
| Create trip | TBD/ALLOW | TBD | DENY | DENY | DENY |
| Publish trip | TBD/ALLOW | TBD | DENY | DENY | DENY |
| View available trip | TBD | TBD | ALLOW if eligible | ALLOW if eligible | DENY |
| Accept trip | DENY | DENY | N/A | ALLOW if eligible | DENY |
| Arrival | TBD | TBD | ALLOW only if assigned | DENY | DENY |
| Check-in | TBD | TBD | ALLOW only if assigned | DENY | DENY |
| Departure | TBD | TBD | ALLOW only if assigned | DENY | DENY |
| Delivery confirmation | TBD | ALLOW for receiver relationship | TBD | DENY | DENY |
| View evidence | ALLOW according to trip relationship | ALLOW according to trip relationship | ALLOW | DENY | DENY |

**TBD cells must be explicitly resolved before Node 1 is complete.**

### 8. IDOR protection rules

Define server-side checks for every privileged API operation.

Critical invariant:

```text
Driver A cannot submit an event
for a trip assigned to Driver B.
```

Because privileged service-role database access bypasses RLS, application/API authorization must enforce these relationship checks.

## Acceptance Criteria

Node 1 is complete only when:

```text
[ ] Roles locked
[ ] Identity mapping locked
[ ] Company/receiver relationship locked
[ ] Trip schema relationships locked
[ ] Trip state machine locked
[ ] Driver eligibility locked
[ ] Atomic acceptance rule locked
[ ] Authorization matrix has no unresolved TBD permissions
[ ] IDOR protection rules defined
[ ] Authentication requirements derived from the final model
```

**Gate:** Do not begin Node 2 until these criteria are explicitly verified.

---

# 6. NODE 2 — Authentication + Identity

**Planned duration:** 3 days  
**Status:** 🔵 BLOCKED BY NODE 1  
**Dependency:** Node 1 complete

## Objective

Implement authentication that correctly identifies the product actor and connects that identity to the Company/Driver model.

## Tasks

- Company authentication
- Driver authentication
- Role identification
- Auth user → company/driver mapping
- Protected routes
- Session handling
- Authenticated request context
- Unauthorized-access tests
- Role/identity verification tests

## Acceptance Criteria

```text
[ ] Company can authenticate
[ ] Driver can authenticate
[ ] Authenticated user identity is reliable
[ ] Application can determine Company vs Driver
[ ] Identity maps to the correct company/driver record
[ ] Protected routes reject unauthenticated users
[ ] Wrong-role access is rejected
[ ] Authentication tests pass
```

---

# 7. NODE 3 — Company Trip Creation + Publishing

**Planned duration:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Node 1 + Node 2

## Objective

Allow a company to create and publish a real trip opportunity for eligible drivers.

## Tasks

- Company trip creation UI/API
- Pickup location
- Destination / receiving company
- Distance
- Expected duration / hours / days
- Payment/offer
- Shipment/trip details
- Draft state if required
- Publish action
- Company authorization checks

## Dynamic Offer Scope

The initial offer is manually controlled by the company.

If no driver accepts, the company may increase the offer.

For the hackathon MVP:

```text
Manual price adjustment → ALLOWED
Automated pricing engine → DEFERRED
```

## Acceptance Criteria

```text
[ ] Company can create a valid trip
[ ] Receiver relationship is stored
[ ] Trip details are visible
[ ] Initial offer is stored
[ ] Company can publish the trip
[ ] Published trip becomes available according to eligibility rules
[ ] Unauthorized users cannot create/publish for another company
```

---

# 8. NODE 4 — Driver Marketplace + Atomic Claim

**Planned duration:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Node 3

## Objective

Create the driver-side opportunity/acceptance flow.

## Tasks

- Available trip list
- Trip details
- Pickup/destination display
- Distance display
- Duration display
- Payment/offer display
- Driver acceptance
- Atomic first-winner claim
- Assigned-driver persistence
- Losing-driver response
- Race-condition testing

## Core Invariant

```text
Trip AVAILABLE
       ↓
Driver A accepts
Driver B accepts
       ↓
Exactly ONE wins
       ↓
Trip CLAIMED
       ↓
assigned_driver_id = winner
```

## Acceptance Criteria

```text
[ ] Eligible drivers can see available trips
[ ] Driver can evaluate the trip
[ ] Driver can accept
[ ] Exactly one simultaneous acceptance succeeds
[ ] Winner becomes assigned driver
[ ] Trip cannot be claimed again
[ ] Losing driver receives a clear response
[ ] Assignment cannot be manipulated through client input
```

---

# 9. NODE 5 — Whole Delivery Tracking

**Planned duration:** 5 days  
**Status:** 🔵 FUTURE  
**Dependency:** Node 4

## Objective

Turn the product from a marketplace/assignment demo into a complete delivery lifecycle.

## Target flow

```text
Pickup
 ↓
Arrival
 ↓
Check-in
 ↓
Load
 ↓
Depart
 ↓
In transit
 ↓
Destination
 ↓
Receiver Arrival
 ↓
Receiver Check-in
 ↓
Unload / Delivery
 ↓
Receiver confirmation
 ↓
Completed
```

## Tasks

### Pickup side

- Arrival
- GPS capture
- Server timestamp
- Photo/evidence as required
- Check-in
- Load
- Depart

### Transit

- In-transit state
- Trip status progression

### Destination side

- Destination arrival
- Receiving-company relationship
- Receiver check-in
- Unload/delivery event
- Delivery confirmation
- Completion state

### Evidence

Maintain immutable event history according to the existing evidence architecture.

## Scope Protection

Multiple stops may be supported architecturally, but the hackathon MVP should not introduce multi-stop complexity unless the core single-trip delivery lifecycle is already reliable.

## Acceptance Criteria

```text
[ ] One trip can move from published → claimed → in-progress → delivered/completed
[ ] Pickup events work
[ ] Transit state works
[ ] Destination events work
[ ] Receiving company can perform its permitted action
[ ] Unauthorized actors are blocked
[ ] Evidence timeline remains coherent
[ ] End-to-end single-delivery scenario works
```

---

# 10. NODE 6 — Security + Evidence

**Planned duration:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Nodes 2–5

## Objective

Close the API authorization/IDOR gap and verify the security boundaries of the final product model.

## Tasks

### IDOR / API authorization

Verify every privileged route checks the authenticated actor's actual relationship to the target trip/resource.

Examples:

```text
Driver A → Trip assigned to Driver B → DENY
Driver B → Trip assigned to Driver B → ALLOW

Unrelated company → another company's private trip → DENY
Receiving company → its receiving trip → ALLOW only for permitted actions
Creating company → its created trip → ALLOW only for permitted actions
```

### Evidence integrity

Verify:

- immutable event behavior
- server timestamps
- GPS capture
- photo/evidence constraints
- event ordering/state rules

### Rate limiting

Implement/verify the already-decided rate-limiting architecture.

Do not reopen the architecture decision unless new evidence requires it.

### Security testing

Test:

- direct API manipulation
- forged trip IDs
- forged driver IDs
- forged company IDs
- wrong-role requests
- unassigned driver event submission
- receiver/creator boundary violations
- duplicate acceptance
- replay/duplicate event attempts as applicable

## Acceptance Criteria

```text
[ ] IDOR attack paths are blocked
[ ] Every privileged API route has explicit authorization
[ ] Driver assignment boundary is enforced
[ ] Company relationship boundary is enforced
[ ] Atomic claim remains secure
[ ] Evidence remains immutable
[ ] Rate limiting is verified
[ ] Security test results are recorded
```

---

# 11. NODE 7 — AI + Final Integration + Demo

**Planned duration:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Nodes 3–6

## Objective

Make the complete product demo-ready without expanding scope unnecessarily.

## Tasks

- AI evidence-grounded summary
- Timeline UI
- Final API/UI integration
- End-to-end test
- Realistic demo data
- Security regression test
- Critical bug fixing
- UX polish
- Hackathon presentation/demo flow

## AI Boundary

The AI summary must remain evidence-grounded and deterministic-input based.

The AI should summarize recorded evidence rather than inventing facts or making unsupported blame/decision claims.

## Acceptance Criteria

```text
[ ] Complete delivery scenario works from company creation to completion
[ ] Evidence timeline is visible
[ ] AI summary is generated from recorded evidence
[ ] Security regression passes
[ ] Critical bugs resolved
[ ] Demo can be repeated reliably
[ ] Final presentation story is coherent
```

---

# 12. Day-Level Baseline

This is the initial 22-day allocation.

```text
Day 01–02 → NODE 1  Product + Authorization Rework
Day 03–05 → NODE 2  Authentication + Identity
Day 06–08 → NODE 3  Company Trip Creation + Publishing
Day 09–11 → NODE 4  Driver Marketplace + Atomic Claim
Day 12–16 → NODE 5  Whole Delivery Tracking
Day 17–19 → NODE 6  Security + Evidence
Day 20–22 → NODE 7  AI + Final Integration + Demo
```

This is a **planning baseline only**.

If Node 1 takes 3 days instead of 2, the roadmap should be updated explicitly rather than pretending the original estimate was still accurate.

---

# 13. Progress Tracking Template

After every Node, record:

```text
NODE:
NAME:

PLANNED DAYS:
ACTUAL DAYS:

START DATE:
END DATE:

STATUS:

COMPLETED:
- 

REMAINING:
- 

BUGS DISCOVERED:
- 

BUGS RESOLVED:
- 

BUGS DEFERRED:
- 

SECURITY FINDINGS:
- 

ACCEPTANCE CRITERIA:
- [ ]
- [ ]
- [ ]

IMPLEMENTATION REPORT:
03_IMPLEMENTATION/implementation_reports/<report-file>.md

NEXT NODE:

ROADMAP CHANGE REQUIRED:
YES / NO
```

---

# 14. Working Method for Every Node

The project continues to use the existing investigation-first workflow.

Do not jump directly into implementation when a Node starts.

For a new implementation topic:

```text
1. Investigate actual repository/code/database state
2. Identify current architecture
3. Identify gaps/bugs
4. Create investigation record
5. Create implementation plan/prompt
6. Review/approve the plan
7. Antigravity executes through the established GitHub bridge
8. Test implementation
9. Create implementation report
10. Update this roadmap
11. Move to next Node
```

Do not guess what the implementation currently does.

---

# 15. Record-Repository Workflow

The GitHub record repository remains the bridge between ChatGPT and Antigravity.

ChatGPT should not bypass this workflow by directly pasting implementation prompts into Antigravity.

### Investigation / implementation prompts

Store under:

```text
03_IMPLEMENTATION/prompts/
```

### Antigravity implementation reports

Store under:

```text
03_IMPLEMENTATION/implementation_reports/
```

### ChatGPT architecture/product handoffs

Store under:

```text
01_BRAIN_HANDOFFS/ChatGPT/
```

Every major Node should leave a durable record in the repository.

---

# 16. What Is Locked vs What Can Change

## Locked / already decided

```text
RLS investigation status → VERIFIED / CLOSED
Rate-limiting architecture → DECIDED
Company creates/publishes trip → CLARIFIED
Driver chooses/accepts trip → CLARIFIED
Atomic first-winner claim → REQUIRED
Contextual company sender/receiver relationship → REQUIRED
Full delivery lifecycle → CORE GOAL
Node-based planning method → ADOPTED
```

## Must be locked in Node 1

```text
Exact roles
Identity mapping
Trip schema relationships
Trip state machine
Driver eligibility
Acceptance implementation rule
Authorization matrix
IDOR protection rules
Exact MVP delivery scope
```

## Can change later

```text
Exact day allocation
Task order inside a Node
UI details
Optional evidence features
Dynamic offer UX
Multi-stop support
Demo polish
Stretch features
```

Changing a planning estimate is normal. Changing a security/product invariant must be explicitly recorded and reviewed.

---

# 17. Scope Protection Rules

With approximately 22 days remaining, prioritize a reliable vertical slice over enterprise completeness.

Do NOT let the hackathon expand into:

- enterprise-grade marketplace economics
- fully automated dynamic pricing
- unnecessary multi-stop complexity
- unnecessary background tracking
- unrelated features
- speculative AI features

The target is:

```text
Company creates trip
       ↓
Driver sees/evaluates
       ↓
Driver accepts
       ↓
Atomic claim
       ↓
Delivery is tracked
       ↓
Receiver participates where required
       ↓
Evidence is preserved
       ↓
AI summarizes evidence
       ↓
Security boundaries are demonstrated
```

That is the core hackathon story.

---

# 18. Current Position

```text
NODE 1 — Product + Authorization Rework
→ 🔵 NEXT

NODE 2 — Authentication + Identity
→ ⏸ PAUSED until Node 1 complete

NODE 3 — Company Trip Creation
→ 🔵 FUTURE

NODE 4 — Driver Marketplace + Atomic Claim
→ 🔵 FUTURE

NODE 5 — Whole Delivery Tracking
→ 🔵 FUTURE

NODE 6 — Security + Evidence
→ 🔵 FUTURE

NODE 7 — AI + Final Integration + Demo
→ 🔵 FUTURE
```

## Immediate next action

**Start Node 1 only.**

Do not begin authentication implementation yet.

The immediate objective is to finish the product/role/trip/authorization rework and explicitly lock the model that authentication and IDOR protection must implement.

---

# 19. Roadmap Change Rule

This roadmap is a living execution plan.

If implementation reveals that the plan must change:

```text
Discover change
    ↓
Record why
    ↓
Evaluate impact on security/product
    ↓
Update Node/task/day allocation
    ↓
Commit updated roadmap
    ↓
Continue execution
```

Never treat the original 22-day estimate as more important than the actual product/security requirements.

**The roadmap exists to control the project, not to prevent necessary changes.**
