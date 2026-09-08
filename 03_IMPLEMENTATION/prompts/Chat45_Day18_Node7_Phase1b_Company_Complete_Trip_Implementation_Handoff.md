# Chat45 — Day 18 — Node 7 Phase 1b — Company Complete Trip Implementation Handoff

## Status
Implementation-ready handoff.

## Objective
Implement the locked Company completion-discovery UX for Phase 1b without changing backend behavior, APIs, database/schema, RLS/security, authentication/authorization, trip lifecycle semantics, claiming behavior, evidence requirements, persistent review state, or AI behavior.

## Source-of-truth decisions
- Both Driver and Receiving Company confirmations are required for lifecycle completion.
- Company completion discovery is Company-specific and must not literally copy the Driver completion banner.
- Company is multi-trip/list based, so completion discovery belongs on Company active/created-trip surfaces as an appropriate recent-completion discovery treatment.
- The immediate completion flow and later-return active-trip flow are two UI entry points to the same underlying trip-specific temporary completion acknowledgement purpose.
- Once the Company uses either entry point to open the exact completed Trip Detail, that temporary completion discovery is consumed for that trip and must not continue appearing in the other Company surface.
- Acknowledgement is frontend-only and must not mutate lifecycle state in the database.

## Required preflight
Before editing:
1. Confirm project root/current working directory.
2. Confirm target source repository and current Git repository.
3. Confirm current branch.
4. Inspect current source state for the Company routes/components named below; do not assume the inspected historical state is unchanged.
5. Confirm the requested change is frontend-only and identify any unexpected dependency. If backend/API/schema/RLS/business-rule changes appear necessary, STOP and report rather than expanding scope.

## Intended Company behavior
### Case 1 — Company confirms first, Driver completes while Company remains
1. Company completes the Receiving Company confirmation interaction.
2. If Driver has not yet confirmed, Company remains in the waiting state.
3. If Driver confirms while Company remains in the flow, the Company can reach the completed state.
4. The completed state provides a clear `View Completed Trip` action.
5. That action opens the exact Company Trip Detail for the same trip ID.
6. Viewing the completed Trip Detail consumes the temporary completion discovery for that trip.
7. The completed trip is no longer presented as an active trip and is available through Company History.

### Case 2 — Company leaves and returns later
1. Company confirms and leaves while the trip is waiting for Driver confirmation.
2. Driver later confirms and the trip becomes completed.
3. Company returns to the relevant active/created-trip surface.
4. The completed trip is no longer an active trip, but a Company-specific recent-completion discovery treatment is shown for the relevant recently completed trip.
5. The treatment clearly communicates that the recent trip is finished and provides `View Completed Trip`.
6. The CTA opens the exact `/company/trips/[id]` Trip Detail.
7. After the completed Trip Detail is viewed, the temporary discovery is consumed for that trip.
8. The completed trip remains accessible in History.

### Recovery case — Company returns while Driver is still pending
If the trip is still unresolved, it remains active. The relevant trip card should provide `View Completion Status`, which opens the current completion state. Do not show a completed-trip discovery treatment unless the trip is actually completed.

## Trip identity and acknowledgement
- Preserve the exact trip ID through all completion CTAs and routing.
- Use the existing Company Trip Detail route: `/company/trips/[id]`.
- Use a per-trip frontend acknowledgement mechanism consistent with the verified pattern: `acked_completed_trip_${tripId}` in browser `localStorage`.
- The acknowledgement only dismisses temporary discovery. It must never change `trip.status`, confirmation timestamps, or any server-side lifecycle state.
- Multiple completed trips must be handled independently; acknowledgement of one trip must not hide another unacknowledged completed trip.
- The Company UI may present a recent-completions list rather than a single global banner. Do not invent unsupported backend fields or APIs.

## Verified source surfaces to work with
Re-verify current source before modification:
- `src/app/(authenticated)/company/created/page.tsx`
- `src/app/(authenticated)/company/completion/page.tsx`
- `src/app/(authenticated)/company/trips/[id]/page.tsx`
- `src/app/(authenticated)/company/history/page.tsx`

Relevant verified behavior from readiness inspection:
- Company Created Trips scopes to the authenticated company and filters out `status === 'completed'`.
- Company Completion is a Receiving Company confirmation interaction and is restricted to active/claimed/in_progress states; it is not a general completed-trip viewer.
- Company Trip Detail accepts the exact trip ID and allows access when the company is either sender or receiving company.
- Company History queries completed trips in which the company participates and links to the exact Company Trip Detail.
- Existing Company navigation back to Dashboard uses a fresh Server Component render; no custom polling/realtime was verified.
- The frontend can support trip-specific localStorage acknowledgement with a client component.
- Multiple completed trips can be represented as a list and acknowledged independently.

## Implementation scope
### In scope
- Company frontend UI/UX needed for recent completed-trip discovery.
- Company active/created-trip presentation needed to distinguish unresolved active trips from recently completed trips.
- `View Completion Status` for unresolved waiting trips where required by the locked interaction model.
- `View Completed Trip` routing to exact Company Trip Detail.
- Client-side, per-trip acknowledgement of temporary completion discovery.
- Responsive presentation consistent with the locked Company and shared design system.
- Reuse existing data already exposed by current Company queries/routes.
- UI-only corrections required to make this locked interaction coherent.

### Out of scope / protected boundaries
Do NOT change:
- API contracts or server behavior.
- Database schema, migrations, or stored data model.
- RLS/security policies.
- Authentication or role/authorization rules.
- Trip lifecycle/state semantics.
- Claiming/marketplace behavior.
- Evidence requirements/types/integrity.
- Persistent review state.
- Backend business rules.
- AI behavior.
- Reviewer authority expansion.
- C-05 or R-03 protected areas.
- R-05 dependency beyond the already verified narrow data-source readiness boundary.
- Any new backend endpoint solely to support this UI.

## UX requirements
- Keep Company as one unified portal.
- Do not create a second progress dashboard for Incoming Deliveries.
- Preserve the locked Company Trip Detail hierarchy:
  1. Current Status
  2. Visual Delivery Progress
  3. Next Required Action
  4. Driver / Claim Information
  5. Trip Details
  6. Delivery Evidence
  7. Timeline / History
- Completed trips are review-only.
- History remains the durable destination for completed-trip access.
- Temporary completion discovery is only a convenience/re-entry mechanism.
- Avoid duplicate or contradictory completion messaging.
- Do not expose a completed trip as active.
- Do not rely on completion status alone to decide whether a temporary discovery has been acknowledged; acknowledgement is per trip and client-side.

## Acceptance criteria
1. Company can complete its confirmation and see the correct waiting/completed state according to existing lifecycle state.
2. If the Driver is still pending, the Company can return to the relevant active/created trip and use `View Completion Status`.
3. If the trip is completed, the Company active/created-trip surface can expose a clear recent-completion discovery for that exact trip even though completed trips are filtered from active presentation.
4. `View Completed Trip` always opens `/company/trips/{exactTripId}` for the correct trip.
5. Viewing the completed Trip Detail consumes the temporary discovery for that trip.
6. After acknowledgement, the same completed trip does not continue to appear as an unacknowledged recent completion on another Company surface.
7. Acknowledgement does not change database lifecycle state.
8. Multiple completed trips can be discovered and acknowledged independently.
9. Completed trips remain available in Company History.
10. Sender and Receiver Company access rules already enforced by the current Trip Detail route remain unchanged.
11. No backend/API/DB/RLS/business-rule changes are introduced.
12. Responsive behavior follows the locked Company design system and does not introduce normal workflow horizontal scrolling.
13. No unrelated portal behavior is changed.

## Testing requirements for Antigravity
After implementation:
- Run the relevant lint/typecheck/build/test commands available in the project.
- Verify no backend/API/schema/RLS files were changed.
- Verify exact trip IDs are preserved in every completion CTA.
- Verify localStorage acknowledgement is per trip.
- Verify unresolved waiting trips remain active.
- Verify completed trips disappear from active trip presentation while remaining discoverable through the temporary recent-completion treatment and durable History.
- Verify multiple completed trips independently.
- Verify both Company sender and receiving-company access to completed Trip Detail where already supported.
- Record all changed files and test/build results in the implementation report.

## Manual verification required from Ayush
Antigravity must not claim final acceptance. After implementation and automated checks, stop at the handoff to Ayush for manual browser/UI verification.

Ayush should manually verify at minimum:
- Company confirms first → waiting state → Driver confirms → completed discovery → `View Completed Trip` → exact Trip Detail.
- Company confirms → leaves → Driver confirms later → Company returns → recent completion discovery → exact Trip Detail.
- Company returns before Driver confirmation → `View Completion Status` → still waiting.
- Acknowledgement removes only the relevant temporary discovery.
- A second completed trip remains independently discoverable.
- Completed trip is available in History.
- Sender/Receiver Company Trip Detail access remains correct.
- Responsive desktop/mobile presentation is coherent.

## Postflight requirements
Before reporting completion:
1. List all changed source files.
2. Confirm there were no unexpected changes.
3. Report build/lint/typecheck/test results.
4. Explicitly state whether any protected boundary was touched. If yes, STOP and report.
5. Write the implementation report in the Records repo under `03_IMPLEMENTATION/implementation_reports/` using the current Chat45 naming convention.
6. Do not push unless Ayush explicitly authorizes the push.

## Final implementation boundary
This handoff authorizes implementation of the locked Company completion-discovery UX only. It does not authorize architecture changes or backend changes. Any unexpected requirement must be surfaced as UNKNOWN and returned to Ayush/active reasoning brain for a decision before proceeding.