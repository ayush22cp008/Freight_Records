# Chat25 — Node 5 Subnode 5.S2 — Secure Check-in Photo Storage Implementation

## Approval
Ayush explicitly approved Option B — Security Rewrite for the Check-in photo storage problem.

Approved decision record:
`02_ARCHITECTURE/Chat25_Node5_S2_Secure_Storage_Design_Decision.md`

Design investigation:
`03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Checkin_Photo_Storage_Design_Report.md`

## Role
You are Antigravity, the implementation/execution agent.
Implement the approved secure storage design. Do not broaden scope beyond the evidence and decisions recorded above.

## Objective
Fix the Check-in photo upload failure by securely configuring the `event-photos` storage capability and securing the upload endpoint so that an unauthenticated caller cannot use the service-role proxy to upload arbitrary files.

## Required security behavior
- Require an authenticated user at the upload endpoint.
- Verify that the authenticated user is the driver authorized for the relevant trip before accepting trip evidence.
- Bind the stored object to the authorized driver/trip using a deterministic, non-user-controlled path or equivalent server-enforced binding.
- Do not expose or leak the service-role key to the client.
- Configure storage access according to the approved implementation-ready design and least privilege.
- Preserve Timeline's ability to display legitimate evidence.
- Handle upload/event-write failure without silently leaving uncontrolled/orphaned evidence where practical.

## Important implementation gate
Before modifying source or database files, inspect the current application and Records design again. If the existing records do not provide enough information to determine the exact trip/event authorization rule or read-access model safely, STOP and create a design-gap report instead of guessing.

## Compatibility
- Preserve existing Arrival, Check-in, and Departure evidence behavior.
- Preserve the already-fixed optional Check-in photo behavior.
- Do not change Node 5 S1 event vocabulary.
- Do not implement unrelated Node 5 lifecycle work.
- Do not alter unrelated RLS/security architecture.

## Storage migration
If the approved design supports it, create the required migration for the `event-photos` bucket and only the necessary Storage policies/configuration. Do not simply create a public bucket while retaining an unauthenticated upload endpoint.

## Validation
Run appropriate build/typecheck/lint/test checks. Where environment access permits, validate:
1. unauthenticated upload is rejected;
2. authenticated unauthorized driver/trip upload is rejected;
3. authorized driver upload succeeds;
4. Check-in event can retain the resulting evidence URL;
5. Timeline can display legitimate evidence;
6. existing no-photo Check-in still succeeds.

Do not claim Ayush manual verification unless Ayush performs it.

## Source-control
Record exact source commit before and after. Do not push source changes without explicit authorization.

## Required Records report
Create:
`03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Secure_Storage_Implementation_Report.md`

Include:
1. Baseline
2. Design references
3. Files changed
4. Storage/bucket changes
5. Policy/RLS changes
6. Upload authorization changes
7. Validation commands/results
8. Security verification
9. Manual verification status
10. Scope/non-changes
11. Commit status
12. VERIFIED / INFERRED / UNKNOWN
13. Remaining action

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

Security checks:
<summary>

Manual Ayush verification:
NOT PERFORMED

Push:
NOT PERFORMED
```

Do not paste source code or the full report into chat.
