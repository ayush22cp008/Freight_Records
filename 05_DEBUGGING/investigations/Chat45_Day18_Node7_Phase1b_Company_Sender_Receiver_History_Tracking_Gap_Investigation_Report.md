# Chat45 — Day 18 — Node 7 Phase 1b — Company Sender / Receiver History & Tracking Gap Investigation Report

## 1. Investigation Status

**Status:** INVESTIGATION COMPLETE — DECISION REQUIRED BEFORE IMPLEMENTATION

**Node:** Node 7 — AI + Final Integration + Demo

**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign

**Portal:** Company Portal

**Investigation type:** Company relationship visibility, completed-trip history separation, and received-trip discoverability

**Scope:** Frontend/UI/UX only unless source verification proves that required information is unavailable from existing data/contracts.

**Backend/API/DB changes:** None proposed or authorized by this investigation.

---

## 2. Problem Statement

A Company account represents one company identity, but that company can participate in different trips in different relationships:

- **Sender Company:** the company that created/sent the trip.
- **Receiver Company:** the company receiving the delivery.

The same Company account may be the sender on one trip and the receiver on another trip. Therefore, Company UI must not present completed trips or completion notifications in a way that makes the relationship ambiguous.

Two related gaps were identified:

1. The `Recently Completed` discovery currently needs to distinguish whether the completed trip was **sent by this company** or **received by this company**.
2. Company History currently combines completed trips where the company was either sender or receiver, without an explicit relationship indicator/filter.

A third concern was raised during the same review: whether the Company actually tracks deliveries sent by other companies to this company. Source inspection was required to distinguish a true data/tracking gap from a discoverability/UI gap.

---

## 3. Investigation Questions

1. Does the current Company system retrieve completed trips where the Company is the sender?
2. Does it retrieve completed trips where the Company is the receiver?
3. Can the UI determine whether a completed trip is **Sent** or **Received** using existing trip fields?
4. Does Company History currently mix both relationships together?
5. Does Company have an active surface for trips sent by other companies to this Company?
6. Would fixing these issues require backend/database/API changes, or can the existing frontend data be used?
7. What should be locked before implementation?

---

## 4. Source-Level Evidence

### 4.1 Company Dashboard completed-trip retrieval

Verified source:

`src/app/(authenticated)/page.tsx`

The Dashboard retrieves recent completed trips using both relationship fields:

- `company_id = company.id`
- `receiving_company_id = company.id`

The query therefore includes completed trips in which the current Company participated either as sender or receiver. It is limited to the five most recent completed trips.

**Finding:** Completed received trips are not absent from the underlying Company completed-trip query.

**Confidence:** VERIFIED

### 4.2 Company Incoming Deliveries

Verified source:

`src/app/(authenticated)/company/incoming/page.tsx`

The Incoming Deliveries page queries trips using:

`receiving_company_id = company.id`

and includes active lifecycle states (`active`, `claimed`, `in_progress`). It derives receiver-facing workflow states and actions such as receiver check-in, confirmation, and waiting for driver completion.

**Finding:** A Company already has a dedicated receiver-side active tracking surface for deliveries sent by another Company.

**Confidence:** VERIFIED

### 4.3 Company Created Trips

Verified source:

`src/app/(authenticated)/company/created/page.tsx`

The My Created Trips page queries:

`company_id = company.id`

and presents trips created by the current Company. Completed trips are excluded from the active presentation.

**Finding:** Sender-side active tracking is represented separately from receiver-side Incoming Deliveries.

**Confidence:** VERIFIED

### 4.4 Company History

Verified source:

`src/app/(authenticated)/company/history/page.tsx`

History queries completed trips where either:

`company_id = company.id`

or

`receiving_company_id = company.id`

The current card presentation shows the route and `Completed` status, but does not explicitly identify the Company's relationship to the trip as **Sent** or **Received**.

**Finding:** History is currently a combined completed-trip list without a visible sender/receiver distinction.

**Confidence:** VERIFIED

---

## 5. Root Cause

The issue is primarily a **relationship-context presentation gap**, not an absence of trip participation data.

The existing trip model already distinguishes the two Company relationships through:

- `company_id` — creating/sending Company
- `receiving_company_id` — receiving Company

The frontend already uses those fields for separate active surfaces and for the combined completed-trip query.

However, the combined completed-trip surfaces do not consistently expose the relationship to the user.

Therefore a Company can see a completed trip but may not immediately know:

> "Was this something I sent, or something I received?"

The same ambiguity can occur in the temporary `Recently Completed` discovery.

**Root cause classification:** Frontend information architecture / relationship labeling gap.

**Confidence:** VERIFIED for the current source behavior; exact final UI implementation remains a design decision.

---

## 6. Important Clarification — Received Trips Are Already Tracked

The concern that the Company does not track a trip sent by another Company is **not confirmed as a data/tracking failure**.

The current system already has an Incoming Deliveries surface scoped to `receiving_company_id`, so active received deliveries are represented there.

The current completed-trip query also includes `receiving_company_id`, so received trips can enter Company completed-history/discovery data as well.

The actual weakness is that the Company experience does not make the two relationships sufficiently explicit and unified across the relevant views.

**Conclusion:** This should not be solved by inventing a second tracking model or changing the trip lifecycle. The first solution should be frontend relationship-aware presentation using the existing trip data.

**Confidence:** VERIFIED

---

## 7. Required UX Mental Model

The Company remains **one account**.

The account has two possible trip relationships:

```text
COMPANY ACCOUNT
│
├── SENT
│   └── Trips where company_id = current company
│
└── RECEIVED
    └── Trips where receiving_company_id = current company
```

The UI must make this relationship visible without creating duplicate Company accounts or changing the underlying trip lifecycle.

---

## 8. Recently Completed Requirement

The existing generic wording:

> Recently Completed
>
> Your recent trip is finished
>
> Delivery from baroda has been fully completed.

is insufficient when the same Company can be sender for one trip and receiver for another.

The notification should identify the Company's relationship.

### Sender example

```text
Recently Completed

Your recent sent trip is finished

Your delivery from baroda to gandevi has been fully completed.

[View Completed Trip]
```

### Receiver example

```text
Recently Completed

Your recent received delivery is finished

A delivery from baroda to gandevi has been fully completed.

[View Completed Trip]
```

Exact copy remains subject to implementation review, but the relationship distinction is required.

**Decision:** REQUIRED

---

## 9. History Requirement

Company History should remain a company-wide completed-trip history, but every trip must make the Company's relationship clear.

Recommended conceptual presentation:

```text
History & Timeline

All | Sent | Received

--------------------------------
SENT
baroda → gandevi
Completed

--------------------------------
RECEIVED
madhyapradesh → surat
Completed
```

The relationship should be derived from the existing Company/trip relationship, not from a newly invented database field unless later source verification proves the existing fields are insufficient.

The `All / Sent / Received` filtering is a UX recommendation to make a combined history useful while preserving a single history destination.

**Decision:** REQUIRED relationship distinction; filter behavior should be included in implementation if it can be achieved with the existing fetched data without changing protected backend behavior.

---

## 10. Active Tracking Requirement

The Company must be able to understand both categories of active work:

### Sent

Trips created by the Company remain represented through:

`My Created Trips`

### Received

Trips sent by another Company remain represented through:

`Incoming Deliveries`

This separation is consistent with the locked Company blueprint:

- `My Created Trips` represents sender-side responsibility.
- `Incoming Deliveries` is the receiver action inbox.

Incoming Deliveries must **not** become a second generic progress dashboard.

**Decision:** Preserve this separation.

---

## 11. Unified Company History Model

The intended Company information architecture is:

```text
Dashboard
│
├── Recently Completed
│   ├── Sent completion
│   └── Received completion
│
├── Needs Attention
│   └── Receiver actions when applicable
│
├── Active Created Trips
│
└── Quick Access
    ├── My Created Trips
    ├── Incoming Deliveries
    └── History

My Created Trips
└── Sent / created active trips

Incoming Deliveries
└── Received / receiver action inbox

History
└── All completed participation
    ├── Sent
    └── Received
```

A single Company account therefore has complete visibility without duplicating trip records.

---

## 12. Completed Trip Detail

The existing Company Trip Detail route accepts the exact trip ID and is the correct destination for both sender and receiver completed-trip viewing.

The relationship context should be visible in Trip Detail as well where appropriate, so that a user entering the same trip from either discovery surface can understand whether the Company participated as sender or receiver.

The exact Trip Detail hierarchy remains locked:

1. Current Status
2. Visual Delivery Progress
3. Next Required Action
4. Driver / Claim Information
5. Trip Details
6. Delivery Evidence
7. Timeline / History

No new lifecycle state is required for this distinction.

---

## 13. Interaction With Completion Acknowledgement

The previously investigated Company completion discovery uses a trip-specific temporary acknowledgement concept.

The acknowledgement must remain tied to the exact trip ID and must dismiss temporary discovery only. It must not mutate lifecycle state.

The Sender/Receiver relationship distinction should be part of the displayed completion discovery, but it must not create separate lifecycle or acknowledgement systems for sender and receiver.

Conceptually:

```text
same trip ID
    ↓
relationship-aware label
    ↓
View Completed Trip
    ↓
exact Company Trip Detail
    ↓
temporary discovery acknowledged
```

---

## 14. Backend Boundary

No backend change is justified by the evidence currently available.

Existing fields are sufficient to determine the relationship:

```text
if trip.company_id === currentCompany.id
    → SENT

if trip.receiving_company_id === currentCompany.id
    → RECEIVED
```

For a valid trip in the Company context, these relationships provide the required distinction.

The implementation must not modify:

- database schema
- RLS/security
- trip lifecycle/state semantics
- claiming behavior
- receiver confirmation behavior
- driver confirmation behavior
- completion semantics
- API contracts
- evidence requirements
- reviewer authority

If implementation discovers that a required UI value cannot be obtained from existing frontend data/contracts, stop and classify that information as UNKNOWN rather than inventing a backend change.

---

## 15. Implementation Scope Recommendation

### In scope

- Make Company completion discovery relationship-aware.
- Distinguish **Sent** versus **Received** in Company completed-trip presentation.
- Improve History information architecture so the relationship is immediately visible.
- Add Sent/Received filtering to History if supported entirely through existing data.
- Preserve separate My Created Trips and Incoming Deliveries responsibilities.
- Preserve exact trip-ID routing to Company Trip Detail.
- Preserve one-time temporary completion discovery acknowledgement.
- Ensure responsive behavior remains consistent with the locked Company design system.

### Out of scope

- New backend functionality.
- New database columns.
- New API contracts.
- New trip states.
- Changes to completion lifecycle.
- Changes to claiming/marketplace behavior.
- Changes to RLS/security.
- Changes to evidence requirements.
- Changes to Reviewer authority.
- Creating a separate Company account for sender and receiver roles.

---

## 16. Acceptance Criteria for the Future Fix

### AC-01 — Sender completion distinction

When the current Company is the sender of a newly completed trip, the temporary completion discovery clearly communicates that the trip was **sent** by the Company.

### AC-02 — Receiver completion distinction

When the current Company is the receiver of a newly completed trip, the temporary completion discovery clearly communicates that the delivery was **received** by the Company.

### AC-03 — Same account, different relationships

The same Company account can show one completed trip as Sent and another completed trip as Received without ambiguity.

### AC-04 — History includes both relationships

Company History continues to include completed trips where the Company was sender or receiver.

### AC-05 — History relationship visibility

Every History trip visibly identifies whether it was Sent or Received.

### AC-06 — History filtering

If implemented, `All`, `Sent`, and `Received` views correctly filter the already available completed-trip collection without changing backend semantics.

### AC-07 — Received active tracking preserved

A delivery created by another Company and addressed to the current Company remains visible in Incoming Deliveries while active.

### AC-08 — Created active tracking preserved

A trip created by the current Company remains visible in My Created Trips while active.

### AC-09 — No duplicate lifecycle representation

The same trip is not duplicated into separate underlying trip records or given a new lifecycle state merely to support sender/receiver presentation.

### AC-10 — Exact Trip Detail routing

Opening a completed trip always reaches the exact trip's Company Trip Detail.

### AC-11 — Acknowledgement semantics preserved

Viewing the completed trip consumes only the temporary completion discovery for that exact trip; it does not alter database lifecycle state.

### AC-12 — Protected boundaries preserved

No backend/API/DB/RLS/business-rule changes are introduced by the frontend implementation.

---

## 17. Evidence / Confidence Summary

| Finding | Confidence |
|---|---|
| Dashboard completed query includes sender and receiver trips | VERIFIED |
| Incoming Deliveries tracks receiver-side active trips | VERIFIED |
| My Created Trips tracks sender-side active trips | VERIFIED |
| History combines sender and receiver completed trips | VERIFIED |
| History currently lacks explicit Sent/Received distinction | VERIFIED |
| Current system lacks all received-trip tracking | NOT CONFIRMED / CONTRADICTED BY SOURCE |
| Sender/Receiver distinction can be derived from existing trip relationship fields | VERIFIED |
| Backend change is required | NOT SUPPORTED BY EVIDENCE |
| Exact final copy for notifications | DESIGN DECISION |
| Exact History visual treatment | DESIGN DECISION |

---

## 18. Final Investigation Decision

**Decision:** Treat this as a **Company Portal frontend information-architecture and relationship-visibility refinement**, not as a backend tracking redesign.

The Company already has the underlying relationship information and separate active sender/receiver surfaces. The missing piece is a clear, consistent way for one Company account to understand whether a trip was **Sent** or **Received**, especially in completion discovery and History.

The future implementation should therefore:

1. Keep one Company account.
2. Keep existing sender and receiver relationship fields.
3. Keep My Created Trips as the sender-side active view.
4. Keep Incoming Deliveries as the receiver action inbox.
5. Make Recently Completed relationship-aware.
6. Make History relationship-aware.
7. Prefer `All | Sent | Received` filtering in History when possible without backend changes.
8. Keep exact trip-specific routing and acknowledgement behavior.
9. Make no protected backend/product changes.

**Investigation outcome:** READY FOR IMPLEMENTATION PREPARATION, subject to the active brain's implementation handoff and Ayush approval.

---

## 19. Source Verification References

The following source files were inspected during this investigation:

- `src/app/(authenticated)/page.tsx`
- `src/app/(authenticated)/company/incoming/page.tsx`
- `src/app/(authenticated)/company/created/page.tsx`
- `src/app/(authenticated)/company/history/page.tsx`

Relevant verified behavior includes the use of both `company_id` and `receiving_company_id` for Company completed-trip retrieval, receiver-side active filtering through `receiving_company_id`, and sender-side active filtering through `company_id`.

---

**Record:** Chat45 / Day18 / Node7 / Phase1b / Company Portal
