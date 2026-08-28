# Chat15 — Day 8 — Node 2 Company/Driver Onboarding + Verification Design Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat15  
**Day:** Day 8  
**Status:** DECISION — AYUSH APPROVED  

## 1. Purpose

Lock the Company/Driver onboarding and verification design required to continue Node 2 implementation after the Chat15 integration investigation.

This decision is based on the existing Node 1 / Node 2 identity model, the Chat15 investigation, and Ayush's explicit approval in Chat15.

## 2. Identity Foundation

The previously approved Chat15 decision selects **Option B** for role-specific mapping:

```text
Supabase Auth User
        │
        │ auth_id
        ▼
freight_identities
        │
        ├── trusted_role = DRIVER
        │       ↓
        │     drivers.auth_id
        │
        └── trusted_role = COMPANY
                ↓
              companies.auth_id
```

`freight_identities` is the canonical application identity / verification / trusted-role context.

`auth_id` is the stable authenticated-account join key.

`drivers` and `companies` are role-specific business/profile records and are not authentication authorities.

## 3. Signup Design

Signup must support both application roles.

The user-facing signup flow must provide:

```text
Email
Password
Requested Role:
  Driver
  Company
```

The selected role is stored as:

```text
requested_role = DRIVER | COMPANY
```

The requested role is user intent only. It does not grant authorization and must not be treated as proof of trusted role.

The current implementation's unconditional default to `DRIVER` is therefore an incomplete onboarding state and must be replaced by explicit role selection.

## 4. Evidence / Onboarding Design

After selecting a requested role, the user submits role-specific verification evidence.

### Driver

```text
requested_role = DRIVER
        ↓
Driving Licence evidence
        ↓
Submit
        ↓
PENDING
```

### Company

```text
requested_role = COMPANY
        ↓
GST details/evidence
        ↓
Submit
        ↓
PENDING
```

The hackathon MVP does not require automated government/KYC verification. Evidence is submitted for controlled human review.

## 5. Pending Identity State

After signup/onboarding submission:

```text
verification_status = PENDING
trusted_role = NULL
```

A PENDING identity may authenticate but is not a trusted Driver or Company.

A PENDING identity must not:

- perform trusted Driver-only operations;
- perform trusted Company-only operations;
- appear as an eligible Driver in marketplace queries;
- create/own protected business resources requiring a trusted role;
- obtain access by manipulating client-controlled role fields.

## 6. Evidence Storage

Evidence must be stored in a private Supabase Storage boundary with database metadata sufficient for the reviewer workflow.

The client must not be able to use evidence submission to establish authorization.

Conceptually:

```text
Authenticated user
       ↓
Role-specific onboarding form
       ↓
Private evidence storage
       ↓
Verification record / metadata
       ↓
Reviewer queue
```

Exact bucket naming, object-path format, MIME/size validation, retention policy, and signed-access mechanism are implementation details that must be designed without weakening the security boundary.

## 7. Verification Authority

The hackathon MVP uses a controlled human verifier: **Ayush**.

Approval/rejection must occur through a server-authorized path.

The user must not be able to directly modify:

```text
verification_status
trusted_role
```

Approval:

```text
PENDING
   ↓
VERIFIED
   ↓
trusted_role = requested_role
```

Rejection:

```text
PENDING
   ↓
REJECTED
   ↓
trusted_role = NULL
```

The reviewer workflow must not rely on a browser-only trust decision.

## 8. Role-Specific Profile Creation Timing — APPROVED

The role-specific application profile is created or activated **only after successful verification**.

Therefore:

```text
PENDING
→ generic Freight Identity only
```

and:

```text
VERIFIED + trusted_role = DRIVER
→ Driver profile may be created/activated
```

or:

```text
VERIFIED + trusted_role = COMPANY
→ Company profile may be created/activated
```

This prevents an unverified identity from being represented as an active role-specific identity merely because it requested that role.

## 9. Authenticated Application Routing

The authenticated application must no longer assume that every authenticated user is a Driver.

The server-side application context must first resolve:

```text
Authenticated Supabase User
        ↓
freight_identities by auth_id
        ↓
verification_status + trusted_role
```

Then:

```text
trusted_role = DRIVER
        ↓
Driver application context
        ↓
drivers.auth_id
```

or:

```text
trusted_role = COMPANY
        ↓
Company application context
        ↓
companies.auth_id
```

For PENDING / REJECTED / missing trusted role, the user remains at the appropriate onboarding/verification boundary.

This replaces the legacy behavior in which the authenticated page directly assumes a Driver profile.

## 10. Authentication Credential Boundary

Driver Code is not an authentication credential.

Authentication remains based on the Supabase Auth credential/session (Email + Password for the current MVP).

Driver Code may be a business identifier, but it must not be used as proof of authentication or trusted role.

## 11. Security Rules

The implementation must preserve:

- server-derived authenticated identity;
- server-controlled trusted role;
- PENDING cannot access trusted role operations;
- client-supplied `role`, `trusted_role`, `verification_status`, `driver_id`, or `company_id` cannot escalate privileges;
- `service_role` credentials remain server-only;
- role-specific business authorization remains subject to Node 1 authorization rules.

## 12. Reviewer / Audit Boundary

The reviewer must be able to determine at minimum:

- which identity is being reviewed;
- requested role;
- submitted evidence;
- current verification state;
- approval or rejection action.

An audit record should capture reviewer identity and decision time. The exact schema and UI are implementation details, but the implementation must not silently omit decision provenance if an audit record is required by the final Node 2 acceptance tests.

## 13. Error / State Expectations

The application should distinguish these states:

```text
No authenticated session
        → authentication boundary

Authenticated + PENDING
        → onboarding / pending verification

Authenticated + REJECTED
        → rejected/retry boundary

Authenticated + VERIFIED + DRIVER
        → Driver application

Authenticated + VERIFIED + COMPANY
        → Company application
```

A verified Driver without a Driver profile is a data-integrity/application-mapping problem to be surfaced and investigated, not a reason to bypass identity verification.

## 14. Scope Boundary

This design covers Node 2 identity/onboarding integration only.

It does not redefine:

- trip ownership;
- trip creation/publishing;
- Driver marketplace discovery;
- atomic first-valid claim;
- delivery lifecycle;
- immutable delivery evidence timeline;
- AI evidence-grounded summary;
- Node 1 business-resource authorization rules.

## 15. Implementation Acceptance Direction

The next implementation must demonstrate at minimum:

1. Signup allows Driver or Company selection.
2. Selected role is recorded as requested role only.
3. Driver onboarding collects Driving Licence evidence.
4. Company onboarding collects GST details/evidence.
5. New identity starts PENDING with no trusted role.
6. PENDING users cannot access trusted role operations.
7. Reviewer approval/rejection is server-controlled.
8. Approval establishes VERIFIED + trusted role.
9. Rejection leaves the user without trusted role.
10. Role-specific profile creation/activation occurs only after successful verification.
11. Authenticated application routing is role-aware.
12. Driver and Company application records use `auth_id` under the approved Option B mapping.
13. Driver Code is not used as an authentication credential.
14. Client-supplied identity/role fields cannot escalate authorization.
15. Evidence access remains protected/private.
16. Verification decision provenance is recorded as required by the final implementation design.

## 16. Decision Status

```text
Option B identity mapping                    = APPROVED / LOCKED
Company/Driver role selection                = APPROVED / LOCKED
Driver evidence = Driving Licence            = APPROVED / LOCKED
Company evidence = GST                       = APPROVED / LOCKED
PENDING boundary                             = APPROVED / LOCKED
Human verification                          = APPROVED / LOCKED
Profile creation after approval              = APPROVED / LOCKED
Role-aware application routing               = APPROVED / LOCKED
Driver Code authentication boundary          = APPROVED / LOCKED
Evidence storage implementation details      = OPEN FOR IMPLEMENTATION DESIGN
Reviewer UI/API implementation details       = OPEN FOR IMPLEMENTATION DESIGN
Final Node 2 contract                        = NOT YET FULLY LOCKED
Next implementation prompt                   = NOW ELIGIBLE TO BE CREATED
```
