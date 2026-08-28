# Chat15 — Day 8 — Node 2 Identity Mapping Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat15  
**Day:** Day 8  
**Status:** DECISION — AYUSH APPROVED  

## 1. Decision

Ayush approved **Option B** for the relationship between the canonical Freight Identity and role-specific application records.

### Option B

Keep the authenticated Supabase user UUID (`auth_id`) as the stable join key for the generic Freight Identity and role-specific application records.

Conceptually:

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

## 2. Architectural Rule

`freight_identities` is the canonical application identity/verification context.

`auth_id` is the stable authenticated-account join key.

Role-specific `drivers` and `companies` records are business/profile records. They are **not authentication authorities** and must not independently determine the user's trusted role.

The trusted role must be derived from the server-controlled `freight_identities.trusted_role` after verification.

Therefore the intended request flow is:

```text
Authenticated Supabase session
        ↓
Authenticated Auth User ID
        ↓
Freight Identity lookup by auth_id
        ↓
verification_status + trusted_role
        ↓
Role-aware application context
        ↓
Driver or Company business profile by auth_id
        ↓
Node 1 authorization
```

## 3. Why Option B Was Selected

### 3.1 Existing system compatibility

The existing Driver model already uses `drivers.auth_id` to relate a Driver record to the authenticated Supabase user. Option B allows the project to migrate from the legacy Driver-only authentication assumption without unnecessarily replacing this established relationship.

### 3.2 Lower migration complexity

Option A would introduce an additional identity-reference layer:

```text
auth.users.id
      ↓
freight_identities.id
      ↓
drivers.identity_id / companies.identity_id
```

Option B preserves the existing Auth UUID as the common join key:

```text
auth.users.id
      ↓
freight_identities.auth_id
      ↓
drivers.auth_id / companies.auth_id
```

For the current Freight architecture, the additional mapping layer of Option A is not justified by the evidence currently available.

### 3.3 Preserves the new identity boundary

Choosing Option B does **not** restore the old model in which `drivers` is the authentication identity.

The separation remains:

```text
auth.users
  = authenticated account

freight_identities
  = canonical application identity + verification + trusted role

drivers / companies
  = role-specific business data
```

This preserves the Node 1 / Node 2 intent while minimizing unnecessary schema churn.

## 4. Explicit Non-Goals

This decision does not authorize:

- treating Driver Code as an authentication credential;
- assuming every authenticated user is a Driver;
- granting Driver or Company authorization from `drivers` or `companies` alone;
- trusting client-supplied `role`, `trusted_role`, `driver_id`, or `company_id`;
- bypassing Freight Identity verification;
- manually creating Driver records merely to make the current dashboard work.

## 5. Required Application Behavior

The legacy authenticated dashboard behavior must be replaced with role-aware application resolution.

The application must first resolve the authenticated user's Freight Identity and inspect the server-derived trusted role.

Conceptually:

```text
if trusted_role = DRIVER
    → Driver application context

if trusted_role = COMPANY
    → Company application context

if PENDING / REJECTED / no trusted role
    → verification/onboarding boundary
```

A verified Driver may then have a Driver business/profile record resolved through `drivers.auth_id`.

A verified Company may have a Company business/profile record resolved through `companies.auth_id`.

The exact creation/mapping behavior for those role-specific records remains part of the onboarding implementation design and must not be invented by implementation agents.

## 6. Relationship to Signup and Verification

The signup flow must allow the user to request either:

```text
requested_role = DRIVER
```

or:

```text
requested_role = COMPANY
```

Requested role is user intent only.

After evidence review and approval:

```text
verification_status = VERIFIED
trusted_role = requested_role
```

Only then should role-specific application access be established.

The hackathon MVP evidence model remains:

```text
Driver  → Driving Licence evidence
Company → GST evidence/details
```

The exact evidence storage, reviewer interface, audit trail, and role-profile creation sequence remain open until explicitly decided.

## 7. Implementation Boundary

This record is an architecture decision. It does **not** by itself authorize source-code implementation.

Before the next implementation prompt is created, the complete onboarding/evidence/reviewer workflow must be explicitly designed and reconciled with this decision.

The next implementation prompt must require Antigravity to preserve this boundary:

```text
Freight Identity = canonical identity/verification/trusted-role context
Auth ID          = stable authenticated-account join key
Driver/Company   = role-specific business records
```

## 8. Decision Status

```text
Option A vs Option B review       = COMPLETE
Option B recommendation           = MADE
Ayush approval                    = APPROVED
Identity mapping decision         = LOCKED FOR NEXT NODE 2 DESIGN
Complete Node 2 contract          = NOT YET LOCKED
Onboarding/evidence design        = OPEN
Implementation prompt             = NOT YET AUTHORIZED
```
