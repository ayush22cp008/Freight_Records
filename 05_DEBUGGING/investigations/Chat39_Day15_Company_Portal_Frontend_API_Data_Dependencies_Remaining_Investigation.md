# Chat39 Day15 — Company Portal Frontend → API/Data Dependencies — Remaining Investigation

## Purpose

Complete only the remaining unknowns from the Company Portal Existing Frontend Structure / Frontend → API/Data Dependencies investigation. This is a focused follow-up investigation and must not duplicate or reopen findings already verified in the existing Section 2 investigation.

## Already Verified — Preserve, Do Not Re-investigate

The following findings are already VERIFIED and must be treated as established context:

1. The system has many Company entities.
2. `companies` is a generic Company entity; it has no permanent sender/receiver classification.
3. `trips.company_id` identifies the Company associated with creating/sending that particular trip.
4. `trips.receiving_company_id` identifies the receiving Company for that particular trip.
5. Sender/Receiver are trip-specific relationships. The same Company can be sender on one trip and receiver on another.
6. Public Share management is authorized for the receiving Company of that specific trip; the sending Company cannot manage that trip's Public Share.
7. Incoming Deliveries and Completed Deliveries are filtered through `receiving_company_id`.
8. Create Trip sets the creator Company as `company_id` and uses the selected receiving Company as `receiving_company_id`.
9. Publish requires the acting Company to own the trip through `company_id`.
10. The current frontend/shared structure findings from the preceding investigation remain unchanged.

Do not repeat these investigations unless new evidence directly contradicts them. If a remaining flow depends on one of these relationships, reference the established finding rather than re-proving it.

---

## Investigation Scope

### 1. Receiver Check-in → API/Data Dependency

Inspect the existing Company-side Receiver Check-in flow and establish with source evidence:

- Exact Company frontend entry point/component.
- Exact UI action that initiates receiver check-in.
- Exact API route(s) called.
- Request payload / parameters sent by the frontend.
- Backend authorization checks.
- Which Company identity is used and how it is derived.
- Which trip relationship is required (`receiving_company_id`, `company_id`, or another source of truth).
- Database records read/written.
- Event(s) created or updated, including exact event type where available.
- Status/state changes, if any.
- What data the frontend uses to display the resulting state.
- Whether the flow has any frontend/backend mismatch, dead path, or discoverability issue relevant to Phase 1b.

Do not redesign or modify the flow.

### 2. Delivery Completion → API/Data Dependency

Inspect the existing Company-side Receiver Completion flow and establish with source evidence:

- Exact Company frontend entry point/component.
- Exact UI action that initiates completion.
- Exact API route(s) called.
- Request payload / parameters sent by the frontend.
- Backend authorization checks.
- Which Company identity is used and how it is derived.
- Which trip relationship is required.
- Database records read/written.
- Evidence/receipt requirements, if any.
- Event(s) created or updated, including exact event type where available.
- Status transition(s), especially the path to `completed`.
- What data the frontend uses to display the resulting state.
- Whether the flow has any frontend/backend mismatch, dead path, or discoverability issue relevant to Phase 1b.

Do not redesign or modify the flow.

### 3. Exact Public Share Exposed Data

Inspect the complete Public Share flow and determine exactly what a public recipient can see for a shared trip.

Establish with source evidence:

- Public Share creation/activation path.
- Public Share lookup/view route(s).
- Exact authorization boundary for managing the share (preserve the already-verified receiving-Company rule).
- Exact public-facing response/data fields exposed.
- Whether driver/company identities, trip details, location/progress, timestamps, evidence, events, or other data are exposed.
- Whether sensitive/internal-only fields are excluded.
- Whether the public view is read-only.
- Any frontend components/pages that render the public data.
- Any mismatch between what Company UI implies and what Public Share actually exposes.

Do not change public-share behavior.

### 4. Complete Frontend → API → Database/Event Flow Map

Using the evidence gathered in Sections 1–3 and the already-verified Section 2 findings, construct a concise end-to-end Company Portal dependency map covering at minimum:

- Dashboard / Company overview.
- Create Trip.
- Publish Trip.
- Incoming Deliveries.
- Receiver Check-in.
- Receiver Completion.
- Completed Deliveries.
- Public Share management.
- Public Share viewing, where applicable.

For each flow, identify:

`Frontend entry → API route → authorization identity/relationship → database source/write → event/status effect → frontend-visible result`

Mark any unavailable link explicitly as UNKNOWN rather than inferring it.

### 5. Final Evidence Classification

Provide a final table or equivalent structured summary using:

- VERIFIED — directly supported by source-code/schema evidence.
- INFERRED — reasonable interpretation not directly proven by source evidence.
- UNKNOWN — not established by the available evidence.

Separate confirmed facts from interpretation. Do not silently resolve UNKNOWN items using general assumptions.

### 6. Phase 1b Relevance

At the end, identify only the frontend-structure/data-dependency findings that materially affect the upcoming Company Portal blueprint. Focus on:

- Current navigation/entry-point implications.
- Current visibility/state presentation implications.
- Discoverability problems.
- Existing source-of-truth constraints.
- Existing frontend/backend mismatches that the redesign must faithfully account for.

Do not propose a redesign yet. The purpose is evidence collection for the later Company Mental Model and Interaction Mapping decisions.

---

## Explicit Non-Scope

Do NOT:

- modify source code;
- modify database/schema;
- modify API behavior;
- create an implementation prompt;
- redesign Company Portal UI;
- lock the Company Mental Model;
- lock the Company Portal Blueprint;
- reopen completed Nodes 1–6;
- reopen the already-verified trip-specific Company relationship model;
- create duplicate investigations for findings already covered by the existing Section 2 report.

## Required Investigation Method

Follow:

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION/IMPLICATION`

Use exact source paths, routes, component names, schema/table names, and relevant code references where available.

Every substantive conclusion must carry a confidence classification: VERIFIED / INFERRED / UNKNOWN.

## Completion Condition

This investigation is complete only when Sections 1–6 above are addressed with evidence, or when an item is explicitly marked UNKNOWN with the reason the evidence could not establish it.

The output should be a separate investigation report suitable for the Records repo and later use in the Company Portal blueprint sequence.
