# Chat15 — Day 8 — Node 2 Company & Driver Onboarding Identity — V2 Implementation Plan

## Status

**Architecture:** LOCKED / no new architecture decision required  
**Implementation:** NOT YET EXECUTED  
**Purpose:** Correct the stale Chat15 implementation to the already-approved onboarding architecture  
**Node:** Node 2 — Authentication + Identity  
**Day:** Day 8  
**Chat:** Chat15

## 1. Authoritative Inputs

Use these Records as the implementation baseline:

```text
03_IMPLEMENTATION/plans/Chat15_Day8_Node2_Company_Driver_Onboarding_Identity_Implementation_Plan.md
05_DEBUGGING/investigations/Chat15_Day8_Node2_Implementation_Report_vs_Locked_Plan_Investigation.md
03_IMPLEMENTATION/implementation_reports/Chat15_Day8_Node2_Report_Implementation_vs_Locked_Plan.md
```

The V2 plan does not reopen the already-locked architecture. It converts the identified gaps into an implementation sequence.

## 2. Problem Being Corrected

The current Chat15 implementation uses:

```text
onboarding_evidence.document_value TEXT
headless review API
no dedicated reviewer authorization
no private evidence Storage
no signed evidence access
no evidence versioning
no atomic review transition
```

These are non-conformant with the locked design.

The existing migration has already been executed in Supabase. **Do not blindly DROP existing tables.** First reconcile the live schema/data with the replacement migration and preserve any useful existing data where technically appropriate.

## 3. Locked Architecture — MUST NOT CHANGE

### Identity

```text
Supabase Auth user
        ↓
freight_identities.auth_id
        ↓
verification_status + trusted_role
```

Option B remains locked:

```text
freight_identities.auth_id
        ↕
drivers.auth_id / companies.auth_id
```

`freight_identities` remains the canonical identity/verification/trusted-role context.

### Requested role

Signup explicitly supports:

```text
DRIVER
COMPANY
```

`requested_role` represents user intent and is never an authorization authority.

### Evidence

```text
DRIVER  → Driving Licence
COMPANY → GST evidence/details
```

The actual document must be stored in a **private Supabase Storage bucket**. The database stores protected metadata/reference, not the document contents as plain text.

Node 2 does not perform OCR or AI extraction of business fields.

### Review

```text
Reviewer authorization
        ↓
Reviewer Queue
        ↓
Specific review target
        ↓
Private evidence via short-lived signed URL
        ↓
Approve / Reject
```

Reviewer authorization must be independent of `trusted_role`. Do not add `ADMIN` to `trusted_role`.

### Profile creation

After successful verification:

```text
DRIVER  → minimal drivers record
COMPANY → minimal companies record
```

Driver marketplace fields belong to Node 4. Company trip/publishing fields belong to Node 3.

Driver Code is never an authentication credential.

## 4. Implementation Sequence

### Phase A — Schema reconciliation and migration

1. Inspect the exact current production/live schema and existing migration history.
2. Identify whether `onboarding_evidence` contains data and whether any `companies` rows were created.
3. Design a **replacement/superseding migration**, not a destructive blind reset.
4. Remove the obsolete plain-text `document_value` design safely.
5. Add the minimum evidence metadata/reference fields required by the locked architecture, including:
   - applicant/auth identity;
   - role type;
   - document type;
   - private Storage object path/reference;
   - MIME/content type;
   - file size;
   - submission status;
   - submission/version timestamp;
   - review provenance fields as required by the existing audit design.
6. Add immutable evidence-version semantics. A resubmission creates a new evidence record/version rather than overwriting the old submission.
7. Add `reviewer_authorizations` as a dedicated reviewer authority if absent.
8. Preserve `companies` if its existing minimal schema is compatible; modify only as necessary.
9. Preserve existing RLS architecture and add only the policies required for the new operations.
10. Create/configure a private Storage bucket and narrowly scoped Storage policies if the project migration mechanism supports them. Do not make evidence public.

### Phase B — Evidence submission

Refactor onboarding submission to use actual file upload.

Requirements:

- authenticated user only;
- derive `auth_id` from the authenticated session;
- derive/validate role from server-side identity state;
- accept only the permitted evidence type for the requested role;
- validate allowed MIME types and file size;
- upload the actual document to the private Storage bucket;
- generate a non-public deterministic/controlled object path that prevents arbitrary cross-user access;
- store Storage reference + technical metadata in `onboarding_evidence`;
- never store the actual document as a plain-text field;
- never allow client input to set `verification_status` or `trusted_role`;
- preserve prior submissions when resubmitting.

Technical metadata is limited to information the system can safely derive from the upload, such as MIME type, byte size, Storage path/reference, and timestamps. Node 2 does not infer Licence/GST business facts using OCR/AI.

### Phase C — Reviewer authorization bootstrap

Implement a controlled mechanism to authorize the initial reviewer.

Requirements:

- reviewer privilege must be represented independently from normal Driver/Company trusted roles;
- authorization must be checked server-side;
- ordinary client users must not be able to self-register as reviewers;
- no secret flag, hidden route, or client-supplied reviewer boolean may authorize access;
- service-role credentials may be used only on the trusted server side where necessary;
- document the bootstrap procedure and resulting reviewer identity in the implementation report.

### Phase D — Reviewer API

Replace the headless-only model with protected review operations used by the Reviewer UI.

Required operations:

```text
List pending applications
Get specific review target
Get authorized evidence access
Approve
Reject
```

Every operation must:

1. authenticate the caller;
2. verify `reviewer_authorizations` server-side;
3. validate the requested target;
4. prevent arbitrary evidence/object enumeration;
5. perform only permitted state transitions.

Approval:

```text
PENDING
   ↓
verify reviewer
   ↓
verify target/evidence
   ↓
atomic/transactionally consistent transition
   ↓
VERIFIED + trusted_role = server-derived requested role
   ↓
create/activate minimal role profile
   ↓
record reviewer + decision timestamp
```

Rejection:

```text
PENDING
   ↓
REJECTED
trusted_role = NULL
review reason + reviewer + timestamp recorded
```

Concurrent review requests must not allow two successful approvals of the same PENDING application.

Do not accept `trusted_role` from the client.

### Phase E — Private evidence access

Implement a server-side evidence viewing endpoint or equivalent.

Required flow:

```text
Authenticated reviewer
        ↓
reviewer authorization check
        ↓
specific evidence belongs to target
        ↓
short-lived signed Storage URL
```

The browser must never receive a permanent public URL.

Ordinary users and other applicants must not be able to access another user's evidence.

### Phase F — Reviewer UI

Build a minimal Reviewer Queue + Review page.

Queue should show at minimum:

- applicant/account identifier;
- requested role;
- verification status;
- submission timestamp;
- available evidence indicator.

Review page should show:

- application context;
- role;
- evidence type;
- secure evidence viewer/link using the short-lived signed URL;
- Approve action;
- Reject action requiring a reason;
- relevant previous review/evidence history.

UI must call protected server APIs. It must not directly mutate verification state.

### Phase G — Role-aware application boundary

Preserve and correct the existing routing behavior:

```text
PENDING
  → onboarding/pending boundary

REJECTED
  → rejection/resubmission boundary

VERIFIED + DRIVER
  → Driver context

VERIFIED + COMPANY
  → Company context
```

Do not default every verified user to Driver.

A role-specific business record must not independently grant authorization.

### Phase H — Profile provisioning

On successful approval only:

```text
DRIVER
  → minimal drivers record keyed by auth_id

COMPANY
  → minimal companies record keyed by auth_id
```

Do not introduce Node 3 trip fields or Node 4 marketplace fields merely to make Node 2 pass.

If an existing minimal Driver record is already present, reconcile rather than duplicate it.

## 5. Migration Safety Rules

The existing `005_create_onboarding_evidence.sql` has already been run.

Therefore:

```text
DO NOT:
- blindly DROP onboarding_evidence;
- blindly DROP companies;
- erase existing evidence/data without assessment;
- manually repair production state by ad-hoc SQL unless documented and authorized.
```

Instead:

```text
Inspect current data
    ↓
Determine compatibility/preservation requirements
    ↓
Create superseding migration
    ↓
Migrate/reconcile existing rows if necessary
    ↓
Remove obsolete field/design safely
    ↓
Verify schema + RLS + Storage
```

If the migration cannot safely preserve existing state without a material architecture decision, stop and report the blocker.

## 6. Required Source Areas

Inspect and modify only where required, including the currently identified onboarding/auth/review files:

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
src/db/migrations/
```

Also inspect any existing Storage, authorization, audit, database helper, and profile utilities before creating duplicates.

## 7. Security Requirements

Must preserve Node 1 security architecture.

Never trust client-supplied:

```text
trusted_role
verification_status
reviewer status
arbitrary auth_id
arbitrary driver_id/company_id
```

`requested_role` is not authorization.

RLS must not be weakened to make reviewer operations work.

Service-role credentials must remain server-only.

Private evidence must remain private.

Reviewer operations require dedicated server-side authorization.

Evidence object paths must not become an IDOR primitive.

## 8. Verification Requirements

### Build / static checks

Run the repository-supported checks, including where applicable:

```text
npm run build
npx tsc --noEmit
```

Run relevant lint/tests/migration checks supported by the repository.

### Database/security checks

Provide evidence for:

1. `freight_identities.auth_id` remains the identity join key.
2. `drivers.auth_id` and `companies.auth_id` remain role mappings.
3. `document_value` is no longer the document-storage mechanism.
4. Actual evidence is in private Storage.
5. Storage policies prevent public/unauthorized access.
6. `reviewer_authorizations` exists and is checked server-side.
7. RLS remains enabled and appropriate.
8. Reviewer operations cannot be invoked successfully by an ordinary user.
9. Reviewer can access only the intended evidence target.
10. Signed evidence URLs are short-lived.

### Functional checks

Demonstrate:

1. Driver signup.
2. Company signup.
3. Driver Licence file upload.
4. Company GST evidence upload/details.
5. Invalid role/evidence combination rejected.
6. New application is PENDING with `trusted_role = NULL`.
7. PENDING user cannot perform trusted role operations.
8. Reviewer Queue lists pending applications.
9. Reviewer opens one specific application.
10. Reviewer securely views private evidence.
11. Ordinary user cannot view reviewer queue.
12. Ordinary user cannot access another applicant's evidence.
13. Reviewer approves Driver.
14. Approval creates/activates minimal Driver profile.
15. Driver reaches Driver context.
16. Reviewer approves Company.
17. Approval creates/activates minimal Company profile.
18. Company reaches Company context.
19. Reviewer rejects an application with reason.
20. Rejected applicant sees rejection/resubmission boundary.
21. Resubmission creates a new evidence version.
22. Previous evidence/review history remains private.
23. Concurrent approval attempts cannot both succeed.
24. Client tampering cannot set `trusted_role` or `verification_status`.
25. Driver Code is not required for authentication.

## 9. Ayush Manual Verification — AFTER Implementation Evidence

Do not perform this until the implementation report demonstrates the required automated/database evidence.

Manual flow:

```text
1. Create/login Driver account.
2. Select DRIVER.
3. Upload Driving Licence file.
4. Confirm PENDING.
5. Create/login Company account.
6. Select COMPANY.
7. Upload GST evidence file/details.
8. Confirm PENDING.
9. Enter Reviewer Queue.
10. Open Driver application.
11. View its private evidence.
12. Approve Driver.
13. Confirm Driver context/profile.
14. Open Company application.
15. View its private evidence.
16. Approve Company.
17. Confirm Company context/profile.
18. Test rejection with a reason.
19. Resubmit corrected evidence.
20. Confirm previous evidence/review history remains.
21. Attempt unauthorized reviewer/evidence access.
22. Confirm Driver Code is not an authentication credential.
```

## 10. Required Implementation Report

Antigravity must create a new implementation report under:

```text
03_IMPLEMENTATION/implementation_reports/
```

The report must include:

- exact implementation files changed;
- exact migration(s) created/updated;
- exact Storage bucket/policy changes;
- reviewer authorization mechanism;
- exact API/UI changes;
- build/typecheck/lint/test commands and outputs;
- database verification evidence;
- security verification evidence;
- functional verification evidence;
- any migration/data-preservation action taken;
- manual verification status;
- VERIFIED / INFERRED / UNKNOWN classification.

Do not claim GitHub push unless evidence exists.

Do not claim Ayush manual verification unless Ayush has actually performed it.

## 11. Stop Conditions

Stop and return an investigation/blocker instead of inventing a workaround if:

- current live data cannot be safely reconciled;
- private Storage cannot be implemented without changing the locked architecture;
- reviewer authorization requires a different trust model;
- RLS must be weakened;
- evidence privacy cannot be guaranteed;
- atomic approval cannot be achieved with the current architecture;
- implementing the required behavior would make Driver Code an authentication credential;
- a new major domain/identity architecture is required.

Do not create a Subnode for known requirements in this plan. Follow the project Subnode rule only for significant unexpected work.

## 12. Completion Gate

This V2 implementation is accepted only when the evidence supports:

```text
Auth User
   ↓
freight_identities.auth_id
   ↓
requested_role
   ↓
role-specific actual document
   ↓
private Storage + metadata
   ↓
PENDING
   ↓
dedicated reviewer authorization
   ↓
Reviewer Queue / Review page
   ↓
secure signed evidence access
   ↓
atomic review decision
   ↓
VERIFIED + trusted_role
   ↓
minimal Driver/Company profile
   ↓
correct role-aware context
```

Node 2 remains incomplete until the implementation report and Ayush manual verification satisfy the completion gate.

## 13. Explicit Non-Goals

This V2 plan does not implement:

```text
Node 3 Company trip creation/publishing
Node 4 Driver marketplace/atomic trip claim
Node 5 Whole delivery tracking
Node 6 broader security/evidence hardening beyond required Node 2 foundation
Node 7 AI evidence-grounded summary
OCR/AI extraction of Licence/GST fields
Full Driver marketplace profile
Full Company operational/trip profile
```
