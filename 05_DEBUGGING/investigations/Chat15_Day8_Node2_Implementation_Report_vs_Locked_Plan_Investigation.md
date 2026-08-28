# Chat15 — Day 8 — Node 2 Implementation Report vs Locked Plan Investigation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Day:** Day 8  
**Chat:** Chat15  
**Status:** INVESTIGATION — OPEN  

## 1. Investigation Trigger

The Chat15 implementation report claims that the Company/Driver onboarding and verification flow is complete and that automated implementation evidence is VERIFIED. However, the report also explicitly states two behaviors that conflict with the architecture decisions locked after the plan was revised:

1. Evidence collection is implemented as raw string inputs rather than actual file uploads.
2. The reviewer workflow is a headless administrative API rather than the required minimal Reviewer Queue + Review page.

Ayush also executed the reported database migration successfully. The live SQL shown during manual inspection creates `onboarding_evidence.document_value` as `text NOT NULL`, which is inconsistent with the locked private-Storage evidence model.

This investigation must reconcile the actual implementation with the locked Chat15 plan before any fix or additional migration is authorized.

## 2. Authoritative Baseline

The current authoritative implementation plan is:

`03_IMPLEMENTATION/plans/Chat15_Day8_Node2_Company_Driver_Onboarding_Identity_Implementation_Plan.md`

The implementation report under review is:

`03_IMPLEMENTATION/implementation_reports/Chat15_Day8_Node2_Report_Company_Driver_Onboarding_Identity_Implementation.md`

The earlier Chat15 integration investigation is preserved at:

`05_DEBUGGING/investigations/Chat15_Day8_Node2_Company_Driver_Onboarding_Integration_Investigation.md`

Do not overwrite the earlier investigation merely to record this new discrepancy.

## 3. Locked Requirements Relevant to This Investigation

The following are already approved and must not be silently changed:

```text
Option B identity mapping
    freight_identities.auth_id
        ↕
    drivers.auth_id / companies.auth_id

Driver → Driving Licence evidence
Company → GST evidence/details

Actual evidence document
    ↓
Private Supabase Storage
    ↓
Database metadata/reference

Technical file metadata → collected by system
Licence/GST business-field OCR/AI → NOT Node 2

Reviewer authorization
    ↓
Dedicated reviewer_authorizations authority

Reviewer experience
    ↓
Minimal Reviewer Queue + Review page

Evidence access
    ↓
Authorized server check
    ↓
Specific evidence ownership/target check
    ↓
Short-lived signed URL

Approval
    ↓
Server-authorized + transactionally consistent
    ↓
VERIFIED + trusted_role
    ↓
Minimal Driver/Company profile

Driver Code ≠ authentication credential
```

## 4. Observed Evidence

### 4.1 Migration execution

Ayush manually executed the Chat15 database migration in Supabase SQL Editor and received:

```text
Success. No rows returned
```

This proves the SQL executed successfully. It does **not** prove that the schema matches the approved architecture.

### 4.2 Live migration schema shown during execution

The migration shown by Ayush contains:

```text
CREATE TABLE IF NOT EXISTS onboarding_evidence (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    auth_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
    role_type text NOT NULL,
    document_type text NOT NULL,
    document_value text NOT NULL,
    status text DEFAULT 'PENDING',
    created_at timestamptz DEFAULT now()
);
```

The presence of `document_value text NOT NULL` is direct evidence that the actual submitted document is represented as text in the database rather than as a private Storage object/reference.

### 4.3 Implementation report

The Chat15 implementation report states that the implementation added:

- role selection during signup;
- PENDING onboarding UI;
- an onboarding submission endpoint;
- an administrative review endpoint;
- Company/Driver provisioning;
- role-aware routing.

However, its Known Limitations explicitly state:

```text
The admin approval workflow is currently a headless API.
Evidence collection is currently implemented as raw string inputs for the MVP,
rather than full file-blob uploads.
```

Therefore the report itself supplies evidence of non-conformance with the revised locked plan.

## 5. Conformance Assessment

| Requirement | Current evidence | Status |
|---|---|---|
| Explicit Driver/Company signup selection | Report claims implemented | INFERRED until source inspection |
| `requested_role` capture | Report claims implemented | INFERRED until source inspection |
| Driver → Licence | Report claims role-specific input | INFERRED |
| Company → GST | Report claims role-specific input | INFERRED |
| Actual document upload | Report says raw string | **FAILED / NOT CONFORMANT** |
| Private Storage | No evidence in report; migration uses `document_value` | **NOT DEMONSTRATED / likely FAILED** |
| Technical file metadata | Not demonstrated | UNKNOWN |
| No OCR/AI in Node 2 | No conflicting OCR claim | UNKNOWN |
| `reviewer_authorizations` | Not demonstrated | UNKNOWN |
| Reviewer Queue + Review page | Report says headless API | **FAILED / NOT CONFORMANT** |
| Signed URL evidence access | Not demonstrated | UNKNOWN / likely NOT IMPLEMENTED |
| Transactional approval | Report claims endpoint but gives no transaction evidence | UNKNOWN |
| Immutable evidence/version history | Not demonstrated | UNKNOWN |
| Rejection reason | Not demonstrated in report | UNKNOWN |
| Resubmission as new evidence version | Not demonstrated | UNKNOWN |
| Minimal Driver profile after approval | Report claims provisioned | INFERRED |
| Minimal Company profile after approval | Report claims provisioned | INFERRED |
| Role-aware routing | Report claims implemented | INFERRED |
| Driver Code not authentication | Report says logout/login works without Driver Code | INFERRED |

## 6. Investigation Questions

The investigation must answer these from actual source/database evidence, not assumptions:

### A. Database

1. What exactly did migration `005_create_onboarding_evidence.sql` create?
2. Does `onboarding_evidence` contain a plain-text document field?
3. Does a private Supabase Storage bucket/object path exist for onboarding evidence?
4. Is there an actual `reviewer_authorizations` table or equivalent dedicated authority?
5. What tables/constraints exist for minimal `drivers` and `companies` profiles?
6. Do the role records use the approved `auth_id` relationship?
7. What RLS policies exist on all newly created/modified tables?

### B. Source implementation

Inspect the exact current source for:

```text
src/app/signup/page.tsx
src/app/api/auth/signup/route.ts
src/app/(authenticated)/onboarding/page.tsx
src/app/(authenticated)/onboarding/OnboardingForm.tsx
src/app/api/onboarding/submit/route.ts
src/app/api/admin/review/route.ts
src/app/(authenticated)/layout.tsx
src/app/(authenticated)/page.tsx
src/lib/supabase-server.ts
```

Also inspect any Storage helpers, reviewer helpers, database helpers, migrations, and API authorization utilities introduced by Chat15.

Determine whether the source actually implements:

- real multipart/file upload;
- private Storage placement;
- technical file metadata capture;
- role/evidence server validation;
- reviewer authorization independent of `trusted_role`;
- short-lived signed URLs;
- review queue/page;
- atomic/concurrent-safe state transitions;
- evidence history/versioning;
- rejection/resubmission;
- minimal Driver/Company profile creation.

### C. Report accuracy

For every claim in the Chat15 implementation report, classify it:

```text
VERIFIED    = direct source/database/test evidence
INFERRED    = plausible from implementation but not directly demonstrated
UNKNOWN     = insufficient evidence
CONFLICT    = directly inconsistent with the locked plan
```

Do not upgrade INFERRED to VERIFIED merely because `npm run build` or `npx tsc --noEmit` succeeds.

## 7. Root-Cause Question

Determine why the implementation report and migration reflect the older raw-string/headless-review approach even though the current Chat15 plan was revised to require private Storage, dedicated reviewer authorization, signed URLs, and a minimal Reviewer Queue.

Possible causes must be investigated, not guessed. Examples include:

```text
old implementation prompt was executed;
Antigravity used an earlier plan/version;
plan synchronization through the Records bridge failed;
implementation report was generated from stale requirements;
partial implementation occurred but report overstated completion.
```

Do not select a root cause until source/history evidence supports it.

## 8. Safety Boundary

Until this investigation is resolved:

- Do not run another schema migration merely to patch `document_value`.
- Do not manually create Driver/Company records merely to make the dashboard work.
- Do not manually set `trusted_role` as a substitute for the reviewer workflow.
- Do not expose onboarding evidence publicly.
- Do not add `ADMIN` to `trusted_role`.
- Do not restore Driver Code as authentication.
- Do not issue a new implementation prompt based only on the current report.
- Do not mark Node 2 complete.

Small implementation bugs may be fixed after the investigation identifies the exact gap. A major architecture conflict must trigger reassessment rather than an ad-hoc patch.

## 9. Required Investigation Output

The resulting implementation investigation report must contain:

1. exact files inspected;
2. exact database objects inspected;
3. actual current behavior;
4. locked-plan requirement comparison;
5. VERIFIED / INFERRED / UNKNOWN / CONFLICT classification;
6. root cause of any stale implementation;
7. exact gaps requiring correction;
8. whether the existing migration can be safely superseded or must be replaced/reconciled;
9. whether any new architecture decision is required;
10. recommended next action.

The output must distinguish investigation from implementation. **No fix is authorized by this investigation itself.**

## 10. Current State

```text
Migration execution                = VERIFIED
Build/typecheck report             = VERIFIED as reported
Raw-string evidence conflict       = VERIFIED
Headless reviewer conflict         = VERIFIED
Private Storage implementation     = UNKNOWN / likely missing
Reviewer authorization             = UNKNOWN
Signed URL implementation          = UNKNOWN
Evidence versioning                = UNKNOWN
Transactional review               = UNKNOWN
Minimal Driver profile             = INFERRED
Minimal Company profile            = INFERRED
Node 2 acceptance                  = NOT COMPLETE
```

## 11. Next Action

Perform a source-and-database reconciliation against the current Chat15 locked implementation plan and return an evidence-backed investigation report. **Do not implement fixes during this investigation.**
