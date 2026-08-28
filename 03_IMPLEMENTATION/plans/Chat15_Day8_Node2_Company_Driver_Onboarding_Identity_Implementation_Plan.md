# Node 2 Company & Driver Onboarding Identity Implementation Plan

## Status

**Architecture decisions:** LOCKED / Ayush approved  
**Implementation:** NOT YET EXECUTED  
**Node:** Node 2 — Authentication + Identity  
**Day:** Day 8  
**Chat:** Chat15

## Goal

Implement the approved Company/Driver onboarding, evidence collection, verification, and role-aware application routing flow without changing the locked identity or authorization architecture.

The authoritative identity boundary is:

```text
Supabase Auth User
        ↓
freight_identities by auth_id
        ↓
verification_status + trusted_role
        ↓
Driver / Company application context
```

The approved role-specific mapping is:

```text
freight_identities.auth_id
        ↕
drivers.auth_id / companies.auth_id
```

`freight_identities` is the canonical identity / verification / trusted-role context. `drivers` and `companies` are role-specific business/profile records, not authentication authorities.

## Locked Decisions — MUST PRESERVE

### 1. Identity mapping

**Option B is locked.** Use the authenticated Supabase Auth UUID as the stable `auth_id` join key. Do not introduce a new `identity_id` relationship into role tables.

### 2. Signup role

Signup must explicitly allow:

```text
DRIVER
COMPANY
```

Store the user's selection as `requested_role`. `requested_role` is user intent only and is not authorization.

### 3. Role/evidence consistency

The requested role determines the permitted evidence type.

```text
DRIVER  → Driving Licence
COMPANY → GST evidence/details
```

The UI should guide the user, but the **server must enforce** the role/evidence match. Invalid combinations must be rejected.

### 4. Evidence storage

Do **not** store the actual Driving Licence/GST document as plain text in the database.

Use a **private Supabase Storage bucket** for the actual file.

Store protected metadata/reference in the database, for example:

```text
auth_id
role_type
onboarding/review identity reference
document_type
storage_path
status
submitted_at
reviewed_at
reviewer_id
```

Use the actual existing schema/naming where possible. Do not invent duplicate identity authorities.

### 5. Metadata extraction

Node 2 records technical file metadata such as file type/size/path/upload time as appropriate.

Node 2 does **not** perform OCR or AI extraction of Licence/GST business fields.

The authorized reviewer manually verifies the document.

Future OCR/AI extraction requires a separate design decision and must never become the authorization authority by itself.

### 6. Initial verification state

After signup/onboarding submission:

```text
verification_status = PENDING
trusted_role = NULL
```

PENDING users may authenticate but cannot perform trusted Driver/Company operations.

### 7. Reviewer authorization

There is currently no existing reviewer/admin authorization infrastructure in the source code.

Create a dedicated reviewer authorization mechanism, separate from `freight_identities.trusted_role`.

The approved direction is a dedicated `reviewer_authorizations` table/authority checked server-side.

Do not add `ADMIN` to `trusted_role`.

Do not use a hidden UI, secret URL, or client-supplied flag as authorization.

Bootstrap Ayush as the initial reviewer through a controlled setup path, not through the normal user-facing review API.

### 8. Reviewer UI

Build a **minimal Reviewer Queue + Review page**, not a large generic Admin Dashboard.

Minimum review information:

- applicant/account identifier;
- requested role;
- verification status;
- submitted evidence;
- submission time;
- previous review history.

Actions:

```text
APPROVE
REJECT
```

Rejection requires a review reason.

The UI is not the security boundary. Every review operation must perform the server-side reviewer authorization check.

### 9. Private evidence viewing

Evidence remains private in Supabase Storage.

For a specific evidence object, the server must:

```text
Authenticate caller
      ↓
Check reviewer_authorizations
      ↓
Check requested evidence belongs to the review target
      ↓
Generate short-lived signed URL
```

Do not make evidence publicly accessible.

Do not allow arbitrary evidence-ID enumeration.

### 10. Approval transaction

Approval must be a server-authorized, transactionally consistent operation.

Conceptually:

```text
Reviewer
  ↓
Authenticate
  ↓
Check reviewer authorization
  ↓
Check application is still PENDING
  ↓
Verify required evidence exists
  ↓
BEGIN TRANSACTION
  ↓
verification_status = VERIFIED
trusted_role = stored requested_role
create/activate role-specific profile
record reviewer + decision time
  ↓
COMMIT
```

Do not accept `trusted_role` from the client.

Do not allow concurrent review requests to produce two successful transitions from PENDING.

Rejection must similarly be server-authorized and consistent:

```text
PENDING → REJECTED
trusted_role = NULL
record rejection reason + reviewer + timestamp
```

### 11. Evidence history / retention

Evidence submissions are immutable/versioned and must not be overwritten.

If a user resubmits after rejection, create a new evidence submission/version.

Previous rejected evidence and decisions remain private historical records rather than being deleted immediately.

Do not invent a legal retention period. Automatic deletion requires a separately defined project retention policy.

### 12. Rejection / resubmission

The approved state machine is:

```text
PENDING
  ↓
APPROVE → VERIFIED

PENDING
  ↓
REJECT → REJECTED
             ↓
          resubmit
             ↓
          PENDING
```

A rejected user may correct/resubmit evidence. The user cannot directly modify `verification_status` or `trusted_role`.

### 13. Role-specific profile timing

Create/activate role-specific profiles **only after successful verification**.

For Driver:

```text
VERIFIED + DRIVER
        ↓
minimal drivers record
        ↓
drivers.auth_id
```

For Company:

```text
VERIFIED + COMPANY
        ↓
minimal companies record
        ↓
companies.auth_id
```

The full Driver business/marketplace profile remains future Node 4 scope.

The full Company business/trip profile remains future Node 3 scope.

Verification does not by itself mean marketplace eligibility.

### 14. Driver Code

Driver Code may be generated as a business identifier after Driver approval if required by the existing application, but:

```text
Driver Code ≠ authentication credential
```

Do not restore Driver Code/Driver ID as a login mechanism.

### 15. Role-aware application routing

Replace the current unconditional Driver assumption.

Resolve:

```text
Authenticated Supabase User
        ↓
freight_identities by auth_id
        ↓
verification_status + trusted_role
```

Then:

```text
VERIFIED + DRIVER  → Driver application/context
VERIFIED + COMPANY → Company application/context
PENDING            → onboarding/pending boundary
REJECTED           → rejected/retry boundary
No trusted identity → verification boundary
```

A role-specific profile must not independently establish trusted role.

## Proposed Implementation Changes

### Database foundation

Create only the schema required to support the locked architecture and onboarding workflow.

Potential additions include:

```text
reviewer_authorizations
onboarding_evidence / evidence submission records
companies
```

and any required audit/provenance structure.

Before creating migrations, inspect the current schema and existing migrations. Preserve existing data and constraints. Do not create duplicate identity authorities.

For evidence, use a private Storage bucket/object path and database metadata rather than a plain-text document field.

For Company and Driver, use the existing model if present; where a minimal Company model is genuinely absent, create only the minimal Node 2 profile foundation needed for `auth_id` mapping. Do not design Node 3 trip fields or Node 4 marketplace fields here.

### Signup

Modify:

```text
src/app/signup/page.tsx
```

to provide explicit Driver/Company selection and submit `requested_role` through the authenticated signup flow.

The server/database path must validate the resulting role state rather than trusting arbitrary client role fields.

### Onboarding

Create/modify the authenticated onboarding flow so PENDING users can:

- see their requested role;
- submit the required evidence;
- receive a clear pending state;
- resubmit after rejection.

Role/evidence mismatch must be rejected server-side.

### Evidence submission API

Create/modify the onboarding submission API so it:

- requires an authenticated user;
- derives identity from the trusted Auth session;
- validates role/evidence compatibility;
- validates file type/size as appropriate;
- stores actual evidence in private Storage;
- records metadata/reference in the database;
- prevents users from setting verification/trusted-role state.

### Reviewer authorization bootstrap

Create the controlled bootstrap mechanism for the first reviewer account (Ayush).

Do not bootstrap reviewer privilege through the normal client application or review endpoint.

Document exactly how the bootstrap is performed and how the reviewer identity is determined.

### Reviewer API

Create a protected review API that:

1. obtains the authenticated server-side user;
2. checks `reviewer_authorizations` server-side;
3. validates the target application/evidence;
4. performs the approved state transition transactionally/consistently;
5. creates/activates the appropriate minimal role profile after approval;
6. records reviewer decision provenance;
7. returns an appropriate result.

Use `service_role` only server-side and only where the operation genuinely requires privileged access.

### Reviewer UI

Create a minimal queue and review page. It must use the protected reviewer APIs rather than directly mutating verification state from the browser.

Evidence viewing must use short-lived signed URLs generated only after reviewer authorization and target validation.

### Authenticated layout/dashboard

Remove the old unconditional Driver assumption.

PENDING users should be restricted to onboarding/pending state.

REJECTED users should reach the rejection/resubmission boundary.

VERIFIED users should be routed according to server-derived `trusted_role`.

### Profile creation

On approval:

```text
DRIVER  → minimal drivers profile by auth_id
COMPANY → minimal companies profile by auth_id
```

Do not make either profile the source of truth for trusted role.

## Security Requirements

The implementation must preserve the existing Node 1 security architecture.

Never trust client-supplied:

```text
role
requested_role as authorization
trusted_role
verification_status
driver_id
company_id
reviewer status
```

`requested_role` is data about what the user requested; `trusted_role` is server-controlled verification output.

RLS must not be weakened merely to make onboarding/reviewer operations work.

Service-role credentials must never be exposed to the browser.

Private evidence must not be publicly readable.

Reviewer access must be explicitly authorized server-side.

## Verification Plan

Compilation alone is not sufficient.

### Automated / implementation evidence

Run the repository's applicable:

```text
npm run build
npx tsc --noEmit
```

plus relevant lint/tests/migration validation already supported by the repository.

Record exact commands and results.

### Functional verification

Demonstrate evidence for:

1. Driver signup with `requested_role = DRIVER`.
2. Company signup with `requested_role = COMPANY`.
3. Driver Licence submission.
4. GST evidence/details submission.
5. Invalid role/evidence combination rejected.
6. New onboarding identity starts PENDING with `trusted_role = NULL`.
7. PENDING user cannot access trusted Driver/Company operations.
8. Reviewer account is authorized through the dedicated reviewer mechanism.
9. Unauthorized user receives denial from reviewer API.
10. Reviewer can securely view a specific evidence object using short-lived access.
11. Public/other-user evidence access is denied.
12. Approval produces VERIFIED + server-derived trusted role.
13. Approval creates/activates minimal role profile.
14. Rejection produces REJECTED + `trusted_role = NULL` and stores reason.
15. Rejected applicant can submit a new evidence version and return to PENDING.
16. Previous evidence/review history remains available privately.
17. Concurrent review attempts cannot both successfully transition the same PENDING application.
18. Driver routes to Driver context after verification.
19. Company routes to Company context after verification.
20. Driver Code is not used for authentication.
21. Client role/status tampering cannot escalate authorization.

### Ayush manual verification

Clearly separate implementation evidence from manual verification.

Ayush should manually verify:

1. Sign up as Driver.
2. Upload Driving Licence.
3. Observe Pending state.
4. Sign up as Company with a separate account.
5. Upload GST evidence/details.
6. Observe Pending state.
7. Log in as reviewer and open Reviewer Queue.
8. Open a specific private evidence document.
9. Approve Driver application.
10. Confirm Driver application/context appears.
11. Approve Company application.
12. Confirm Company application/context appears.
13. Test rejection with a review reason.
14. Resubmit corrected evidence.
15. Confirm previous review history remains.
16. Confirm ordinary user cannot access reviewer operations/evidence.
17. Confirm Driver Code is not required for login.

Do not mark manual verification as complete until Ayush performs it.

## Stop Conditions

Stop and report instead of silently redesigning if:

- the approved Option B mapping cannot be implemented without a new architecture decision;
- existing schema materially conflicts with the locked identity model;
- existing RLS requires weakening;
- reviewer authorization requires a fundamentally different security architecture;
- private evidence handling requires a major architecture change;
- Company identity requires a major domain redesign;
- implementation would restore Driver Code as authentication;
- implementation would make `drivers` or `companies` the identity authority.

Small bugs should be fixed inside Node 2. Do not create a Subnode for a known core requirement. Follow the roadmap Subnode rule for significant unexpected work.

## Completion Gate

Do not declare Node 2 complete from build success alone.

Completion requires implementation evidence showing:

```text
Auth User
   ↓
Freight Identity
   ↓
requested_role
   ↓
role-specific evidence
   ↓
PENDING
   ↓
server-authorized review
   ↓
VERIFIED + trusted_role
   ↓
minimal Driver/Company profile
   ↓
correct role-aware application context
```

and preservation of:

```text
Freight Identity = canonical identity / verification / trusted-role context
Auth ID          = stable authenticated-account join key
Driver/Company   = role-specific business records
Driver Code      ≠ authentication credential
```

Final implementation report must distinguish VERIFIED / INFERRED / UNKNOWN and must not claim GitHub push, PR, or manual verification without evidence.
