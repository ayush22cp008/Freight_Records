# Chat44 — Day 18 — Node 7 — Phase 1b
## Company Active vs Completed Trip Investigation Instruction

**Status:** INVESTIGATION ONLY — NO SOURCE CHANGES AUTHORIZED
**Portal:** Company
**Phase:** 1b — Frontend Implementation / Verification
**Finding:** Company Active Created Trips / My Created Trips are presenting completed trips as active/current work.

---

## 1. Purpose

Investigate the Company portal behavior observed during Ayush manual verification:

- Dashboard → **Active Created Trips** shows created trips with `CLAIMED`, while the Company portal also contains completed trips elsewhere.
- **My Created Trips** contains both an active/claimed trip and multiple `COMPLETED` trips.
- The same completed trips are correctly visible in **History & Timeline**.
- Ayush's concern is that completed trips should not be presented as active/current trips in the Dashboard's **Active Created Trips** surface, and My Created Trips should clearly distinguish active/current created trips from completed historical trips.

This instruction is for evidence gathering and root-cause analysis only. Do not implement a fix in this task.

---

## 2. Source-of-Truth Documents

Review before investigation:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
3. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
4. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Manual_Verification_Findings_Trip_Detail.md` (if present)
5. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`
6. `03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Implementation_Report.md`

Records repository is the project bridge/source of truth for architecture, decisions, investigations, and implementation state.

---

## 3. Locked Boundary

This investigation must remain strictly inside the locked Company frontend boundary.

### Allowed to inspect

- Company Dashboard data selection/filtering and presentation.
- My Created Trips data selection/filtering and presentation.
- History data selection/filtering and presentation.
- Existing frontend route/component logic.
- Existing API calls and their response usage, for diagnosis only.
- Existing lifecycle/status fields as they are currently consumed by the frontend.
- Whether the frontend has a defined distinction between active/current and completed trips.

### Protected — do not modify

- Database/schema.
- RLS.
- Authentication/authorization policy logic.
- Role assignment.
- Backend business rules.
- Lifecycle semantics/state transitions.
- Claiming/marketplace behavior.
- Evidence requirements/integrity.
- AI behavior.
- Reviewer authority.
- API contracts or response shapes.
- `src/app/api/completion/route.ts` and the C-05 response contract.

If diagnosis indicates that the desired behavior requires any protected change, STOP and report it as a boundary escalation. Do not implement it.

---

## 4. Questions to Answer

Determine from the actual source code and runtime behavior:

### Q1 — Dashboard Active Created Trips

- What exact API/data source populates `Active Created Trips`?
- What exact filter determines which trips are included?
- Does the filter explicitly exclude `completed` trips?
- If not, why are completed trips considered active by the current implementation?
- Is the Dashboard displaying a bounded subset of `My Created Trips`, or using separate logic?

### Q2 — My Created Trips

- What exact API/data source populates `/company/created`?
- Does it intentionally represent all company-created trips, including completed trips?
- If it is intended to be a full created-trip history, is the issue only presentation/separation rather than data inclusion?
- Does the locked Company Blueprint require completed trips to be removed from this surface, or merely clearly separated from active/current trips?

### Q3 — History

- Confirm how `/company/history` obtains completed trips.
- Confirm whether the same completed trips appearing in History is expected.
- Do not treat duplicate underlying records across active/history views as a defect by itself; determine whether the issue is incorrect active classification/presentation.

### Q4 — Status semantics

- Identify the exact frontend status fields used by Dashboard and My Created Trips.
- Distinguish `trip.status` from operational events/timeline events.
- Do not redefine lifecycle semantics.
- Confirm whether `COMPLETED` is already a valid lifecycle value used by the existing system.

### Q5 — Minimal frontend correction

Based on evidence, identify the smallest frontend-only correction that would align the surfaces with the locked Company Blueprint.

Do not implement that correction in this investigation task.

---

## 5. Required Evidence

Capture exact source evidence including:

- File paths.
- Relevant component/page names.
- Relevant query/filter expressions.
- Relevant status comparisons.
- Relevant route relationships.
- Any shared component involved.

Use `VERIFIED`, `INFERRED`, and `UNKNOWN` explicitly.

Do not infer a backend defect merely because a frontend list contains an unexpected record.

---

## 6. Required Root-Cause Classification

Classify the finding as one of:

1. **Frontend filtering defect** — existing data is correct but the Company surface includes completed trips in an active list.
2. **Frontend information-architecture/presentation issue** — My Created Trips intentionally includes historical created trips but does not distinguish active/current from completed clearly enough.
3. **Backend/data contract issue** — only if evidence proves the frontend cannot obtain the necessary distinction without a protected backend/API change. If this occurs, STOP and escalate; do not modify backend code.
4. **No defect / expected behavior** — only if the locked Company Blueprint and actual intended semantics explicitly support the observed behavior.
5. **UNKNOWN** — if evidence is insufficient.

Multiple classifications may apply to different surfaces (for example, Dashboard vs My Created Trips).

---

## 7. Explicit Non-Goals

Do NOT:

- change source code;
- change database data;
- change Supabase schema/RLS;
- change API routes/contracts;
- change lifecycle/state transition rules;
- change claiming behavior;
- change Driver Portal behavior;
- change Reviewer behavior;
- alter C-05;
- redesign the Company Blueprint;
- create new product behavior beyond what is already specified in the locked Blueprint.

---

## 8. Output Report

Create the investigation report at:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Active_vs_Completed_Trip_Investigation_Report.md`

The report must contain:

1. Investigation status.
2. Observation.
3. Reproduction path.
4. Source files inspected.
5. Dashboard evidence.
6. My Created Trips evidence.
7. History evidence.
8. Status/lifecycle evidence.
9. Root cause.
10. Classification per surface.
11. Minimal frontend-only fix recommendation, without implementing it.
12. Protected-boundary assessment.
13. Verification plan for the later fix.
14. Evidence labels: VERIFIED / INFERRED / UNKNOWN.
15. Final recommendation: FIX / NO FIX / STOP & ESCALATE.

The report must clearly distinguish:

- "completed trips should remain available in History" from
- "completed trips should not appear as active/current work".

---

## 9. Stop Conditions

STOP immediately and report if:

- a backend change appears necessary;
- an API contract/response shape must change;
- database/schema/RLS changes appear necessary;
- lifecycle semantics must change;
- claiming/marketplace rules must change;
- authentication/authorization logic must change;
- evidence integrity must change;
- shared Driver behavior would be affected;
- the locked Company Blueprint contradicts the observed requirement;
- the required status distinction cannot be established from existing data/capabilities.

Do not work around a protected boundary.

---

## 10. Authorization

This instruction authorizes **investigation only**.

It does **not** authorize:

- source implementation;
- bug fixing;
- refactoring beyond inspection;
- database/backend changes;
- deployment.

After the report is complete, stop and wait for Ayush/ChatGPT review and explicit authorization for any subsequent fix.
