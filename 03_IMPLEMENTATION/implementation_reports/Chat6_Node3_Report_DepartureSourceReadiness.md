# Chat6 Node3 Investigation: Departure Source Readiness

## 1. Investigation Objective
Perform a read-only source investigation of the current Freight application to determine the implementation readiness of **Event 3 — Departure**.

## 2. Source Snapshot / Commit or Working-State Identifier
- **Environment**: Local working tree for Freight Hackathon
- **Current State**: Arrival and Check-in events are fully implemented and verified. The Dashboard correctly reflects the sequence and points to the next step.

## 3. Files Inspected
- `src/app/(authenticated)/page.tsx` (Dashboard logic)
- `src/db/migrations/002_create_events_table.sql` (Database schema and constraints)
- `src/app/(authenticated)/events/` and `src/app/api/events/` directories

## 4. Reusable Infrastructure Findings
- **GPS, Server Timestamp, Photo Capture**: The utilities in `src/lib/capture/` (`getGpsLocation`, `getServerTime`, `uploadPhoto`) are fully reusable for Departure.
- **Event Insertion Pattern**: The existing pattern used in `api/events/arrival/route.ts` and `api/events/checkin/route.ts` can be safely mirrored for Departure.
- **Duplicate Prevention**: The existing database constraint `UNIQUE (trip_id, event_type)` will automatically handle and prevent duplicate Departure events.

## 5. Dashboard / Workflow Findings
- **State Logic**: The Dashboard correctly tracks `hasDeparture`.
- **Pre-Departure State**: When `hasArrival` and `hasCheckin` are true but `hasDeparture` is false, the Dashboard state is **Check-in Complete** and the CTA is **Start Departure** linking to `/events/departure`.
- **Post-Departure State**: Once `hasDeparture` becomes true, the Dashboard state changes to **Trip Complete** and the CTA changes to **View Timeline** linking to `/timeline`. The Dashboard's state machinery is fully prepared for this transition.

## 6. Existing Departure Findings
- **VERIFIED**: There is currently no source code for Departure. The paths `src/app/(authenticated)/events/departure` and `src/app/api/events/departure` do not exist.

## 7. Database / Schema Findings
- **VERIFIED**: The `events` table schema explicitly allows `event_type IN ('arrival', 'checkin', 'departure')`.
- **VERIFIED**: The uniqueness constraint `UNIQUE (trip_id, event_type)` covers Departure.
- **VERIFIED**: The `photo_url` column is available for the required Departure photo.
- **VERIFIED**: No schema changes are required.

## 8. VERIFIED / INFERRED / UNKNOWN Evidence Table

| Check / Requirement | Status | Evidence Source |
|---------------------|--------|-----------------|
| Shared capture utils reusable | **VERIFIED** | `src/lib/capture/` contents |
| Dashboard tracks `hasDeparture` | **VERIFIED** | `src/app/(authenticated)/page.tsx` |
| Dashboard links to `/events/departure` | **VERIFIED** | `src/app/(authenticated)/page.tsx` |
| Post-departure state is Trip Complete | **VERIFIED** | `src/app/(authenticated)/page.tsx` |
| `departure` enum exists in DB | **VERIFIED** | `002_create_events_table.sql` |
| DB prevents duplicate departure | **VERIFIED** | `UNIQUE (trip_id, event_type)` constraint |
| Existing Departure code | **VERIFIED** | Does not exist |
| API validates order server-side | **INFERRED** | Missing from previous routes; likely only enforced by Dashboard UI |

## 9. Blockers / Risks
- **No strict server-side order validation**: The API endpoints do not currently enforce that Check-in must precede Departure. This is controlled only via Dashboard UI flow. (Out of scope to fix per instructions).

## 10. Departure Implementation Readiness Decision
**Ready.** Departure can be safely implemented using the exact same pattern as Arrival (requiring a photo), reusing the existing infrastructure. The Dashboard is already configured to handle the state transitions.

## 11. Smallest Safe Implementation Surface
Files to **ADD**:
- `src/app/(authenticated)/events/departure/page.tsx`
- `src/app/(authenticated)/events/departure/DepartureClient.tsx`
- `src/app/api/events/departure/route.ts`

Files to **CHANGE**:
- None. (Dashboard already routes correctly).

## 12. Recommended Next Step for ChatGPT/Ayush
Review the implementation surface and proceed to implement Departure. The `Arrival` code should be used as the primary template since Departure also requires a photo.
