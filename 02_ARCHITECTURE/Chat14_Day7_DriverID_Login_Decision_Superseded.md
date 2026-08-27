# Chat14 / Day 7 — Driver ID Login Decision Superseded

## Status

**APPROVED — ACTIVE DECISION**

This record explicitly supersedes the earlier Driver-ID-as-login architecture so future ChatGPT, Claude, and Antigravity sessions do not revive the older design.

## Superseded Decision

An earlier security investigation approved a random, server-generated Driver ID in the format `DRV-XXXX-XXXX` and described Driver ID + Password as the Driver login flow.

That historical record remains preserved as historical evidence. It is **not deleted or rewritten**.

## Current Active Decision

The Freight product will use the following identity architecture for the current Node 2 implementation:

```text
Email + Password
        ↓
Supabase Auth User
        ↓
Exactly one Freight Identity
        ↓
requested_role: Company OR Driver
        ↓
verification_status
        ↓
trusted_role (server-controlled)
```

### Authentication

- Authentication uses **Email + Password** through Supabase Auth.
- Driver ID / Driver Code is **not an authentication credential**.
- Driver ID / Driver Code is **not required to determine whether a user is a Driver or Company**.
- The authenticated Supabase user is the root authentication identity.

### Application identity

- `freight_identities` is the canonical application identity layer.
- One authenticated Supabase user maps to exactly one Freight Identity.
- The Freight Identity records the requested role and server-controlled verification/trust state.
- Company vs Driver is determined from the Freight Identity/role model, not from a Driver Code.

### Verification

- New identities begin in `PENDING` verification state.
- Protected Freight functionality requires the approved active/verification gate.
- An authorized verifier/admin reviews pending submissions and documents.
- Approval/rejection and `verification_status` / `trusted_role` changes are server-controlled.

## Driver ID / Driver Code Future Use

The earlier random Driver ID concept is **not required for the current Node 2 authentication architecture**.

If a Driver-facing identifier is needed later, it may be introduced as a separate Driver profile identifier. It must not be allowed to become a second competing authentication identity without a new explicit architecture decision.

Do not implement the historical `DRV010` sequential login experiment or the later random Driver-ID-as-login flow as part of current Node 2.

## Why This Decision Changed

The newer Freight Identity architecture provides a cleaner separation of concerns:

```text
Authentication identity
→ Supabase Auth

Application identity
→ Freight Identity

Role
→ Company / Driver

Trust / verification
→ verification_status + trusted_role

Optional Driver-facing identifier
→ separate profile concern
```

This avoids using a separate Driver Code to infer application role and avoids maintaining a Driver-ID → email → Supabase Auth authentication indirection.

## Scope Boundary

This superseding decision applies to the current Node 2 Authentication + Identity implementation and any future implementation prompt that claims to implement the current Node 2 contract.

It does not delete or invalidate the historical investigation that originally recommended random Driver IDs. That record remains part of the project's historical decision trail.

Any future proposal to use Driver ID as an authentication identifier must be treated as a **new architecture change requiring explicit project-owner approval**.
