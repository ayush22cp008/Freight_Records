# Chat15 — Day 8 — Node 2 Company/Driver Onboarding Integration Investigation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat15  
**Day:** Day 8  
**Status:** INVESTIGATION — OPEN  

## 1. Investigation Trigger

During Ayush manual verification of the Chat14 Node 2 implementation, the following behavior was observed:

1. A new account was created successfully.
2. The account was shown the `Pending Verification` screen with status `PENDING`.
3. The Supabase migration was executed successfully.
4. The `freight_identities` table was inspected and found to contain the created identity.
5. The live schema uses `auth_id`, not `auth_user_id`.
6. The test identity was manually changed from `PENDING` to `VERIFIED`.
7. Signing in as the verified test account passed the verification gate.
8. The application then displayed `No Driver Profile` and stated that the account was not linked to a driver code.
9. The current signup flow did not present a Company-vs-Driver role selection or document/evidence submission step.

This investigation exists to determine whether these behaviors are consistent with the locked Node 1 / Node 2 product and identity model and to identify the correct integration boundary before any further implementation.

## 2. Evidence Observed

### 2.1 Database identity creation

The live `freight_identities` schema observed by Ayush contains:

```text
id
 auth_id
requested_role
verification_status
trusted_role
email
created_at
updated_at
```

The newly created test identity contained:

```text
requested_role      = DRIVER
verification_status = PENDING
trusted_role        = NULL
```

After manual verification:

```text
verification_status = VERIFIED
trusted_role        = NULL
```

The successful creation of the identity row demonstrates that the Auth User → Freight Identity creation path is operating for the tested signup.

### 2.2 Verification gate

The new account initially reached:

```text
Pending Verification
Current status: PENDING
```

After the identity was manually changed to `VERIFIED`, signing in as that same account passed the verification gate and allowed access to the authenticated application area.

This is evidence that the current verification gate is reading the authenticated account's identity state rather than merely relying on a client-selected status.

### 2.3 Post-verification application behavior

After the verified test account passed the gate, the application displayed:

```text
No Driver Profile
Your account is not linked to a driver code.
```

Inspection of the current source showed the authenticated home page still resolves a Driver by querying `drivers.auth_id` using the authenticated Supabase user ID. If no Driver row is found, the page returns the `No Driver Profile` state.

This establishes a legacy Driver-specific dependency after the new generic Freight Identity verification boundary.

### 2.4 Signup role behavior

The tested signup flow did not present a Company/Driver role selection to Ayush. The resulting Freight Identity had:

```text
requested_role = DRIVER
```

Therefore, the currently observed implementation does not expose the intended user-facing choice between Company and Driver during the tested signup path.

### 2.5 Evidence/document verification behavior

The tested signup flow did not present a document/evidence submission step for either role.

The Node 2 contract draft describes the proposed MVP verification model as:

```text
Driver  → Driving Licence evidence
Company → GST evidence/details
```

and describes a human-review flow in which evidence is reviewed before establishing a trusted role. The current tested UI did not demonstrate that workflow.

## 3. Relevant Contract Baseline

The Node 2 contract establishes the following intended model:

```text
Supabase Auth User
        ↓ exactly one
Generic Freight Identity
        ↓
requested_role = Company / Driver
verification_status = PENDING / VERIFIED / REJECTED
trusted_role = NULL until verification
```

The contract explicitly distinguishes requested role from trusted role and states that requested role is user intent, not authorization.

The proposed hackathon verification model is:

```text
User signup
    ↓
Select requested role
    ↓
Provide official evidence
    ↓
PENDING
    ↓
Ayush reviews evidence
    ↓
APPROVE / REJECT
    ↓
If approved → VERIFIED + trusted_role
```

The exact reviewer interface, audit trail, evidence validation checklist, and role-specific mapping were identified as items requiring further definition in the draft contract.

## 4. Root-Cause Assessment

### Finding A — Verified identity reaches legacy Driver page

**Status:** VERIFIED

The new Freight Identity verification gate and the existing authenticated home page currently represent two different identity boundaries.

The gate can establish that an authenticated user has a verified Freight Identity, but the downstream home page still assumes that the authenticated user must correspond to a row in the legacy `drivers` table.

Therefore:

```text
Verified Freight Identity
        ≠
Existing Driver profile
```

This is an integration mismatch, not evidence that the test user should be manually assigned a Driver profile.

### Finding B — Signup currently produces a Driver requested role in the tested path

**Status:** VERIFIED for the tested signup path

The tested account was created with `requested_role = DRIVER`, and no Company/Driver selection was presented during the observed signup flow.

The cause of the default Driver value and whether the implementation intentionally simplified signup for an intermediate checkpoint must be confirmed against the current implementation/report before changing it.

### Finding C — Document/evidence onboarding is not demonstrated by the current UI

**Status:** VERIFIED for the tested flow

The current manual test did not expose Driver licence or Company GST evidence submission, nor an end-to-end reviewer workflow.

The contract describes this as part of the proposed verification model, but the current implementation/report does not provide sufficient evidence that the complete workflow has been implemented.

## 5. Architectural Questions Requiring Resolution

Before implementation changes are authorized, resolve:

1. What is the authoritative post-verification application identity boundary for Company and Driver?
2. How should `freight_identities` map to role-specific Driver and Company application records, if role-specific records remain necessary?
3. Should the authenticated home/dashboard resolve application context from Freight Identity rather than directly assuming `drivers.auth_id`?
4. Where and how does a user select `requested_role` during signup?
5. What exact evidence is submitted for Driver and Company, and where is that evidence stored?
6. What exact server-authorized reviewer workflow establishes `trusted_role`?
7. What audit evidence is required for approval/rejection?
8. How should a VERIFIED Company user enter the Company application area?
9. How should a VERIFIED Driver user enter the Driver application area without resurrecting Driver Code as an authentication credential?
10. Which parts of the current Driver-specific UI are legacy behavior that must be replaced versus retained as role-specific business functionality?

## 6. Safety Boundary

Do not resolve this investigation by:

- manually creating a Driver profile for the test user merely to make the dashboard load;
- treating Driver Code as the authentication credential;
- assigning `trusted_role` from the client;
- adding a Company/Driver role based only on client-supplied authorization fields;
- bypassing the verification workflow;
- changing the database schema without first reconciling the implementation with the Node 2 contract.

## 7. Current Decision

**No implementation fix is authorized by this investigation yet.**

The correct next step is to inspect the current Node 2 implementation and relevant source/database mapping, reconcile it with the locked Node 1 / Node 2 contract, and then make an explicit architecture/implementation decision.

## 8. Current State

```text
Auth signup                          = OBSERVED WORKING
Freight Identity creation            = OBSERVED WORKING
Pending Verification gate            = OBSERVED WORKING
Manual PENDING → VERIFIED test       = OBSERVED WORKING
Verified user passes gate            = OBSERVED WORKING
Post-gate Driver dependency          = VERIFIED LEGACY INTEGRATION GAP
Company signup selection             = NOT DEMONSTRATED / OPEN
Driver signup selection              = NOT DEMONSTRATED / OPEN
Driver evidence submission           = NOT DEMONSTRATED / OPEN
Company GST evidence submission      = NOT DEMONSTRATED / OPEN
Reviewer workflow                    = NOT DEMONSTRATED / OPEN
Company/Driver application mapping   = OPEN
Node 2 final acceptance              = NOT COMPLETE
```

## 9. Next Investigation Action

Inspect the current source implementation and the Chat14 implementation report together with the Node 2 contract to determine exactly which onboarding and role-mapping behavior was intended for this implementation checkpoint, then produce a decision before issuing any further implementation prompt.
