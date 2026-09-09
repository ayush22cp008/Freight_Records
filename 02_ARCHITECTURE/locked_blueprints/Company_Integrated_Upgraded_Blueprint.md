# Company Portal — Integrated Final Blueprint

**Status:** LOCKED  
**Portal:** Company  
**Node:** Node 7 — Phase 1b  
**Purpose:** Consolidate the original Company Locked Blueprint with the later approved and implemented Company upgrades so one document represents the current intended Company product behavior.

## Source baseline

Original authoritative blueprint:

`02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`

This integrated document preserves that baseline and adds the later approved Company upgrades rather than rewriting the historical record.

## Incorporated approved upgrades

- Sender/Receiver-aware Company History with `Sent / Received` labels and `All | Sent | Received` filters.
- Sender/Receiver-aware Recent Completed Dashboard wording and direction.
- Persistent Receiver Delivery Request / handshake.
- Receiver Accept / Reject.
- Server-side Publish gating on Receiver acceptance.
- Independent server-side Claim gating on Receiver acceptance.
- Pending Requests in Incoming Deliveries.
- Dashboard Needs Attention shortcut for pending Receiver Accept/Reject requests.

---

# 1. Company Product Model

A Company is one business participant that may act as Sending Company on one Trip and Receiving Company on another Trip. Sender/Receiver role is trip-specific, not a permanent Company account type.

Authoritative relationships:

```text
Trip.company_id
→ Sending Company

Trip.receiving_company_id
→ Receiving Company
```

The Company portal manages Trips, monitors delivery progress, performs authorized receiver actions, reviews completed history, views delivery evidence, manages Company/account information, and uses existing Public Share capabilities.

# 2. Company Navigation

The Company portal uses one unified navigation:

```text
Dashboard
My Created Trips
Incoming Deliveries
History / Timeline
Profile / Account
```

There are no separate Sender and Receiver portals.

# 3. Dashboard

The Dashboard answers:

> What needs my attention now, and what is happening with my active work?

Hierarchy:

```text
1. Needs Attention
2. Active Created Trips
3. Quick Access
```

## 3.1 Needs Attention

Needs Attention contains only actions that actually require Company action. Normal delivery progress is not treated as an alert.

Existing examples include:

- Receiver Check-in Required.
- Delivery Confirmation Required.
- Other approved state-driven Company actions.

### Receiver Accept/Reject attention entry

When the authenticated Company has one or more genuine `PENDING` Receiver Delivery Requests, show:

```text
Receiver: Accept/Reject Delivery Request    [Take Action]
```

This is a **discovery/shortcut only**. The Dashboard does not contain the actual Accept/Reject controls.

Clicking `Take Action` routes to:

```text
/company/incoming
```

where the existing Pending Requests action surface contains:

```text
[Accept Request] [Reject]
```

## 3.2 Empty attention state

When no Company action is pending, preserve the existing positive state such as:

```text
No actions needed
```

Do not show a false Receiver Accept/Reject alert.

## 3.3 Active Created Trips

Active Created Trips provides an operational snapshot of sender-side work, including current status, Driver/claim status, delivery progress, and relevant next action.

## 3.4 Quick Access

Quick Access continues to provide direct navigation to Create Trip, Incoming Deliveries, History, and other already-approved Company destinations without duplicating those pages.

# 4. My Created Trips

My Created Trips is the sender-side monitoring and management surface.

It answers:

```text
What happened to Trips my Company created?
```

A Trip may communicate:

```text
Trip identity
Current status
Driver / claim status
Delivery progress
Receiver agreement state when relevant
Next relevant action
```

# 5. Incoming Deliveries

Incoming Deliveries is the Company’s **Receiver-side operational action surface**.

It is not a duplicate sender-side trip dashboard.

Current logical structure:

```text
Incoming Deliveries
│
├── Pending Requests
│     └── Accept / Reject
│
└── Active Deliveries
      └── Existing receiving-side operational actions
```

## 5.1 Pending Requests

A pending request clearly presents:

```text
Trip identity
Status: Pending Acceptance
Destination/context
[Accept Request] [Reject]
```

## 5.2 Active Deliveries

After Receiver acceptance and Driver execution begins, existing receiving-side operational states remain available, including Receiver Check-in, Delivery Confirmation/Completion, and the existing evidence/state workflow.

The Receiver Request stage does not replace the operational delivery lifecycle.

# 6. Receiver Accept/Reject Upgrade

Receiver agreement is represented by a separate persistent Receiver Delivery Request / handshake entity that references the existing Trip.

It does not create a duplicate Trip and does not replace the existing Trip lifecycle.

## 6.1 Request state machine

```text
PENDING
   ├── ACCEPTED
   └── REJECTED
```

## 6.2 Operational Trip lifecycle

```text
DRAFT
→ PUBLISHED / AVAILABLE
→ CLAIMED
→ IN_PROGRESS
→ COMPLETED
```

The two state machines remain separate.

## 6.3 Canonical inter-company flow

```text
Sender creates Trip
        ↓
Receiver Request = PENDING
        ↓
Receiver decides
   ┌────┴────┐
   ↓         ↓
 ACCEPT    REJECT
   ↓         ↓
Accepted   Rejected
   ↓         ↓
Publish    Publish blocked
allowed
   ↓
Driver Marketplace
   ↓
Driver Claim
   ↓
Existing delivery lifecycle
```

Required invariant:

```text
PENDING  → Driver marketplace execution blocked
REJECTED → Driver marketplace execution blocked
ACCEPTED → normal publication/claim rules may proceed
```

# 7. Receiver Authorization

Only the authenticated Receiving Company matching the Trip’s `receiving_company_id` may Accept or Reject the Receiver Request.

The following cannot decide the request:

- Sending Company.
- Driver.
- Unrelated Company.
- Unauthenticated user.
- Client-supplied identity that does not match the authenticated Company.

Authorization remains server-authoritative.

# 8. Accept / Reject Atomicity

Accept and Reject are mutually exclusive terminal decisions.

Valid transitions are:

```text
PENDING → ACCEPTED
PENDING → REJECTED
```

A later conflicting decision cannot overwrite the first valid decision.

At most one active `PENDING` Receiver Request may exist for a Trip.

Future resend, when separately approved, uses a new request identity rather than reopening a terminal record.

# 9. Publish Gate

For external Receiver Trips:

```text
PENDING  → Publish blocked
REJECTED → Publish blocked
ACCEPTED → Publish may proceed
```

The rule is server-side. UI visibility does not establish marketplace eligibility.

# 10. Claim Gate

Driver Claim independently checks Receiver agreement:

```text
PENDING  + PUBLISHED → Claim denied
REJECTED + PUBLISHED → Claim denied
ACCEPTED + PUBLISHED → Existing claim rules apply
```

The existing atomic Claim transaction remains authoritative. The agreement check is an additional precondition, not a replacement for existing Driver authorization or concurrency protection.

# 11. Same-Company Sender = Receiver

When authoritative Trip data shows:

```text
Trip.company_id == Trip.receiving_company_id
```

an external Company-to-Company handshake is not required.

The bypass is derived server-side from authoritative Trip relationships and authenticated Company identity. The client cannot manufacture it.

Normal Trip publication and marketplace rules still apply.

# 12. Legacy Trip Compatibility

Existing operational Trips created before the Receiver Request layer must continue working.

Where Company relationships are valid, legacy operational Trips are treated as agreement-satisfied through the approved migration/backfill strategy.

Legacy Trips that predate Company relationships and lack authoritative Company identities are not assigned fabricated Receiver decisions.

Historical records remain within their original operational boundaries.

# 13. Unified Trip Detail

All Company Trips use one unified Trip Detail structure:

```text
1. Current Status
2. Visual Delivery Progress
3. Next Required Action
4. Driver / Claim Information
5. Trip Details
6. Delivery Evidence
7. Timeline / History
```

Sender and Receiver use the same core structure; relationship-specific actions appear only when authorized and appropriate for the current state.

Receiver agreement status may be displayed when relevant, but Incoming Deliveries remains the primary Accept/Reject action surface.

# 14. Receiver Operational Workflow

Receiver Accept/Reject and operational receiving actions are separate stages:

```text
Accept/Reject
→ inter-company agreement before marketplace execution

Receiver Check-in
→ operational delivery action after Driver execution begins

Receiver Completion
→ operational delivery completion
```

Existing Receiver Check-in, Receiver Completion, and Driver Completion behavior remains protected.

# 15. Company History Upgrade

History is the Company-wide historical view of Trips in which the Company participated.

Both Sender and Receiver participation are included.

Completed Trips remain review-only and open the same read-only unified Trip Detail.

## 15.1 Relationship labels

Completed history explicitly distinguishes:

```text
Sent
Received
```

## 15.2 Filters

History provides:

```text
All | Sent | Received
```

Meaning:

```text
All
→ all completed Trips involving the Company

Sent
→ Trips where Trip.company_id = authenticated Company

Received
→ Trips where Trip.receiving_company_id = authenticated Company
```

## 15.3 Recent Completed Dashboard wording

For a sent Trip, the Dashboard may communicate:

```text
Your recent sent trip is finished
Your delivery to [destination]...
```

For a received Trip:

```text
Your recent received delivery is finished
Your delivery from [facility]...
```

This prevents ambiguity when the same Company participates in both roles.

# 16. Completed Trip Discovery

A completed Trip can be reached from:

- Recent Completed Dashboard content.
- History.
- Other relevant approved Company surfaces.

The destination is the exact unified Trip Detail for that Trip.

# 17. Public Share

Only the Receiving Company can create or revoke Public Share for an applicable Trip.

The Receiver Accept/Reject feature does not transfer Public Share authority.

# 18. Profile / Account

Company Profile / Account remains the dedicated destination for existing Company/account information and approved basic controls.

No separate Company-management subsystem is introduced.

# 19. Security Model

Sensitive operations remain server-authoritative:

- Receiver Accept.
- Receiver Reject.
- Publish.
- Claim.
- Public Share.
- Company relationship checks.

Client-side visibility and Dashboard navigation do not establish authorization.

Cross-tenant access must remain blocked.

The current server-side service-role authorization pattern remains the application security boundary.

# 20. Driver Compatibility

The Driver Portal keeps the existing marketplace and claim workflow.

The Receiver agreement is an additional server-side prerequisite:

```text
Pending  → not claimable
Rejected → not claimable
Accepted → normal marketplace/claim flow
```

No unnecessary Driver UI redesign is required.

# 21. Reviewer Compatibility

Reviewer behavior begins at the operational delivery workflow and remains unchanged by the pre-marketplace Receiver agreement layer.

No Receiver Accept/Reject action is added to Reviewer surfaces.

# 22. Responsive Company Portal

One responsive Company Portal is maintained across phone, tablet/intermediate widths, and laptop/desktop.

The same information hierarchy and workflow destinations are preserved.

No normal Company workflow requires horizontal scrolling.

# 23. State-Driven Attention Model

The Company Dashboard highlights only actions that require Company action.

Canonical examples:

```text
Pending Receiver Request
→ Receiver: Accept/Reject Delivery Request
→ Take Action
→ /company/incoming

Receiver Check-in Required
→ existing Receiver action surface

Delivery Confirmation Required
→ existing completion action surface
```

Discovery and action are intentionally separated:

```text
Dashboard
→ discover attention

Incoming Deliveries / Trip Detail
→ perform authorized action
```

# 24. Canonical Company Journeys

## Sender

```text
Create Trip
→ Receiver Request PENDING
→ Receiver Accepts
→ Publish
→ Driver Marketplace
→ Driver Claims
→ Delivery Progress
→ Completion
→ History
```

## Receiver

```text
Dashboard
→ Needs Attention
→ Receiver: Accept/Reject Delivery Request
→ Take Action
→ Incoming Deliveries
→ Pending Request
→ Accept / Reject
```

After Accept:

```text
Accepted
→ existing receiving workflow
→ Check-in when required
→ Completion when required
→ History
```

After Reject:

```text
Rejected
→ Publish blocked
→ Driver execution blocked
```

## Same-company

```text
Same authenticated Company
acts as Sender + Receiver
→ no external handshake
→ normal publication rules
→ Driver Marketplace
→ existing delivery lifecycle
```

# 25. Protected Existing Capabilities

The integrated blueprint does not replace:

- Company authentication and role model.
- Driver authentication and role model.
- Existing Trip operational lifecycle.
- Atomic Driver Claim.
- Receiver Check-in.
- Receiver Completion.
- Driver Completion.
- Existing delivery evidence.
- Public Share authorization.
- Reviewer operational workflow.
- Unified Trip Detail.
- Core Company navigation.

# 26. Integrated Approved Upgrades

The following are now part of the current intended Company product:

1. Receiver Delivery Request / handshake.
2. Receiver Accept.
3. Receiver Reject.
4. Server-side Publish gate.
5. Independent server-side Claim gate.
6. Pending Requests in Incoming Deliveries.
7. Dashboard Receiver Accept/Reject attention shortcut.
8. Sender/Receiver-aware Recent Completed messaging.
9. Sent/Received History labels.
10. All/Sent/Received History filters.
11. Explicit separation of Dashboard discovery from actual Receiver action.

# 27. Acceptance Checklist

### Navigation

- [x] Dashboard
- [x] My Created Trips
- [x] Incoming Deliveries
- [x] History / Timeline
- [x] Profile / Account

### Dashboard

- [x] Needs Attention is state-driven.
- [x] Pending Receiver Request appears as attention.
- [x] Take Action routes to `/company/incoming`.
- [x] Accept/Reject controls are not duplicated on Dashboard.
- [x] Existing Check-in/Completion attention remains intact.
- [x] No false Receiver Request alert appears without a PENDING request.

### Receiver Request

- [x] External request starts PENDING.
- [x] Receiver can Accept.
- [x] Receiver can Reject.
- [x] Unauthorized users cannot decide.
- [x] PENDING blocks Publish.
- [x] REJECTED blocks Publish.
- [x] ACCEPTED permits normal publication.
- [x] Claim independently checks acceptance.
- [x] Only one active PENDING request exists per Trip.

### History

- [x] All works.
- [x] Sent works.
- [x] Received works.
- [x] Sent/Received labels are visible.
- [x] Completed Trip Detail opens correctly.

### Existing lifecycle

- [x] Driver Claim remains atomic.
- [x] Receiver Check-in remains correct.
- [x] Receiver Completion remains correct.
- [x] Driver Completion remains correct.
- [x] Completion/history behavior remains correct.

### Security

- [x] Receiver-only Accept/Reject authorization.
- [x] Cross-tenant access blocked.
- [x] Server-side Publish gate.
- [x] Server-side Claim gate.
- [x] Public Share remains Receiver-only.

### Responsive

- [x] Desktop verified.
- [x] Intermediate/tablet verified.
- [x] Mobile verified.
- [x] No normal workflow requires horizontal scrolling.

# 28. Final Governance Position

This document is the consolidated current Company blueprint combining the original Company Locked Blueprint with all subsequently approved and implemented Company upgrades.

The historical `Company_Locked_Blueprint.md` remains preserved as the original baseline record. This integrated document is now the single current Company blueprint for the locked Company portal.

**Final status:** LOCKED  
**Lock authority:** Ayush  
**Lock basis:** Final System Audit completed with 142/142 requirements VERIFIED and verdict READY FOR COMPANY LOCK.
