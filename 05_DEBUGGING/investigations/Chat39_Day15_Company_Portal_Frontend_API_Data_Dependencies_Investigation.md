# Chat39 — Day 15 — Company Portal Frontend → API/Data Dependencies Investigation

## Investigation Purpose

Trace the existing Company frontend to its actual API, server-side, database, and event/data dependencies so we can establish what the Company portal currently reads, writes, and exposes.

This is an investigation only. Do not redesign the Company portal, change source code, change backend behavior, or create an implementation prompt.

## Critical Relationship Model Question

Explicitly investigate the distinction between a **Company entity** and the **company relationships attached to a particular Trip**.

The working question is:

> Are “Sending Company” and “Receiving Company” trip-specific relationships between existing Company entities, rather than permanent Company types/categories?

Do not assume this from naming alone. Trace the actual schema, foreign keys, frontend usage, API/server logic, and authorization/data filtering.

Verify specifically:

- What `company_id` represents for a trip.
- What `receiving_company_id` represents for a trip.
- Whether both reference the same `companies` entity/table.
- Whether a Company can be the sender on one trip and receiver on another trip.
- Whether there is any permanent sender/receiver classification stored on the Company itself.
- How these trip-specific relationships affect what a Company can see and do for each particular trip.

### Critical Public Share Question

Within that trip-specific relationship model, explicitly verify whether **only the Receiving Company for a particular completed trip** can create/manage its Public Share link, while the Sending Company for that same trip cannot.

Do not assume this from frontend rendering alone. Trace the actual authorization/data path and mark each conclusion VERIFIED / INFERRED / UNKNOWN.

## Required Investigation Areas

### 1. Company Dashboard Data Dependencies

Trace:

- Incoming Deliveries data source.
- Completed Deliveries data source.
- Delivery status/event data used to derive displayed status.
- Company identity and trip relationship used to filter trips.
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
- Which Company relationship is established by the operation.
- Ownership/authorization relationship used by the operation.

### 3. Receiver Check-in Dependencies

Trace:

- How the Company frontend identifies the relevant delivery.
- How driver arrival/check-in is recorded.
- API/server action/event/database dependency.
- Which trip/company relationship authorizes the action.
- How the resulting state becomes visible to the Company dashboard.

### 4. Delivery Completion Dependencies

Trace:

- How the receiving Company confirms completion.
- API/server action/event/database dependency.
- Authorization/identity checks.
- Which trip/company relationship authorizes the action.
- How completion becomes visible in Completed Deliveries.

### 5. Public Share Dependencies — Highest Priority

Trace the complete Public Share flow:

- Frontend component and rendering condition.
- Create Public Share API.
- Replace Public Share API.
- Revoke Public Share API.
- Database/storage records involved.
- Authorization checks.
- Relationship between authenticated Company and the particular trip.
- Whether the trip must be completed.
- Whether the authenticated Company must be the trip's Receiving Company.
- Whether the trip's Sending Company can create/manage the same share link.
- Exactly what information the public share exposes.

Required conclusion format:

```text
Company entity model: VERIFIED / INFERRED / UNKNOWN
Trip-specific Sending Company relationship: VERIFIED / INFERRED / UNKNOWN
Trip-specific Receiving Company relationship: VERIFIED / INFERRED / UNKNOWN
Can the same Company be sender on one trip and receiver on another: VERIFIED / INFERRED / UNKNOWN
Permanent sender/receiver Company classification exists: VERIFIED / INFERRED / UNKNOWN

For a particular completed Trip:
Sending Company → Public Share: VERIFIED / INFERRED / UNKNOWN
Receiving Company → Public Share: VERIFIED / INFERRED / UNKNOWN
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
- Company ↔ Trip relationship data used by each flow.

## Evidence Rules

- Use concrete source-code paths.
- Trace actual calls/queries/schema/authorization rather than relying on component names.
- Distinguish frontend visibility from backend authorization.
- Do not infer Sending vs Receiving Company authority from UI rendering alone.
- Do not treat Sending Company or Receiving Company as permanent Company types unless source evidence explicitly establishes such a model.
- Mark substantive findings VERIFIED / INFERRED / UNKNOWN.
- If the source does not establish a point, explicitly say UNKNOWN.

## Output

Update/create the investigation report with:

1. Executive finding.
2. Company entity vs trip-specific relationship model.
3. Company Dashboard dependencies.
4. Create/Publish dependencies.
5. Receiver Check-in dependencies.
6. Completion dependencies.
7. Public Share dependency and authorization analysis.
8. Sending Company vs Receiving Company conclusion for a particular trip.
9. Frontend → API/Data flow map.
10. VERIFIED / INFERRED / UNKNOWN confidence summary.

## Strict Boundaries

- No source-code changes.
- No API/backend changes.
- No database changes.
- No UI redesign.
- No implementation prompt.
- No final Company Mental Model decisions.
- Do not reopen or modify the locked Driver blueprint.
- Investigation only.
