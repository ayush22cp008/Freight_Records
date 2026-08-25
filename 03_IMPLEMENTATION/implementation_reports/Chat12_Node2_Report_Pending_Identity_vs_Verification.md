# Chat12 — Node 2 Investigation: Pending Identity vs Verification Timing

## 1. Preflight State
- **Source repository root:** `freight`
- **Branch:** `main`
- **Git status:** Up to date. Local-only changes exist in API routes and migration `004`.
- **Records repo state:** `Freight_Records` on `main`, up to date with recent architecture draft updates.

## 2. Records / Source / Configuration Inspected
- `02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md` (Updated)
- `02_ARCHITECTURE/Chat12_Node2_Signup_Onboarding_Consistency_Decision.md`
- `03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md`
- `03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Independent_Review_Claude.md`

## 3. Node 1 Invariant Compatibility
Invariant: `1 Auth User ↔ exactly 1 application identity ↔ exactly 1 role`
- **Model A (Auth → PENDING Specific Identity):** Compatible. The "PENDING Driver/Company" record serves as the single application identity. [VERIFIED]
- **Model B (Auth → Verification → Identity):** **Incompatible** temporarily. During verification, the Auth User has 0 application identities, breaking the strict 1:1 invariant. [VERIFIED]
- **Model C (Auth → Generic Pending Identity → Verification → Trusted Role):** **Highly Compatible.** The user gets exactly 1 generic application identity immediately upon Auth creation. [VERIFIED]

## 4. Role Establishment
- **Requested Role:** Supplied by the user during signup (e.g., via Auth metadata).
- **Trusted Role:** Granted by the system only after verification. Must be stored in a secured column.
- **Verification Status:** An enum (`PENDING`, `VERIFIED`, `REJECTED`) controlled strictly by the server.
- **Active State:** Depends on both Email Confirmation and `VERIFIED` status. [VERIFIED]

## 5. Trigger Compatibility
- **Model A:** The trigger can create a specific identity if `requested_role` is passed in metadata. However, creating `drivers` records for unverified users pollutes the table. [VERIFIED]
- **Model B:** **Incompatible.** We cannot use an `AFTER INSERT` trigger on `auth.users` to create the identity because we are waiting for verification. [VERIFIED]
- **Model C:** **Perfectly Compatible.** The trigger securely and atomically creates the `freight_identities` record immediately. [VERIFIED]

## 6. Unverified Identity Risk
- **Model A (High Risk):** PENDING users exist in the `drivers`/`companies` tables. Accidental RLS misconfigurations could expose them in marketplace queries or allow them to act prematurely. [VERIFIED]
- **Model C (Low Risk):** PENDING users exist only in a generic identity table. They structurally cannot act as a driver/company until verified and moved/linked to the specific table. [VERIFIED]

## 7. Verification Workflow Fit
Workflow: Upload → PENDING → OCR/Review → VERIFIED
- **Model A/C:** Both provide an immediate database record to attach uploaded documents and OCR states to. Model C is cleaner as it isolates this from business logic. [VERIFIED]

## 8. Security Boundary
- The user must never be able to set their `trusted_role` or `verification_status`.
- This state must be managed by the server (`service_role` webhook from OCR or admin panel) and protected by strict RLS. [VERIFIED]

## 9. Authentication and Authorization
- A PENDING user can log in and hold a valid JWT session.
- Protected API calls and business operations (e.g., accepting a trip) must enforce `verification_status = VERIFIED` via RLS or server-side checks. [VERIFIED]

## 10. Email Confirmation Interaction
- Identity creation happens immediately via trigger, independent of email confirmation.
- No security conflict exists, provided the final `VERIFIED` state requires both document approval AND email confirmation. [VERIFIED]

## 11. Data / Privacy Implications
- PENDING and REJECTED accounts hold sensitive documents.
- A background process is needed to purge documents/OCR data for REJECTED users or users stuck in PENDING for too long. [VERIFIED]

## 12. Acceptance Criteria Minimums
- **One Auth User cannot receive two identities:** Insert to identity table throws unique constraint violation.
- **One Auth User cannot become both Company and Driver:** Trigger/API strictly sets one trusted role.
- **PENDING users cannot perform protected operations:** RLS blocks `INSERT/UPDATE` on business tables if `status != VERIFIED`.
- **Verification cannot be self-approved:** RLS blocks `UPDATE` on `verification_status` for the user.
- **Rejected users cannot gain trusted role:** Status `REJECTED` permanently blocks role elevation.
- **Concurrent signup:** GoTrue blocks duplicate emails; DB unique constraints block duplicate identities.
- **Email confirmation bypass:** RLS/Server checks both email confirmed and document verified flags.

## 13. A/B/C Architecture Comparison

| Feature | Model A (Pending Specific) | Model B (Wait for Verification) | Model C (Generic Pending → Trusted) |
|---|---|---|---|
| **Node 1 Compatibility** | Compatible | Violates (0 identities temporarily) | Compatible |
| **Trigger Compatibility** | Compatible | **Incompatible** | Compatible |
| **Security / Risk** | Medium (leaks unverified users) | Low | Low (structural isolation) |
| **Abandoned Accounts** | Clutters business tables | Orphans Auth Users | Clutters generic auth table only |
| **Implementation** | Requires RLS on all queries | Requires custom compensation | Cleanest boundary |

## 14. Remaining UNKNOWNs
- None regarding the data model. (Specific OCR/AI tooling is outside the scope of this architecture decision).

## 15. Decision Inputs for Node 2 Contract
1. **Model B is technically disqualified** as it breaks the Node 1 invariant and prevents the use of atomic Auth triggers.
2. **Model C provides the strongest security boundaries** by keeping unverified users out of the core business tables (`drivers`, `companies`) until they are fully verified.
3. Auth metadata is the most reliable way to pass the user's `requested_role` to the database trigger for Model C.
