# Chat26 — Node 5 — Receiver Check-In Driver Profile Not Found Bug Fix

## Objective
Fix the `RECEIVER_CHECKED_IN` submission failure that shows `Driver profile not found` when a receiving COMPANY user attaches photo evidence.

## Evidence / Root Cause
Read and follow:
`05_DEBUGGING/investigations/Chat26_Node5_Receiver_Checked_In_Driver_Profile_Not_Found_Investigation.md`

The shared `POST /api/upload-photo` endpoint currently assumes the authenticated user must have a driver profile. A valid receiving COMPANY therefore fails at the photo-upload step even though the driver profile exists.

## Required Fix
Modify only the evidence-upload authorization needed in:
`src/app/api/upload-photo/route.ts`

The endpoint must support both existing actor types:
- COMPANY: authenticate the Freight identity, resolve the company profile, and authorize the upload against the trip's `receiving_company_id`.
- DRIVER: preserve the existing driver authorization behavior unchanged.

Keep the existing storage bucket/path conventions compatible with the current implementation. Do not create or duplicate driver profiles.

## Security Constraints
- Do not weaken authentication or authorization.
- Do not allow an arbitrary company to upload evidence for another company's trip.
- Preserve the existing server-side relationship check.
- Do not move authorization to the client.
- Do not change Node 5 event vocabulary, milestone sequencing, or database schema.
- Do not modify later Node 5 milestones.

## Verification
1. Run `npx tsc --noEmit`.
2. Verify the existing driver photo-upload flow remains intact.
3. Verify a verified receiving COMPANY can upload optional photo evidence for its own `RECEIVER_CHECKED_IN` trip.
4. Verify an unrelated company cannot upload evidence for that trip.
5. Report exact files changed, authorization logic, test result, and any remaining manual verification requirement.

## Scope
This is a focused bug fix, not a redesign. Do not push to GitHub. Record the implementation result in:
`03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Receiver_Checked_In_Driver_Profile_Not_Found_Bugfix_Implementation_Report.md`
