# Chat8 — NEW UPDATE: Authentication Implementation Pause Checkpoint

## Purpose

This record establishes the current decision to **pause authentication implementation** until the product, role, trip lifecycle, and authorization model are reworked and locked.

We should not implement authentication against the earlier simplified architecture because the intended Freight product model has now been clarified.

## Current Decision

```text
Authentication implementation → PAUSED

Reason:
Product + role + trip + authorization model must be locked first.
```

Authentication is not being rejected or redesigned unnecessarily. It is being deliberately sequenced after the business/authorization decisions so that the eventual authentication system supports the correct identity and role model.

## What Is Already Verified / Decided

### RLS

The real authenticated Supabase client path was tested.

Verified:

- Real Supabase Auth session established.
- Session had the expected `authenticated` role context.
- Direct authenticated public-client reads of protected core tables were blocked/empty under the current no-policy RLS configuration.
- Service-role access can still see the data because it bypasses RLS.

Status:

```text
RLS investigation → VERIFIED / CLOSED
```

Do not reopen this investigation unless new evidence appears.

### Rate Limiting

The rate-limiting security/architecture decision has already been made.

Do not reopen the architecture question during this product rework. Track implementation separately from the architecture decision.

### IDOR

The API-level IDOR/authorization issue remains open, but the old mental model must not be used for the final fix.

The correct model is now:

```text
Company creates/publishes trip
        ↓
Eligible drivers see trip
        ↓
Driver accepts
        ↓
Atomic first-valid acceptance wins
        ↓
Trip becomes assigned/locked
        ↓
Only assigned driver can perform transport events
```

Therefore the eventual IDOR check should be based on the authenticated driver's relationship to the claimed/assigned trip, not on the assumption that a driver created the trip.

## Product Model That Authentication Must Support

### Company

A company can create/publish a trip.

A company can also be the receiving company for another trip.

Company role is **contextual to each trip**, not permanently sender-only or receiver-only.

Example:

```text
Trip A:
Company A = creating/sending company
Company B = receiving company

Trip B:
Company B = creating/sending company
Company C = receiving company
```

### Driver

A driver:

- views eligible available trips
- sees trip details and economic offer
- decides whether the trip is worth accepting
- accepts a trip
- becomes the assigned driver only after successful atomic claim
- performs transport/delivery events only for trips assigned to them

### Trip

A trip is created/published by a company.

The trip contains enough information for eligible drivers to evaluate it, including the relevant pickup/destination, distance, duration, payment/offer, receiver, and shipment details required by the MVP.

### Acceptance

Two or more drivers may attempt to accept the same available trip simultaneously.

The backend/database must ensure exactly one valid winner.

The frontend must not be relied upon for exclusivity.

## Full Delivery Goal

The target product story is whole-delivery tracking, not only a three-event driver demo.

Conceptually:

```text
Company creates/publishes trip
        ↓
Driver accepts
        ↓
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
Destination / receiving company
        ↓
Arrival
        ↓
Check-in
        ↓
Unload / delivery confirmation
        ↓
Delivery completed
        ↓
Immutable evidence timeline
        ↓
AI evidence-grounded summary
```

The exact MVP scope still needs to be locked before implementation.

## Required Rework Before Authentication

The following decisions must be completed first.

### 1. Role model

Lock the exact MVP roles and identities:

- Company
- Driver
- Company as trip creator/sender for a specific trip
- Company as receiver for a specific trip

Define how a user maps to a company or driver record.

### 2. Trip data model

Lock the minimum trip relationships, including the equivalent of:

```text
creator_company_id
receiver_company_id
assigned_driver_id
status
```

Exact schema names and additional fields must be decided during architecture review.

### 3. Trip state machine

Define exact states and legal transitions.

Candidate concept:

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

This is a proposal, not yet a locked database implementation.

### 4. Driver eligibility

Define which drivers can see a given trip.

Do not assume every authenticated driver automatically has access to every trip unless that is explicitly decided.

### 5. Acceptance / claim rule

Define the atomic operation that changes:

```text
AVAILABLE → CLAIMED
```

and assigns exactly one driver.

### 6. Authorization matrix

For every important action, define who can perform it.

At minimum consider:

- create trip
- edit trip
- publish trip
- view available trip
- accept trip
- cancel trip
- arrival
- check-in
- departure
- upload evidence
- view delivery evidence
- receiver confirmation
- complete trip

Actors to evaluate:

- creating company
- receiving company
- assigned driver
- unassigned driver
- unrelated company/user

### 7. IDOR protection model

After the authorization matrix is locked, define the exact server-side ownership/relationship checks needed for every privileged API route.

Particular rule:

```text
Driver A cannot submit Arrival/Check-in/Departure
for a trip assigned to Driver B.
```

Because the API uses privileged service-role access, these checks must exist at the application/API authorization layer even though direct Supabase RLS access is blocked.

### 8. Authentication design

Only after the above decisions are locked should we finalize:

- login/signup requirements
- company authentication
- driver authentication
- role identification
- identity-to-company/driver mapping
- protected routes
- session handling
- authorization middleware/context

## New Execution Planning Model

The original day-by-day plan should be reworked into **Nodes**.

Each Node should contain:

```text
Node ID
Node name
Objective
Tasks
Dependencies
Estimated duration (days)
Actual duration
Start date
End date
Status
Acceptance criteria
Implementation report path
Manual test requirements
```

This is preferable for an AI-agent-driven hackathon because planned calendar days can differ significantly from actual implementation time.

## Proposed Node Structure

These are planning placeholders and must be finalized after the architecture rework.

### NODE 1 — Product + Authorization Rework

Target: 1–2 days

Tasks:

- Lock roles
- Lock company/receiver relationship
- Lock trip data model
- Lock trip lifecycle/state machine
- Lock driver eligibility
- Lock acceptance/claim rule
- Lock authorization matrix
- Define IDOR protection rules

Acceptance:

```text
Product + authorization architecture is explicitly locked.
```

### NODE 2 — Authentication + Identity

Target: 2–3 days

Tasks:

- Company authentication
- Driver authentication
- Role identification
- Identity-to-company/driver mapping
- Protected routes
- Session handling
- Authentication tests

Acceptance:

```text
Users can authenticate and the application can reliably identify
who they are and their product role/identity.
```

### NODE 3 — Company Trip Creation / Publishing

Target: 2–3 days

Tasks:

- Create trip
- Receiver company
- Pickup/destination
- Distance/duration
- Payment/offer
- Shipment details
- Publish trip

Acceptance:

```text
A company can publish a complete trip opportunity.
```

### NODE 4 — Driver Marketplace + Atomic Claim

Target: 2–3 days

Tasks:

- Available trip list
- Trip detail view
- Payment/offer visibility
- Accept trip
- Atomic first-winner claim
- Lock assignment
- Race-condition testing

Acceptance:

```text
Exactly one driver can successfully claim an available trip.
```

### NODE 5 — Whole Delivery Tracking

Target: 4–5 days

Tasks:

- Pickup flow
- Arrival
- Check-in
- Load
- Depart
- In transit
- Destination
- Receiver-side flow
- Delivery confirmation
- Completion

Acceptance:

```text
One complete end-to-end delivery can be tracked.
```

### NODE 6 — Security + Evidence

Target: 2–3 days

Tasks:

- IDOR/API authorization
- Role/relationship authorization
- Trip assignment checks
- Evidence integrity
- Rate-limit implementation/verification
- Security testing

Acceptance:

```text
Unauthorized users cannot perform actions outside their trip role.
```

### NODE 7 — AI + Final Integration

Target: 2–3 days

Tasks:

- AI evidence summary
- Timeline
- End-to-end integration
- Demo scenario
- Bug fixing
- Final testing

Acceptance:

```text
Complete hackathon-ready delivery story works end-to-end.
```

## Progress Tracking Rule

For every Node, record both:

```text
Estimated duration
Actual duration
```

Example:

```text
Node 2
Estimated: 3 days
Actual: 7 hours
Status: COMPLETE
```

After each Node, create/update an implementation report and manually verify the acceptance criteria before declaring it complete.

## Current Status

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture           → DECIDED
IDOR/API authorization                → OPEN
Product model                         → CLARIFIED, needs locking
Role model                            → NEEDS LOCK
Trip data model                       → NEEDS LOCK
Trip state machine                    → NEEDS LOCK
Driver eligibility                    → NEEDS LOCK
Atomic acceptance                     → REQUIRED / NEEDS DESIGN
Authorization matrix                 → NEEDS LOCK
Authentication implementation         → PAUSED
New Node-based roadmap                → REQUIRED
```

## Final Rule

**Do not start authentication implementation until NODE 1 is completed and its acceptance criteria are explicitly verified.**

The objective is to prevent authentication/authorization rework after the marketplace, company/receiver relationship, trip lifecycle, and IDOR rules are finalized.
