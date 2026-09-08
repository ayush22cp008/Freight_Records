# Chat44 — Day 18 — Node 7 — Phase 1b — Company Trip Detail Evidence Visibility Investigation Instruction

## 1. Investigation Status

**INVESTIGATION AUTHORIZED — SOURCE INSPECTION ONLY**

**Portal:** Company  
**Node:** 7  
**Phase:** 1b  
**Scope:** Company Unified Trip Detail — Delivery Evidence visibility

This instruction is for source-level investigation only. **Do not modify source code, APIs, database schema, RLS, authentication, authorization, evidence storage, or product behavior.**

---

## 2. Investigation Objective

Determine whether the Company Unified Trip Detail can already display existing delivery evidence/photos using the current application data and authorization boundaries, or whether evidence visibility would require protected backend/security changes.

The key question is:

> **Does Company already have authorized access to existing evidence/photo data, with the current gap being frontend presentation only?**

Possible outcomes:

- **FRONTEND-ONLY:** Existing Company-accessible evidence data is available and the Trip Detail simply does not present it correctly.
- **PROTECTED-BACKEND BLOCKER:** Evidence data or authorization required by the Blueprint is not currently available to Company and would require API/RLS/database/security changes.
- **UNKNOWN:** The source does not provide enough evidence to determine the answer safely; stop and identify exactly what remains unknown.

Do not assume the outcome before inspecting the implementation.

---

## 3. Governing Records

Use these Records as the investigation basis:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
3. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`
4. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Trip_Detail_Investigation_Report.md`
5. `03_IMPLEMENTATION/implementation_reports/Chat43_Day17_Node7_Phase1b_Driver_P1_Fix_Implementation_Report.md`
6. `05_DEBUGGING/investigations/Chat39_Day15_Company_Portal_Final_Remaining_System_Presence_Investigation_Report.md`

The locked Company Blueprint requires Delivery Evidence in Unified Trip Detail and states that both participating companies should see relevant delivery evidence within existing authorization/data boundaries.

The Driver implementation provides an existing working reference for consuming `events.event_type` and `events.photo_url` without changing backend contracts.

---

## 4. Required Source Inspection

Inspect the actual `freight` source implementation and record exact file paths and relevant code behavior.

### Target A — Company Unified Trip Detail

Inspect:

- `src/app/(authenticated)/company/trips/[id]/page.tsx`
- Any components imported by this page that render events, timeline, evidence, trip progress, or driver information.

Determine:

- What data the page currently queries.
- Whether the query includes `events`.
- Which event fields are selected.
- Whether `photo_url` is selected.
- Whether evidence data is passed into a child component.
- Whether evidence is rendered at all.
- Whether evidence is intentionally omitted from the Company presentation.

### Target B — Existing Company event/evidence access

Search the Company frontend for:

- `photo_url`
- `events`
- `event_type`
- evidence-related components
- timeline/event rendering
- Supabase queries involving Company trips

Determine whether existing Company-accessible queries already retrieve evidence fields.

Do not change the queries during this investigation.

### Target C — Driver evidence implementation

Inspect the existing Driver implementation that was already verified to consume evidence:

- `src/app/(authenticated)/driver/active/page.tsx`
- Driver Timeline/event rendering components if applicable.

Determine exactly how the Driver frontend accesses and presents:

- `event_type`
- `photo_url`
- event timestamps
- GPS information
- evidence status

Use this only as a reference for existing data capability. Do not copy Driver-specific authorization or workflow assumptions into Company.

### Target D — Authorization / data boundary

Determine whether Company evidence visibility is already supported by the existing authorization/data model.

Specifically inspect existing Company trip/event access patterns and distinguish:

- Sending Company relationship: `trips.company_id`
- Receiving Company relationship: `trips.receiving_company_id`

Do not treat the existence of `photo_url` alone as proof that every Company can access every event. Establish the actual access path from source evidence.

Do not modify RLS or authorization logic.

### Target E — Public Share boundary

Inspect existing Public Share behavior only to confirm that public evidence exposure remains separate from authenticated Company evidence visibility.

Confirm whether Public Share intentionally excludes `photo_url` or other private evidence fields.

Do not expand Public Share exposure.

---

## 5. Specific Questions That Must Be Answered

### Q1 — Does the Company Trip Detail currently retrieve events?

Answer:
- YES / NO
- Exact source file and query.

### Q2 — Does it currently retrieve `photo_url`?

Answer:
- YES / NO
- Exact source file/query.

### Q3 — If `photo_url` is retrieved, is it rendered?

Answer:
- YES / NO / PARTIAL
- Exact component/path.

### Q4 — If it is not retrieved, does another existing Company-accessible query already provide it?

Answer:
- YES / NO / UNKNOWN
- Evidence required.

### Q5 — Can both Company relationships access relevant evidence using existing supported data access?

Evaluate separately:
- Sending Company
- Receiving Company

Do not infer authorization merely from UI visibility.

### Q6 — Is there an existing Company timeline/event surface that already contains the evidence information?

Answer:
- YES / NO / PARTIAL
- Identify exact route/component and fields.

### Q7 — Does the existing evidence model use `events.photo_url`?

Answer:
- YES / NO
- Cite the exact implementation evidence.

### Q8 — Would implementing Company Delivery Evidence require any of the following?

Check each explicitly:

- API contract change
- API response-shape change
- new API endpoint
- database schema change
- RLS/security policy change
- authentication/authorization change
- evidence-model change
- evidence-storage change
- new business rule

Expected result should be **NO** if the requirement can be fulfilled by existing Company-accessible data.

If any answer is YES or cannot be safely determined, classify it as a blocker/UNKNOWN and stop. Do not implement around it.

### Q9 — Is the Company evidence gap frontend-only?

Classify:

- **VERIFIED FRONTEND-ONLY**
- **VERIFIED PROTECTED-BACKEND DEPENDENCY**
- **UNKNOWN**

### Q10 — What is the narrowest safe next action?

Choose one:

1. Create a frontend-only implementation decision/prompt for Company Trip Detail evidence.
2. Investigate a specific protected dependency before implementation.
3. No change required because evidence is already correctly implemented.
4. Stop because required evidence capability is unavailable or authorization is unresolved.

---

## 6. Protected Boundaries — DO NOT CHANGE

The following are strictly protected during this investigation:

- API contracts
- API response shapes
- API routes
- database schema
- `events` table structure
- `events.photo_url` evidence model
- photo upload/storage behavior
- RLS policies
- authentication
- authorization policy logic
- role assignment
- trip lifecycle semantics
- claiming/marketplace behavior
- evidence requirements/integrity rules
- persistent workflow state
- AI behavior
- Reviewer authority/workflow
- Driver Portal behavior
- Public Share authorization/projection

### C-05

`src/app/api/completion/route.ts` is explicitly protected.

Do not modify it or alter its response contract during this investigation.

---

## 7. Company Blueprint Requirements Being Verified

The locked Company Blueprint requires Unified Trip Detail to present:

1. Current Status
2. Visual Delivery Progress
3. Next Required Action
4. Driver / Claim Information
5. Trip Details
6. Delivery Evidence
7. Timeline / History

It also establishes that:

- Sender and Receiver share the same core Trip Detail structure.
- Relationship-specific information/actions vary by authorization/state.
- Both participating companies can see relevant delivery evidence within existing authorization/data boundaries.
- Public Share remains Receiving Company-only.

The investigation must verify the implementation path for these evidence requirements without expanding the product boundary.

---

## 8. Evidence Collection Requirements

The investigation report must include:

- Exact files inspected.
- Exact relevant routes/components.
- Existing event/evidence query behavior.
- Whether `photo_url` is selected.
- Whether `photo_url` is rendered.
- Company sender/receiver access findings.
- Driver reference implementation findings.
- Public Share separation findings.
- Protected-boundary assessment.
- Clear VERIFIED / INFERRED / UNKNOWN classification.

Do not claim manual UI behavior as verified unless it was actually observed and documented.

Source inspection findings must be distinguished from inference.

---

## 9. Investigation Report Output

Create the investigation report at:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Trip_Detail_Evidence_Visibility_Investigation_Report.md`

The report must contain at minimum:

```text
1. Investigation Status
2. Executive Result
3. Source Files Inspected
4. Company Trip Detail Evidence Path
5. Existing Evidence Data Availability
6. Sender vs Receiver Access Assessment
7. Driver Reference Comparison
8. Public Share Boundary
9. Protected-Boundary Assessment
10. Root Cause / Classification
11. VERIFIED / INFERRED / UNKNOWN Summary
12. Recommended Next Action
13. Explicit Implementation Authorization Status
```

The report must state clearly:

**NO SOURCE CHANGES MADE DURING THIS INVESTIGATION.**

---

## 10. Stop Conditions

Stop the investigation and record the blocker if:

- Evidence requires a new API.
- Existing API response shape appears insufficient and would need modification.
- RLS/security policy must change.
- Company authorization is unclear.
- Evidence data is not accessible through existing supported capabilities.
- A new evidence type or persistence mechanism appears necessary.
- Public Share would need to expose private evidence.
- Driver behavior would need to change.
- The locked Company Blueprint contradicts actual protected system behavior.

Do not solve a protected-boundary problem by assumption.

---

## 11. Final Decision Rule

The investigation is successful only when it can answer, with source evidence:

> **Can Company Unified Trip Detail display the existing delivery evidence/photos using the existing data and authorization model, without changing backend/API/database/RLS/security behavior?**

If **YES**, the next step is a narrowly scoped frontend-only implementation decision/prompt.

If **NO**, identify the exact protected dependency and stop.

If **UNKNOWN**, do not implement; identify the missing evidence needed to resolve the uncertainty.

---

## 12. Execution Authorization

This document authorizes **investigation only**.

It does **not** authorize source-code implementation.

After the report is produced, return to the ChatGPT architecture/reasoning workflow for review and next-step authorization.
