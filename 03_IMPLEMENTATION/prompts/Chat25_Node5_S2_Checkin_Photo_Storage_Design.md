# Chat25 — Node 5 Subnode 5.S2 — Check-in Photo Storage Design

## Role
You are Antigravity, the implementation/execution agent. This is INVESTIGATION + DESIGN ONLY. Do not modify application source code, database schema, migrations, configuration, tests, or shared/production data.

## Why this Subnode exists
The Chat25 Check-in investigation verified a significant unexpected legacy defect: Check-in photo upload targets the Supabase Storage bucket `event-photos`, but the repository migration history does not create that bucket or its Storage RLS policies. The no-photo bug has already been fixed and manually verified. The remaining defect is photo upload with a selected file.

Because creating a Storage bucket and its access policies is database/security infrastructure work, track this as **Node 5 Subnode 5.S2** rather than treating it as a small source-only bug.

## Source of truth
Read:
1. `00_PROJECT_CONTROL/ROADMAP.md`
2. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
3. `00_PROJECT_CONTROL/PROJECT_STATE.md`
4. `05_DEBUGGING/investigations/Chat25_Checkin_Bug_Investigation_Report.md`
5. `03_IMPLEMENTATION/implementation_reports/Chat25_Checkin_Bug_Implementation_Report.md`
6. `02_ARCHITECTURE/locked_decisions/Chat24_Node5_Architecture_Decisions.md`
7. Relevant existing storage/upload/auth/RLS records.

## Locked current facts
- Check-in without photo is already manually verified working.
- Check-in with photo still fails because the `event-photos` bucket is absent according to the investigation.
- The upload route uses `supabaseServer.storage.from('event-photos').upload(...)`.
- Do not change Node 5 event vocabulary or the approved S1 event-schema design.
- Do not reopen the completed RLS investigation unless new contradictory evidence appears.

## Required design questions
Determine from actual source and project records:
1. What exact Storage bucket configuration is required for the existing upload route (bucket name, public/private setting, file-size/type expectations if present)?
2. What exact actors should be allowed to INSERT objects?
3. What exact actors should be allowed to SELECT/read objects?
4. Should the bucket be public or private for the current product behavior?
5. How are stored photo URLs consumed by Timeline and AI Summary?
6. Does the current app expect a public URL, signed URL, or authenticated object access?
7. Can a Storage policy safely enforce that a driver can upload only to an object path associated with their authorized trip, or does the current upload route lack enough identity/path binding to enforce that? If the latter, identify it as a security/design gap rather than inventing a weak policy.
8. Are there existing Storage policies/buckets elsewhere that establish the project's intended pattern?
9. Can the required bucket/policies be expressed as a repository migration without modifying unrelated database objects?
10. What exact migration and policy design is the smallest safe fix for the current photo-upload defect?

## Security boundary
Do NOT assume that “authenticated users can insert” or “anyone can read” is acceptable. Those are examples only and are NOT approved policy decisions.

The design must explicitly account for the project's authorization model and evidence/privacy requirements. If the current upload endpoint uses a server-side privileged Supabase client that bypasses Storage RLS, state that clearly and distinguish bucket existence from policy enforcement.

Do not weaken access controls merely to make the upload work.

## Required output
Create exactly one Records design report:

`05_DEBUGGING/investigations/Chat25_Node5_S2_Checkin_Photo_Storage_Design_Report.md`

The report must contain:
1. Subnode Status
2. Parent Node / Subnode Rationale
3. Records Baseline
4. Source Baseline
5. Existing Upload Flow
6. Existing Storage Configuration Evidence
7. Bucket Design
8. Access-Control / RLS Design
9. URL/Read-Access Design
10. Migration Design
11. Security Assessment
12. Node 5 / S1 Dependency Assessment
13. Proposed Implementation Scope
14. Decision Required from ChatGPT/Ayush
15. VERIFIED / INFERRED / UNKNOWN Summary
16. Explicit Non-Changes

## Decision gate
Do not implement the migration or change source code in this task.

If a safe design can be determined, clearly state the exact proposed bucket/policy/migration behavior so ChatGPT can review and approve it before implementation.

If a safe policy cannot be determined from the current architecture, stop and identify exactly what decision is missing.

## Final response to ChatGPT
Return only:

```text
NODE 5 SUBNODE 5.S2 STORAGE DESIGN COMPLETE

Report:
05_DEBUGGING/investigations/Chat25_Node5_S2_Checkin_Photo_Storage_Design_Report.md

Design status:
READY FOR REVIEW / BLOCKED ON DECISION

Bucket:
<exact conclusion>

Access policy:
<exact conclusion>

Migration:
<proposed / blocked>

Implementation performed:
NO

Source changes:
NONE

Database changes:
NONE

Tests added:
NONE
```

Do not paste the report contents or source code into chat.
