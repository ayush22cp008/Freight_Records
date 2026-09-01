# Chat26 — Node 5 — Receiver Checked In Driver Profile Not Found Investigation

## 1. Issue Description
When a verified company attempts to submit the `RECEIVER_CHECKED_IN` event with photo evidence, the request fails with the error message: `Driver profile not found`.

## 2. Root Cause Analysis
The frontend for `RECEIVER_CHECKED_IN` correctly implements the company authorization logic. However, when a photo is attached, it calls the shared `uploadPhoto` utility which posts to `POST /api/upload-photo`.

In `src/app/api/upload-photo/route.ts`, the endpoint currently hardcodes driver-specific authorization logic:
```typescript
// 1. Verify driver identity
const { data: driver } = await supabaseServer
  .from('drivers')
  .select('id')
  .eq('auth_id', user.id)
  .single();

if (!driver) {
  return NextResponse.json({ error: 'Driver profile not found' }, { status: 403 });
}
```
Because the `upload-photo` endpoint assumes only drivers upload evidence (which was true for all prior milestones like Load, Departure, Transit, Arrival), it fails to recognize the valid `COMPANY` identity of the receiving company.

## 3. Impact
The `RECEIVER_CHECKED_IN` milestone API route works correctly for text/GPS data, but fails entirely if the user attempts to include a photo, as the photo upload blocks the entire submission. 

## 4. Proposed Solution
Update `src/app/api/upload-photo/route.ts` to implement dual-role authorization:
1. Identify the user using `getFreightIdentity()`.
2. If `identity.trusted_role === 'COMPANY'`:
   - Fetch the company profile.
   - Verify that `trip.receiving_company_id === company.id`.
   - Store the photo in `event-photos` bucket under a path like `${company.id}/${trip_id}/${Date.now()}-...`
3. If `identity.trusted_role === 'DRIVER'` (or if a driver profile exists):
   - Fall back to the existing driver-based authorization.
   - Store the photo under `${driver.id}/${trip_id}/${Date.now()}-...`

This cleanly isolates the evidence authorization to match the existing actor relationship logic established in the API routes.

## 5. Next Steps
Awaiting approval to implement the dual-role authorization in the `upload-photo` endpoint.
