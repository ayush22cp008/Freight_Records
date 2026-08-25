# Chat12 — Node 2 Signup / Onboarding Consistency Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Status:** PROPOSED DECISION — NOT YET LOCKED / REQUIRES INDEPENDENT REVIEW + AYUSH APPROVAL

## 1. Problem

Current signup creates the Supabase Auth User and application Driver identity through separate operations. This creates a durable partial-failure risk where an Auth User can exist without its required application identity.

## 2. Evidence basis

The Chat12 investigations established:

- Current Auth User creation and Driver identity creation are separate operations.
- The current flow is not atomic.
- A Driver identity creation failure can leave an orphaned Auth User.
- Retrying the current signup route does not repair that state because Auth signup rejects the already-registered email.
- `drivers.auth_id` is unique.
- The current server has a server-only service-role capability for Auth administration.
- Compensation is unsafe as the primary consistency mechanism for unknown-outcome timeouts because the current `ON DELETE SET NULL` relationship can leave an orphaned Driver record.
- A PostgreSQL trigger on `auth.users` is technically available and can execute within the Auth User insertion transaction.
- If the trigger-created application identity fails, the Auth User creation transaction can roll back.

Source records:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Signup_Onboarding_Consistency.md`

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md`

## 3. Proposed architecture decision

Use **PostgreSQL-trigger-based atomic application-identity creation** as the preferred signup/onboarding consistency mechanism.

Conceptually:

```text
Supabase Auth User creation
        ↓
PostgreSQL transaction
        ↓
auth.users INSERT trigger
        ↓
Application Identity creation
        ↓
Both succeed → durable consistent state

OR

Identity creation fails
        ↓
Transaction rolls back
        ↓
No durable Auth User without identity
```

This decision is about **identity consistency**, not authorization.

## 4. Email confirmation and authorization gates

Application identity existence must not be treated as equivalent to an active or authorized account.

The contract should distinguish:

```text
Identity exists
    ≠
Email confirmed
    ≠
Authorized
    ≠
Active/usable account
```

The proposed model is:

1. Auth User + application identity are created consistently/atomically.
2. Email confirmation is a separate account-state gate.
3. Application identity/role validity is a separate authorization gate.
4. Protected application use requires the final required gates to be satisfied.

The exact email-confirmation product policy remains an open contract decision until explicitly reviewed and approved.

## 5. Why compensation is not selected as primary mechanism

Server-side compensation can delete an Auth User after an explicit Driver insert failure, but it is unsafe as the primary consistency guarantee when the Driver request has an unknown outcome.

Example:

```text
Driver insert actually succeeds
        ↓
Response times out
        ↓
Server assumes failure
        ↓
Auth User deleted
        ↓
ON DELETE SET NULL
        ↓
Driver record remains without auth_id
```

Therefore compensation is not the preferred primary consistency boundary.

## 6. Scope boundary

This decision does NOT yet authorize:

- trigger implementation;
- migration changes;
- signup route changes;
- email-confirmation configuration changes;
- role model implementation;
- authentication implementation generally.

Those remain blocked until the Node 2 contract is independently reviewed and locked.

## 7. Required follow-up before lock

Before this decision becomes final:

- independently review trigger-based identity creation;
- explicitly resolve email-confirmation timing/policy;
- verify compatibility with the final one-user/one-identity/one-role model;
- define acceptance tests for atomic identity consistency;
- obtain Ayush approval;
- update the Node 2 contract to LOCKED status only after those gates pass.

## 8. Status

```text
Evidence                    = VERIFIED
Preferred architecture       = PROPOSED
Independent review           = PENDING
Ayush approval               = PENDING
Node 2 contract lock         = NOT YET
Implementation authorization = NOT GRANTED
```
