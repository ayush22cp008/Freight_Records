# Chat25 — Node 5 Subnode 5.S2 — Secure Storage Implementation Gate

## Role
You are Antigravity, the implementation/execution agent.

## Approval
Ayush approved Option B — Security Rewrite for Check-in photo storage.

Approved decision:
`02_ARCHITECTURE/Chat25_Node5_S2_Secure_Storage_Design_Decision.md`

Design evidence:
`03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Checkin_Photo_Storage_Design_Report.md`

## Important execution gate
Before making any source/database/storage changes, inspect the approved design and current source again.

The implementation may proceed only if the exact secure authorization and storage access behavior is sufficiently specified by the existing Records. Do not invent security policy behavior.

## Required implementation scope
Implement Option B only:
1. Authenticate the upload caller.
2. Verify the caller is an authorized driver for the relevant trip/evidence action.
3. Bind the stored object to the authorized trip/driver using a server-enforced path or equivalent.
4. Create/configure the required `event-photos` storage bucket.
5. Apply least-privilege storage access needed for legitimate evidence display.
6. Preserve the already-fixed optional-photo Check-in behavior.
7. Preserve existing Arrival and Departure photo behavior.
8. Do not modify Node 5 S1 event vocabulary/schema as part of this S2 work.

## Security boundary
The existing upload route uses a Supabase service-role key. Do not leave an unauthenticated public upload route in place after implementation.

Do not make the bucket public merely to avoid designing read access. Preserve the actual Timeline requirement using the least-privilege read model supported by the current application/design.

If the current design does not contain enough evidence to safely determine public/private access or exact Storage RLS policy conditions, STOP and create a design-gap report in Records. Do not guess.

## Validation requirements
Where environment access permits, verify:
- unauthenticated upload denied;
- authenticated but unauthorized driver/trip upload denied;
- authorized driver upload succeeds;
- evidence URL is persisted/usable by the Timeline;
- no-photo Check-in still succeeds;
- Arrival and Departure photo paths are not unintentionally broken.

Run project build/typecheck/lint/test validation appropriate to changed files.

## Source control
Record exact source baseline and resulting local commit. Do not push source changes without explicit authorization.

## Required report
Create:
`03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Secure_Storage_Implementation_Report.md`

Include exact files changed, migration/policy details, authorization behavior, validation evidence, manual verification status, and VERIFIED/INFERRED/UNKNOWN classification.

## Final response
Return only:

```text
NODE5 S2 SECURE STORAGE IMPLEMENTATION COMPLETE

Report:
03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Secure_Storage_Implementation_Report.md

Source commit before:
<exact SHA>

Source commit after:
<exact SHA or NOT COMMITTED>

Validation:
PASS / FAIL / PARTIAL

Security verification:
<summary>

Manual Ayush verification:
NOT PERFORMED

Push:
NOT PERFORMED
```

Do not paste source code or full report into chat.
