# Chat15 — Day 8 — Node 2 Company/Driver Onboarding + Identity Integration Implementation

## EXECUTION TYPE

**Implementation Prompt — Antigravity**

This prompt is issued only after the following Chat15 architecture decisions were explicitly approved by Ayush and recorded in the Records repository.

## AUTHORITATIVE RECORDS

Read these Records before changing source code:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
02_ARCHITECTURE/Chat15_Day8_Node2_Identity_Mapping_OptionB_Decision.md
02_ARCHITECTURE/Chat15_Day8_Node2_Onboarding_Verification_Design_Decision.md
05_DEBUGGING/investigations/Chat15_Day8_Node2_Company_Driver_Onboarding_Integration_Investigation.md
03_IMPLEMENTATION/implementation_reports/Chat14_Day7_Node2_Report_Authentication_Identity_Implementation.md
03_IMPLEMENTATION/implementation_reports/Chat15_Day8_Node2_Report_Company_Driver_Onboarding_Integration.md
```

The Records repository is the bridge between the reasoning/architecture process and implementation. Do not invent architecture that conflicts with these records.

## SOURCE REPOSITORY

Implement in:

```text
ayush22cp008/freight_hackathon
```

Use the current source repository state as implementation evidence. Do not assume local-only changes are pushed to GitHub.

---

# 1. OBJECTIVE

Complete the Node 2 Authentication + Identity onboarding integration so the application supports the approved Company/Driver identity model instead of assuming every authenticated user is a Driver.

The implementation must establish this boundary:

```text
Supabase Auth User
        ↓
freight_identities by auth_id
        ↓
verification_status + trusted_role
        ↓
Role-aware application context
        ├── DRIVER  → Driver business/profile context
        └── COMPANY → Company business/profile context
```

The implementation must also add the approved onboarding path:

```text
Signup
  ↓
Email + Password
  ↓
Choose requested role
  ├── DRIVER
  │     ↓
  │   Driving Licence evidence
  │
  └── COMPANY
        ↓
      GST details/evidence
  ↓
PENDING
  ↓
Human review
  ├── APPROVE → VERIFIED + trusted_role
  └── REJECT  → REJECTED + no trusted_role
  ↓
Role-specific profile created/activated only after approval
```

---

# 2. NON-NEGOTIABLE ARCHITECTURE

## 2.1 Option B is locked

Use the authenticated Supabase Auth UUID (`auth_id`) as the stable join key.

Conceptually:

```text
freight_identities.auth_id
        ↕
Supabase Auth User ID
        ↕
drivers.auth_id / companies.auth_id
```

Do not introduce `identity_id` references into role tables unless an explicit new architecture decision is created outside this prompt.

## 2.2 Freight Identity is canonical

`freight_identities` is the canonical application identity / verification / trusted-role context.

`drivers` and `companies` are role-specific business/profile records.

Neither role-specific table may independently establish the user's trusted role.

The application must derive trusted role from the server-controlled Freight Identity state.

## 2.3 Driver Code is not authentication

Do not restore Driver Code/Driver ID as a login credential.

Authentication remains Supabase Auth using the current Email + Password MVP flow.

---

# 3. REQUIRED IMPLEMENTATION

## A. Signup role selection

Replace the tested unconditional Driver-request behavior with explicit user-facing role selection:

```text
Driver
Company
```

Store the selection as:

```text
requested_role = DRIVER | COMPANY
```

Do not treat `requested_role` as authorization.

The user must not be able to submit an arbitrary trusted role or verification status.

## B. Driver onboarding evidence

For a Driver request, collect Driving Licence evidence.

Requirements:

- authenticated submission;
- private evidence storage;
- server-side validation of relevant metadata and access;
- evidence linked to the identity/review record;
- user cannot mark their own evidence as verified;
- evidence must not become an authentication credential.

## C. Company onboarding evidence

For a Company request, collect GST details/evidence.

Requirements mirror Driver onboarding:

- authenticated submission;
- private evidence storage;
- server-side validation;
- evidence linked to the identity/review record;
- user cannot approve their own evidence;
- GST evidence is verification evidence, not an authentication credential.

Do not invent automated government verification.

## D. Verification state

A new onboarding identity must remain:

```text
verification_status = PENDING
trusted_role = NULL
```

until an authorized reviewer approves it.

Approval must establish:

```text
verification_status = VERIFIED
trusted_role = requested_role
```

Rejection must establish:

```text
verification_status = REJECTED
trusted_role = NULL
```

These transitions must be server-controlled.

## E. Reviewer workflow

Implement the minimum hackathon MVP reviewer path needed for Ayush to inspect and decide verification.

The reviewer must be able to determine:

- identity/account under review;
- requested role;
- submitted evidence;
- current verification status;
- approve/reject action.

Record reviewer identity and decision time in an audit/provenance record appropriate to the existing schema.

Do not expose privileged reviewer operations to ordinary users.

Do not place service-role credentials in browser/client code.

## F. Role-specific profile timing

Create or activate role-specific business/profile records only after successful verification.

Therefore:

```text
PENDING
→ generic Freight Identity only
```

and:

```text
VERIFIED + trusted_role = DRIVER
→ Driver profile creation/activation
```

or:

```text
VERIFIED + trusted_role = COMPANY
→ Company profile creation/activation
```

Use the approved Option B `auth_id` relationship for these role-specific records.

Do not create fake Driver profiles merely to satisfy the current dashboard.

## G. Replace Driver-only authenticated routing

The existing authenticated application must stop assuming every authenticated user is a Driver.

The server-side/authenticated application context must first resolve:

```text
Auth User ID
   ↓
freight_identities.auth_id
   ↓
verification_status + trusted_role
```

Then route/load the appropriate application context:

```text
VERIFIED + DRIVER
   → Driver application

VERIFIED + COMPANY
   → Company application

PENDING
   → Pending/onboarding boundary

REJECTED
   → Rejected/retry boundary

No trusted identity
   → verification boundary
```

A verified Driver may resolve a Driver business profile through `drivers.auth_id`.

A verified Company may resolve a Company business profile through `companies.auth_id`.

Do not use the existence of a Driver row as the source of truth for the user's trusted role.

---

# 4. SECURITY REQUIREMENTS

Preserve all Node 1 authorization boundaries.

The implementation must prevent client-controlled escalation through:

```text
role
requested_role
trusted_role
verification_status
driver_id
company_id
identity IDs
```

The server must derive authenticated identity from the trusted Auth session.

The server must derive trusted role from the verified Freight Identity.

RLS and API authorization must remain consistent with the existing Node 1 decisions.

Do not weaken existing RLS policies merely to make onboarding work.

Do not expose private evidence files publicly.

Use protected/signed access appropriate to the existing application architecture for reviewer evidence viewing.

---

# 5. DATABASE / MIGRATION RULES

Before writing migrations:

1. Inspect the current source migrations and actual expected schema.
2. Reuse the existing `freight_identities.auth_id` naming.
3. Do not rename `auth_id` to `auth_user_id` merely because an earlier test query used that name.
4. Preserve existing data and constraints.
5. Add only the schema required for the approved onboarding/reviewer/evidence workflow.
6. Do not duplicate identity tables.

If a schema conflict with the approved architecture is discovered, stop and report it instead of silently redesigning the model.

---

# 6. UX / STATE REQUIREMENTS

The user should be able to understand which state they are in.

Minimum states:

```text
Unauthenticated
→ Sign in / Sign up

Signup
→ Select Driver or Company

Driver onboarding
→ Submit Driving Licence

Company onboarding
→ Submit GST details/evidence

Pending
→ Verification pending

Rejected
→ Rejected / retry or correction boundary

Verified Driver
→ Driver application

Verified Company
→ Company application
```

Do not expose an internal database status without a useful user-facing explanation.

---

# 7. TESTING / ACCEPTANCE EVIDENCE

Do not declare completion from compilation alone.

Provide evidence for all of the following.

### Test 1 — Driver signup

Create a fresh test account and select Driver.

Verify:

```text
requested_role = DRIVER
verification_status = PENDING
trusted_role = NULL
```

Verify Driving Licence evidence is associated with the identity/review record.

### Test 2 — Company signup

Create a fresh test account and select Company.

Verify:

```text
requested_role = COMPANY
verification_status = PENDING
trusted_role = NULL
```

Verify GST evidence/details are associated with the identity/review record.

### Test 3 — Pending access

Confirm PENDING users cannot enter trusted Driver or Company operations.

### Test 4 — Reviewer approval

Using the authorized reviewer path:

```text
PENDING → VERIFIED
trusted_role = requested_role
```

Verify the audit/provenance record.

### Test 5 — Reviewer rejection

Verify:

```text
PENDING → REJECTED
trusted_role = NULL
```

Verify the rejected user does not receive trusted Driver/Company access.

### Test 6 — Driver routing

For a verified Driver:

```text
Freight Identity → DRIVER → Driver context
```

Verify the application no longer relies on the old unconditional Driver assumption.

### Test 7 — Company routing

For a verified Company:

```text
Freight Identity → COMPANY → Company context
```

Verify the application does not attempt to load a Driver profile as the primary application identity.

### Test 8 — Authentication boundary

Verify Driver Code is not required for authentication.

### Test 9 — Authorization tampering

Attempt client-side manipulation of role/trusted role/verification state and confirm it cannot escalate access.

### Test 10 — Evidence privacy

Confirm ordinary users cannot publicly fetch another user's private evidence.

### Test 11 — Build/test

Run the relevant project build, type checks, lint/tests, and migration validation available in the repository.

Report exact commands and outcomes.

---

# 8. MANUAL AYUSH VERIFICATION

Separate automated/build evidence from Ayush manual verification.

The final report must explicitly identify:

```text
AUTOMATED / IMPLEMENTATION EVIDENCE
vs
AYUSH MANUAL VERIFICATION REQUIRED
```

Do not claim manual verification has happened unless Ayush actually performed it.

Provide a concise manual test checklist for Ayush covering at minimum:

1. Driver signup.
2. Company signup.
3. Driver evidence submission.
4. Company evidence submission.
5. Pending state.
6. Reviewer approval.
7. Reviewer rejection.
8. Verified Driver routing.
9. Verified Company routing.
10. Logout/login behavior.

---

# 9. INVESTIGATION / STOP CONDITIONS

Stop implementation and report if any of the following occurs:

- the approved Option B architecture cannot be implemented without a new architecture decision;
- existing schema contradicts the locked identity model;
- existing RLS prevents the intended secure reviewer/evidence workflow and would need weakening;
- a new role/authorization model is required beyond the locked design;
- evidence handling requires a materially different security architecture;
- the existing Company model is absent in a way that requires a major domain redesign;
- implementation would require treating Driver Code as authentication;
- implementation would require making `drivers` the identity authority again.

Do not create a Subnode merely because a small bug is found. Fix small bugs inside Node 2. Create a Subnode only for significant unexpected work according to the project roadmap rules.

---

# 10. IMPLEMENTATION REPORT REQUIREMENTS

After implementation, create a new report in the Records repository under:

```text
03_IMPLEMENTATION/implementation_reports/
```

Use a Chat15 / Day8 / Node2 filename.

The report must contain:

1. Executive summary.
2. Files changed.
3. Database migrations added/changed.
4. Identity/onboarding architecture implemented.
5. Evidence-storage implementation.
6. Reviewer workflow.
7. Security controls.
8. Tests run and exact results.
9. Database verification evidence.
10. Manual verification checklist for Ayush.
11. Known limitations.
12. Any unresolved issues.
13. Git commit(s)/PR information if available.
14. Explicit status using VERIFIED / INFERRED / UNKNOWN where appropriate.

Do not claim GitHub push/PR completion unless actual Git evidence exists.

---

# 11. FINAL COMPLETION GATE

Node 2 implementation is not complete merely because the application builds.

The implementation is eligible for final Node 2 verification only when the evidence demonstrates:

```text
Auth User
   ↓
Freight Identity
   ↓
requested role
   ↓
role-specific evidence
   ↓
PENDING
   ↓
server-authorized review
   ↓
VERIFIED + trusted_role
   ↓
correct Driver/Company application context
```

and the following architectural invariant remains true:

```text
Freight Identity = canonical identity / verification / trusted-role context
Auth ID          = stable authenticated-account join key
Driver/Company   = role-specific business records
Driver Code      ≠ authentication credential
```

Do not declare Node 2 complete until implementation evidence and Ayush manual verification requirements are clearly separated.
