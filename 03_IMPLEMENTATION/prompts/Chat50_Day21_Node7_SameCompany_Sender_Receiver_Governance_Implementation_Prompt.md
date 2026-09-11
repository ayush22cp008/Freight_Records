# Chat50 — Day21 — Node 7 — Same-Company Sender/Receiver Governance Implementation Prompt

## Status

- Governance decision: **APPROVED BY AYUSH**
- Implementation authorization: **GRANTED by Ayush through this implementation handoff request**
- Scope: **NEW Trip creation only**
- Legacy same-company Trips: **PRESERVE; no destructive migration or retrofit**
- Required architecture action: previous locked same-company behavior has been formally superseded for NEW Trips by the Chat50 governance decision.

## Authoritative Product Rule

For every **NEW Trip**, the Sending Company and Receiving Company MUST be different companies.

Canonical invariant:

```text
Trip.company_id != Trip.receiving_company_id
```

A Company MUST NOT be allowed to select itself as the Receiving Company when it is the Sending Company for a new Trip.

This rule MUST be enforced in both layers:

1. **Client/UI:** the authenticated sender company must not be presented as a valid receiving-company option.
2. **Server/API:** the Trip creation endpoint must independently reject any attempted same-company request, including direct API manipulation.

Client filtering alone is insufficient.

## Explicitly Out of Scope

Do NOT:

- delete existing Trips where `company_id == receiving_company_id`;
- reassign existing receiver companies;
- rewrite historical receiver requests;
- run a migration/backfill for legacy same-company records;
- alter unrelated Driver, Reviewer, Trip lifecycle, marketplace, publication, claim, completion, or event behavior;
- reintroduce the former automatic same-company receiver-request `ACCEPTED` bypass for newly created Trips;
- modify unrelated UI or architecture as a side quest;
- push commits to GitHub unless Ayush separately gives explicit push authorization.

Existing same-company Trips remain valid legacy data for this change unless a separate governance decision is made later.

## Verified Source-Repository Preflight

Source repository:

```text
https://github.com/ayush22cp008/freight_hackathon
```

Branch observed during preflight:

```text
main
```

Relevant verified source files:

```text
src/app/(authenticated)/company/trips/create/CreateTripClient.tsx
src/app/api/trips/create/route.ts
src/app/api/companies/lookup/route.ts
```

Verified current behavior:

- `CreateTripClient.tsx` loads receiving companies from `/api/companies/lookup` and currently renders the returned companies without excluding the authenticated sender company.
- `POST /api/trips/create` derives the creator company from the authenticated user but currently validates only that the requested receiving company exists; it does not reject `creatorCompany.id === receiving_company_id`.
- The existing API explicitly calculates `isSameCompany` and creates the receiver request as `ACCEPTED` for same-company Trips.

These findings are the basis for the required fix.

## Required Implementation Work

### 1. Confirm execution boundary before changing source

Before editing anything, confirm and report:

- actual project root;
- current working directory;
- target Git repository;
- current branch;
- clean/dirty working-tree state;
- exact target files to be changed.

Stop immediately if the repository or project root does not match the expected source repository.

### 2. Receiving-company lookup / UI behavior

Inspect the existing company lookup flow and choose the smallest architecture-consistent implementation that guarantees the authenticated sender company is not offered as a receiver.

Preferred outcome:

- the selector contains eligible receiving companies other than the authenticated sender company;
- the sender company is not merely disabled/hidden after selection while remaining otherwise available;
- normal company lookup behavior for other consumers is preserved unless source inspection proves a server-filtered lookup is the safer canonical boundary.

Do not expose the sender company as a valid choice through the UI.

### 3. Server-side Trip creation enforcement

Update the authoritative Trip creation path so that, after the authenticated creator company and requested receiver are known, it explicitly rejects:

```text
creatorCompany.id === receiving_company_id
```

The request MUST fail safely with a clear client-consumable error response and MUST NOT persist a new Trip with the same company in both roles.

Expected semantics:

```text
A -> B  = allowed, subject to existing validation
A -> A  = rejected
```

The sender identity MUST continue to be derived from authenticated server-side state. Do not trust a client-supplied sender company id.

### 4. Remove the obsolete NEW-Trip same-company bypass

Because same-company is no longer valid for NEW Trips, the newly-created same-company branch must no longer be reachable for valid creation requests.

Do not preserve the former behavior by silently converting a same-company receiver request into another state.

For valid new Trips:

- sender and receiver are distinct;
- existing receiver-request creation behavior for cross-company Trips remains intact;
- existing publication/claim gating remains intact.

### 5. Preserve legacy records

Do not change database rows for existing Trips or receiver requests merely because their sender and receiver are the same company.

Do not add a migration unless a separate approved legacy-data decision exists.

When inspecting downstream code, distinguish:

```text
NEW Trip creation invariant
```
from

```text
LEGACY existing Trip compatibility
```

The new invariant must not accidentally break legitimate historical records that already contain same-company relationships.

### 6. Inspect alternate creation paths

Search the source repository for other Trip creation paths that can create a `trips` row or otherwise bypass `/api/trips/create`.

If another authoritative creation path exists, report it before changing it and determine whether it is in scope for the same invariant.

Do not modify speculative or unrelated code simply because it mentions `receiving_company_id`.

## Required Test/Verification Coverage

Implement or add the smallest appropriate automated coverage supported by the existing repository/testing structure.

At minimum, verify:

1. **Cross-company creation allowed**
   - authenticated company A creates a Trip for company B;
   - existing expected creation/receiver-request behavior remains valid.

2. **Direct API same-company rejection**
   - authenticated company A submits `receiving_company_id = A`;
   - API rejects the request;
   - no new Trip is persisted.

3. **UI receiver selector exclusion**
   - authenticated company A does not appear as a selectable receiver.

4. **Legacy compatibility**
   - existing same-company Trip records are not deleted, reassigned, or rewritten by this change.

5. **No regression to unrelated Trip lifecycle behavior**
   - do not change publication, claim, receiver acceptance/rejection, driver flow, reviewer flow, or completion behavior except where directly required to enforce the new creation invariant.

## Error Handling Requirement

The same-company rejection should be deterministic and safe. Do not leak unnecessary internal details.

Use the repository's existing API error-response conventions where applicable. Keep status code and message conventions consistent with adjacent validation failures unless there is a strong repository-specific reason to do otherwise.

## Scope Discipline

Only modify files directly required for:

- excluding the authenticated sender from valid receiving-company selection;
- enforcing sender != receiver on NEW Trip creation;
- removing/reaching-inaccessible the obsolete new-Trip same-company bypass;
- adding the necessary focused automated tests/documentation if the repository structure supports them.

Do not redesign the Company portal.

## Validation Before Reporting Completion

After implementation:

1. Inspect the final diff.
2. Confirm only intended files changed.
3. Confirm there are no unexpected generated files or unrelated edits.
4. Run the repository's appropriate lint/typecheck/test/build commands for the affected scope.
5. Explicitly verify the API path rejects same-company creation before any new Trip is persisted.
6. Confirm legacy same-company records were not modified by the implementation.
7. Report exact commands run and their outcomes.
8. Report any test limitations or environment-dependent checks honestly.

## Required Antigravity Implementation Report

Create the implementation report in the Records repository after the source work is complete, following the existing project reporting convention.

The report must include:

- implementation status;
- exact changed source files;
- what changed in each file;
- API validation behavior;
- UI receiver-list behavior;
- legacy-data preservation confirmation;
- tests/checks executed and results;
- build/lint/typecheck results;
- unexpected changes, if any;
- remaining limitations;
- explicit statement that no GitHub push was performed unless Ayush separately authorized it.

## Final Gate

This handoff authorizes implementation of the approved Chat50 product rule, but it does **not** authorize a GitHub push.

Do not push or merge until Ayush explicitly authorizes that action after implementation review and manual verification.
