# Chat42 — Day 16 — Node 7 — Phase 1b — Implementation Preparation Master Scope

## 1. Preparation Status

- Preparation: **READY FOR REVIEW**
- Implementation: **NOT AUTHORIZED**
- Authorization owner: Ayush
- Architecture/reasoning: ChatGPT
- Execution: Antigravity
- Coordination: `Freight_Records`

This is an implementation-preparation record only. It defines the execution boundary and sequencing; it does not authorize code changes.

## 2. Governing Records

Use these as the implementation source of truth, in this order:

1. `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`
2. `02_ARCHITECTURE/locked_blueprints/Driver_Locked_Blueprint.md`
3. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
4. `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
5. Chat41 locked shared Cross-Portal Design System decision/record.
6. Existing-system investigation reports for Driver, Company, Reviewer.
7. Chat42 disputed-11 resolution report.

If a lower-level implementation assumption conflicts with a locked blueprint or the boundary decision, stop and report the conflict. Do not silently reinterpret the architecture.

## 3. Core Product Model

The shared product experience is organized around:

**Evidence → Timeline → Verification → Decision**

The Phase 1b objective is to make the existing frontend clearly express this model for each role without changing underlying product authority or backend behavior.

## 4. Global Implementation Boundary

### Allowed

- Frontend page restructuring.
- Frontend route/surface presentation where existing data and capabilities support it.
- Shared navigation presentation and role-aware route exposure.
- Existing-component refactoring/decomposition.
- Layout, spacing, typography, cards, panels, status treatment.
- Responsive/mobile corrections.
- Accessibility and interaction-clarity improvements.
- Existing-data presentation improvements.
- Custom frontend modal replacing `window.prompt` for the existing rejection-reason interaction.
- Consistent loading/empty/error/success presentation using existing state truth.

### Protected — Do Not Change

- API contracts and response shapes.
- Database schema/migrations.
- Supabase RLS policies.
- Authentication/authorization policy logic.
- Role assignment logic.
- Business rules.
- Trip lifecycle semantics.
- Claiming/marketplace behavior.
- Evidence requirements or new evidence types.
- New persistent workflow states.
- AI verification/scoring/automation.
- Reviewer authority outside identity/evidence verification.
- Security/evidence-integrity boundaries.
- Backend implementation.

Protected defects:

- C-05 — Receiver Completion API response-shape defect.
- R-03 — Reviewer authorization/RLS issue.

## 5. Shared Foundation Preparation

### Primary shared surfaces

- `src/app/(authenticated)/Navbar.tsx`
- `src/app/(authenticated)/layout.tsx`
- Any shared navigation/status/layout components discovered during preflight.

### Required behavior

- Navigation must be role-aware.
- Driver must not receive Company/Reviewer destinations.
- Company must not receive Driver/Reviewer destinations.
- Reviewer must not enter Driver/Company operational routes through generic navigation.
- Existing authentication truth remains authoritative.
- Avoid introducing a second role/authorization system in the frontend.
- Preserve valid existing routes until their replacement surface is verified.
- Remove the known navigation trap without changing backend authorization.

### Shared regression requirements

After any shared navigation change, test all three roles for:

- login entry
- dashboard/home destination
- portal navigation
- timeline/history destination
- profile/account destination
- logout
- unauthorized cross-portal route access

## 6. Driver Preparation Map

### Target structure

1. Dashboard
2. Available Trips
3. Trip Detail
4. My Active Trip
5. Completed Trips / History
6. Profile

### Existing capability to preserve

- Existing trip retrieval.
- Existing trip claiming.
- Existing active-trip progression.
- Existing lifecycle/event recording.
- Existing completed-trip history/timeline.
- Existing authentication and Driver identity behavior.

### Phase 1b changes

- Refactor monolithic Dashboard presentation into clearer surfaces/components.
- Make Available Trips accessible even when an active trip exists, consistent with the locked blueprint.
- Provide dedicated Trip Detail presentation using existing trip data.
- Provide dedicated My Active Trip presentation using existing lifecycle/state data.
- Preserve one-next-step operational presentation; do not change lifecycle semantics.
- Present evidence/status coherently using only verified existing evidence data.
- Improve Driver Profile presentation using existing profile/identity data.
- Improve completed/history presentation.
- Normalize loading, empty, error, and relevant state presentations.
- Apply shared responsive/accessibility rules.

### Driver non-goals

- No new trip lifecycle.
- No new claim rules.
- No new evidence requirement.
- No backend endpoint changes.
- No AI behavior.

## 7. Company Preparation Map

### Target structure

1. Dashboard
2. My Created Trips
3. Incoming Deliveries
4. History / Timeline
5. Profile / Account

### Existing capability to preserve

- Company identity/authentication.
- Trip creation/publishing.
- Incoming delivery visibility.
- Receiver check-in.
- Receiver completion.
- Existing public share behavior.

### Phase 1b changes

- Fix Sender visibility presentation using existing trip data/capabilities.
- Restructure Dashboard around Company operational context.
- Separate My Created Trips from Incoming Deliveries presentation.
- Make History/Timeline a clearly discoverable Company surface using existing available data.
- Provide Company-specific Profile/Account presentation using existing Company data.
- Add accessible mobile navigation through the shared navigation system.
- Correct Create Trip responsive layout.
- Promote existing Company workflow destinations into clear top-level navigation where supported.
- Normalize responsive/accessibility/state presentation.

### Company protected boundary

Do not change `src/app/api/completion/route.ts` or its response contract as part of Phase 1b.

C-05 remains outside implementation even if the existing frontend appears inconsistent with that response.

## 8. Reviewer Preparation Map

### Target workflow

1. Verification Queue
2. Applicant Verification
3. Evidence Examination
4. Decision Result
5. Verification History
6. Read-only Verification Record
7. Submitted Evidence Viewer

### Reviewer responsibility boundary

Identity and evidence verification only.

### Allowed Phase 1b changes

- Replace queue-only presentation with the locked verification workflow using existing Reviewer data/capabilities.
- Fix Reviewer navigation trap through role-aware frontend navigation.
- Present applicant, role, and evidence context clearly.
- Replace native `window.prompt` rejection UX with a custom frontend modal.
- Improve decision result and record presentation where existing data supports it.
- Use existing evidence viewer/storage mechanisms.

### R-05 gate

Before implementing Verification History, perform a narrow readiness check:

- Identify the exact existing data source for completed Reviewer decisions.
- Identify the exact existing read/query mechanism available to the Reviewer frontend.
- Confirm whether history can be presented without a new backend/API/database capability.

If existing data/API support is not proven, stop that portion and report `NOT IMPLEMENTATION-READY`. Do not invent a backend history endpoint or persistent state.

### Reviewer protected boundary

Do not modify Reviewer authorization/RLS policies as part of Phase 1b.

R-03 is protected/out of scope.

## 9. Cross-Portal Dependency Matrix

| Dependency | Driver | Company | Reviewer | Control |
|---|---|---|---|---|
| Shared Navbar | Yes | Yes | Yes | One role-aware navigation model |
| Authenticated layout | Yes | Yes | Yes | Preserve existing auth/session behavior |
| Profile route | Yes | Yes | No/role-specific | Avoid generic cross-role destination confusion |
| Timeline/history navigation | Yes | Yes | Yes | Route must resolve to role-valid surface |
| Status treatment | Yes | Yes | Yes | Shared design-system semantics |
| Responsive behavior | Yes | Yes | Yes | Mobile/tablet/desktop verification |
| Existing APIs | Yes | Yes | Yes | Preserve contracts |
| Auth/RLS | Protected | Protected | Protected | No changes |

## 10. Exact Execution Sequence

### Stage 0 — Preflight

Antigravity must first:

- Verify current branch/worktree state.
- Verify current source commit.
- Inspect relevant current files before edits.
- Confirm the actual route/component structure against this plan.
- Confirm no implementation has already started unexpectedly.
- Report any mismatch before changing code.

### Stage 1 — Shared Foundation

- Prepare role-aware navigation/layout changes.
- Do not modify backend.
- Build/test.
- Capture evidence.

### Stage 2 — Driver

- Implement Driver-only Phase 1b changes.
- Build/test.
- Capture screenshots/logs/verification evidence.
- Stop for Ayush manual verification.

### Stage 3 — Company

- Implement Company-only Phase 1b changes.
- Preserve protected C-05.
- Build/test.
- Capture evidence.
- Stop for Ayush manual verification.

### Stage 4 — Reviewer Readiness Gate

- Resolve R-05 data-source readiness narrowly.
- If supported, continue to Reviewer frontend implementation.
- If unsupported, leave History unimplemented and record the boundary.

### Stage 5 — Reviewer

- Implement locked Reviewer workflow frontend.
- Preserve protected R-03.
- Build/test.
- Capture evidence.
- Stop for Ayush manual verification.

### Stage 6 — Cross-Portal E2E

Verify:

- Driver navigation and lifecycle.
- Company sender/receiver flows.
- Reviewer verification flow.
- Shared navigation.
- Role separation.
- Mobile/responsive behavior.
- Existing API compatibility.
- No protected backend/security changes.

### Stage 7 — Final Demo Readiness

Only after all three manual verification gates pass:

- Resolve remaining in-boundary defects.
- Re-run build/test.
- Produce final evidence.
- Prepare demo readiness status.

## 11. Evidence Requirements

Every implementation stage must produce evidence for:

- Files changed.
- Routes/components changed.
- Build/test result.
- Important UI states.
- Role-specific navigation.
- Responsive behavior where relevant.
- Confirmation that protected files/areas were not modified.

No claim of completion without evidence.

## 12. Stop Conditions

Antigravity must stop and report instead of guessing when:

- A blueprint requirement needs backend support not already verified.
- A protected boundary appears necessary to satisfy the UI.
- Existing API response differs from assumptions.
- A route/data source cannot be located.
- Role behavior is ambiguous.
- A shared component change could alter authorization or business behavior.
- R-05 cannot be proven frontend-ready.
- Any requested change would introduce new product behavior.

## 13. Authorization Gate

**Implementation remains NOT AUTHORIZED.**

This preparation record is complete enough to create the implementation master prompt and then request Ayush's explicit authorization.

The next operational artifact should be the master implementation prompt under `03_IMPLEMENTATION/prompts/`. That prompt must reference this preparation record and must explicitly state that execution cannot begin until Ayush authorizes it.
