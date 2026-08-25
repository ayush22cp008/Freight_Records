# Chat12 — Node 2 Investigation: Pending Identity vs Verification Timing

## 1. Preflight State
- **Source repository root:** `freight`
- **Branch:** `main`
- **Git status:** Up to date. Local-only changes exist in API routes and migration `004`.
- **Records repo state:** `Freight_Records` on `main`.

## 2. Records / Source / Configuration Inspected
- `02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`
- `03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md`
- Supabase Auth User metadata capabilities for trigger propagation.

## 3. Node 1 Invariant Compatibility
Invariant: `1 Auth User ↔ exactly 1 application identity ↔ exactly 1 role`
- **Model A (Auth → PENDING Specific Identity):** Compatible, but requires treating a "PENDING Driver" as the 1 application identity.
- **Model B (Auth → Verification → Identity):** **Incompatible** temporarily. During verification, the Auth User has 0 application identities, breaking the strict 1:1 invariant and leaving the system in a partial state.
- **Model C (Auth → Generic Pending Identity → Verification → Trusted Role):** **Highly Compatible.** The user gets exactly 1 generic application identity (e.g., `freight_identities`) immediately upon Auth creation, fulfilling the 1:1 invariant. The role becomes trusted only after verification.

## 4. Role Establishment
- **Requested Role:** Should be passed via Supabase Auth `user_metadata` during signup.
- **Trusted Role:** Must be a secure column (e.g., in `freight_identities`) updated only by `service_role` (admin/webhook) after verification.
- **Verification Status:** Should be an enum (`PENDING`, `VERIFIED`, `REJECTED`) strictly controlled by the server.

## 5. Trigger Compatibility
- **Model A:** The trigger can create a specific role identity (Driver) if the requested role is passed in `user_metadata`. However, it creates Driver records that aren't real drivers yet.
- **Model B:** **Incompatible with Auth Trigger.** If we wait until verification to create the identity, we cannot use an `AFTER INSERT` trigger on `auth.users` to enforce consistency. This reintroduces the severe consistency/orphan issues found in the previous investigation.
- **Model C:** **Perfectly Compatible.** The Auth trigger immediately creates the generic `freight_identities` record with `status = PENDING`. Consistency is guaranteed transactionally.

## 6. Unverified Identity Risk
- **Model A:** High risk. Creating a `drivers` record with a `driver_code` for a PENDING user exposes them to RLS leaks, accidental matching in marketplace queries, or Driver code enumeration. Every query must explicitly filter `WHERE status = 'VERIFIED'`.
- **Model C:** Low risk. The generic identity table holds the pending state. The actual `drivers` table (with `driver_code`) is only populated once VERIFIED, making it structurally impossible for a pending user to act as a driver.

## 7. Verification Workflow Fit
The workflow (Upload → OCR → Review → Decision) requires an entity to attach the uploaded documents and OCR results to. 
- **Model C** provides this perfectly: the generic identity record serves as the anchor for the verification workflow before the user earns their trusted role.

## 8. Security Boundary
- **Client Control:** The client may only supply the *requested* role during signup.
- **Server Control:** The `trusted_role` and `verification_status` must be protected by RLS (read-only for the user) and only modifiable by `service_role` (the OCR webhook or admin reviewer).

## 9. Authentication and Authorization
- A PENDING user can log in and receive a session.
- RLS policies on business tables (e.g., `trips`, `events`) must join against the trusted role/status, or rely on the fact that the user doesn't exist in the `drivers` table yet (Model C).
- Any client-supplied role in JWT or metadata must be treated as untrusted for authorization.

## 10. Email Confirmation Interaction
- Identity creation (via trigger) happens immediately, independent of email confirmation.
- The verification workflow can either mandate email confirmation before allowing document upload, or proceed in parallel. No security conflict exists as long as "VERIFIED" requires both email and document checks.

## 11. Data / Privacy Implications
- PENDING and REJECTED accounts will accumulate sensitive uploaded documents (e.g., licenses). 
- A data retention policy (and background cleanup job) will be required to purge documents and OCR data for accounts that are REJECTED or abandoned in PENDING state for >30 days.

## 12. Acceptance Criteria
1. Auth User creation always results in exactly one application identity.
2. A PENDING user cannot insert or view `events` or `trips`.
3. A user cannot modify their own `verification_status`.
4. A REJECTED user cannot access Driver/Company protected routes.
5. Retrying signup for an existing email fails safely at the Auth layer.

## 13. A/B/C Architecture Comparison

| Feature | Model A (Pending Specific) | Model B (Wait for Verification) | Model C (Generic Pending → Trusted) |
|---|---|---|---|
| **Node 1 Invariant** | Compatible | Violates (0 identities temporarily) | Compatible |
| **Auth Trigger Sync** | Compatible | **Incompatible** | Compatible |
| **Security Risk** | Medium (leaks unverified drivers) | Low | Low (structural isolation) |
| **Implementation** | Complex RLS on every table | Requires custom orphan handling | Cleanest boundary |
| **Major Risks** | Unverified users act as drivers if RLS missed | Orphaned Auth Users; consistency lost | Requires a new generic identity table |

## 14. Remaining UNKNOWNs
- Specific OCR provider webhooks and latency behavior are unknown, but this does not affect the core identity data model.

## 15. Decision Inputs for Node 2 Contract
1. **Model B is technically disqualified** because it prevents the use of the Auth Trigger for consistency, reintroducing the orphan user problem.
2. **Model C is the safest and most compliant** because it preserves the 1:1 invariant transactionally, isolates unverified users from business tables, and provides a clear anchor for the verification workflow.
3. The requested role must be passed in `user_metadata` during `signUp` so the trigger can record it securely.
