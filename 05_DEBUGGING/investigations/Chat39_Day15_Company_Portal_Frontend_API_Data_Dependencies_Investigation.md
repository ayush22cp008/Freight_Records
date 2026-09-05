# Chat39 — Day 15 — Company Portal Frontend → API/Data Dependencies Investigation

## Investigation Purpose

Trace the existing Company frontend to its actual API, server-side, database, and event/data dependencies so we can establish what the Company portal currently reads, writes, and exposes.

This is an investigation only. Do not redesign the Company portal, change source code, change backend behavior, or create an implementation prompt.

## Critical Authorization Question

Explicitly verify the distinction between:

- **Sending Company** — the Company that created/sent the trip/parcel.
- **Receiving Company** — the Company that receives the parcel/delivery.

A key question from the current discussion is whether **only the Receiving Company** can create/manage a Public Share link for a particular completed received delivery, while the Sending Company cannot.

Do not assume this from frontend rendering alone. Trace the actual authorization/data path and mark the conclusion VERIFIED / INFERRED / UNKNOWN.

## Required Investigation Areas

### 1. Company Dashboard Data Dependencies

Trace:

- Incoming Deliveries data source.
- Completed Deliveries data source.
- Delivery status/event data used to derive displayed status.
- Company identity/relationship used to filter trips.
- Any frontend API calls, server actions, Supabase/database queries, or other data sources.

Record source paths and evidence.

### 2. Create Trip / Publish Trip Dependencies

Trace the existing Company trip creation workflow:

- Draft creation.
- Draft persistence.
- Review/readback.
- Publish operation.
- Data fields sent/received.
- APIs/server actions/database tables involved.
- Company ownership/identity relationship used by the operation.

### 3. Receiver Check-in Dependencies

Trace:

- How the Company frontend identifies the relevant delivery.
- How driver arrival/check-in is recorded.
- API/server action/event/database dependency.
- How the resulting state becomes visible to the Company dashboard.

### 4. Delivery Completion Dependencies

Trace:

- How the receiving Company confirms completion.
- API/server action/event/database dependency.
- Authorization/identity checks.
- How completion becomes visible in Completed Deliveries.

### 5. Public Share Dependencies — Highest Priority

Trace the complete Public Share flow:

- Frontend component and rendering condition.
- Create Public Share API.
- Replace Public Share API.
- Revoke Public Share API.
- Database/storage records involved.
- Authorization checks.
- Relationship between authenticated Company and trip.
- Whether the trip must be completed.
- Whether the authenticated Company must be the **Receiving Company**.
- Whether the **Sending Company** can create/manage the same share link.
- Exactly what information the public share exposes.

Required conclusion format:

```text
Sending Company → Public Share for completed received delivery: VERIFIED / INFERRED / UNKNOWN
Receiving Company → Public Share for completed received delivery: VERIFIED / INFERRED / UNKNOWN
Reason: source-backed explanation
```

### 6. Frontend → API/Data Flow Map

Produce a concise mapping such as:

```text
Company UI
  ↓
Frontend component/page
  ↓
API / server action / query
  ↓
Database / event source
  ↓
Displayed or mutated data
```

Cover at minimum:

- Dashboard incoming deliveries.
- Dashboard completed deliveries.
- Create trip.
- Publish trip.
- Receiver check-in.
- Delivery completion.
- Public Share creation/management.

## Evidence Rules

- Use concrete source-code paths.
- Trace actual calls/queries/authorization rather than relying on component names.
- Distinguish frontend visibility from backend authorization.
- Do not infer Sending vs Receiving Company authority from UI rendering alone.
- Mark substantive findings VERIFIED / INFERRED / UNKNOWN.
- If the source does not establish a point, explicitly say UNKNOWN.

## Output

Update/create the investigation report with:

1. Executive finding.
2. Company Dashboard dependencies.
3. Create/Publish dependencies.
4. Receiver Check-in dependencies.
5. Completion dependencies.
6. Public Share dependency and authorization analysis.
7. Sending Company vs Receiving Company conclusion.
8. Frontend → API/Data flow map.
9. VERIFIED / INFERRED / UNKNOWN confidence summary.

## Strict Boundaries

- No source-code changes.
- No API/backend changes.
- No database changes.
- No UI redesign.
- No implementation prompt.
- No final Company Mental Model decisions.
- Do not reopen or modify the locked Driver blueprint.
- Investigation only.
