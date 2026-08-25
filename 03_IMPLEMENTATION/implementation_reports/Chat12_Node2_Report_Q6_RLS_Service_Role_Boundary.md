# Chat12 Node 2 — Q6 Investigation: RLS / Service-Role Boundary

## 1. Executive Conclusion

The recommended policy for Q6 is a **Strict RLS + Privileged Server Boundary Pattern**.

1. **Direct client database access:** Browser Supabase clients use authenticated sessions and are always subject to RLS.
2. **User-facing Next.js routes:** Prefer a user-scoped Supabase client that propagates the user's session and therefore evaluates RLS. A route must not use `service_role` merely for convenience.
3. **Privileged server operations:** `service_role` is permitted only for narrowly defined trusted server-side operations that genuinely require RLS bypass, such as approved verification/admin workflows, controlled webhooks, or background jobs.
4. **Authorization:** A `service_role` client is a database capability, not an authorization decision. Every privileged endpoint must have its own trusted authentication/authorization boundary before using it.

The MVP goal is to make RLS the normal data-isolation boundary and make privileged access exceptional, explicit, auditable, and server-only.

## 2. Current Architecture / Evidence

Prior repository evidence indicates:

- Earlier RLS experiments confirmed ownership policies based on `auth.uid()` can isolate authenticated users' data.
- Earlier API code used a service-role-backed server client for some identity creation operations, creating a potential RLS bypass path.
- `src/lib/supabase/server.ts` provides user-scoped server access, while `src/lib/supabase-server.ts` exposes a service-role-backed client and therefore requires strict usage controls.

These observations are evidence about the current codebase, not permission to implement changes during this investigation.

## 3. Threat Model

Q6 must protect against:

- IDOR / cross-user reads and writes;
- privilege escalation through client-controlled identifiers;
- users modifying verification/trusted-role state;
- accidental use of `service_role` in user-facing paths;
- leaked service-role credentials;
- missing or overly broad RLS policies;
- tables created without appropriate RLS protection;
- privileged endpoints that authenticate a caller but fail to authorize the requested administrative action.

## 4. RLS Responsibility

RLS is the **database row-access boundary**. It should enforce ownership/relationship constraints that can safely be expressed at the database layer.

For every protected table, the project must explicitly define:

```text
RLS enabled?
SELECT policy?
INSERT policy?
UPDATE policy?
DELETE policy?
Relationship/ownership rule?
FORCE ROW LEVEL SECURITY status?
```

RLS does not automatically guarantee security merely because it is enabled. The actual policy expressions must be correct and restrictive.

**Important default-deny rule:** when RLS is enabled on a table, access for ordinary non-owner roles is denied unless an applicable policy grants it. Therefore, an RLS-enabled table with no policy is a fail-closed state for those roles; the project must still verify that every required operation has an intentional policy rather than treating `ENABLE ROW LEVEL SECURITY` alone as sufficient.

The recommendation is that protected Freight business tables have RLS enabled unless a documented architecture exception is approved.

Typical identity binding should use `auth.uid()` through the appropriate ownership/relationship columns, but policies must be designed per table rather than assuming one universal `auth_id` column exists everywhere.

**Table-owner verification:** PostgreSQL RLS does not normally apply to the table owner unless `FORCE ROW LEVEL SECURITY` is used. The implementation audit must therefore verify the owner role and `FORCE ROW LEVEL SECURITY` status for each protected table, or otherwise prove that the table-owner role cannot be reached through an untrusted application path.

## 5. Service-Role Responsibility

Supabase `service_role` is a highly privileged server-side credential that bypasses RLS. It must therefore be treated as a **privileged database capability**, not as an authorization mechanism.

### Allowed pattern

```text
Trusted server request
      ↓
Authenticate caller / verify trusted origin where applicable
      ↓
Authorize exact privileged operation
      ↓
Use service_role only for the required database action
      ↓
Audit privileged mutation where required
```

### Forbidden pattern

```text
User-controlled request
      ↓
service_role
      ↓
Trust req.body IDs
      ↓
Database mutation
```

The exact privileged operation must be explicitly defined and narrowly scoped.

## 6. Access Matrix

| Operation | Browser / User Client | User-scoped Next.js route | Privileged Server / service_role |
|---|---|---|---|
| Read own permitted data | ALLOWED via RLS | ALLOWED via RLS | **DISALLOWED except via approved exception** |
| Read another user's protected data | BLOCKED by correct RLS | BLOCKED by correct RLS | Only if an explicitly authorized privileged operation requires it |
| Modify own ordinary permitted data | ALLOWED only where RLS policy permits | ALLOWED only where RLS policy permits | **DISALLOWED except via approved exception** |
| Modify verification/trusted-role state | BLOCKED by policy | BLOCKED by policy | ALLOWED only through an explicitly authorized privileged workflow |
| Administrative verification | NOT ALLOWED | NOT ALLOWED unless route has explicit admin authorization | ALLOWED after trusted authorization |
| Background cleanup | NOT APPLICABLE | NOT APPLICABLE | ALLOWED in controlled server job |

The matrix deliberately does not treat `service_role` as a general-purpose alternative. Privileged use is **DISALLOWED except via an approved exception** with a documented business purpose and authorization boundary.

## 7. Node 1 Authorization Interaction

RLS and Node 1 authorization are complementary.

```text
RLS
→ database-level row isolation

Node 1 authorization
→ operation-level business authorization
```

Example:

```text
RLS
→ Driver A cannot read Driver B's protected row.

Node 1
→ Driver A may read their own row, but may still be forbidden
  from performing a particular operation because of trip state/role/context.
```

A request passing RLS does not automatically satisfy Node 1 authorization.

A privileged service-role operation must perform its required Node 1/admin authorization checks before the database action; bypassing RLS does not bypass the business contract.

## 8. Identity / Verification Interaction

Users must not directly change security-sensitive identity state such as:

```text
verification_status
trusted_role
administrative approval state
```

RLS policies should prevent ordinary authenticated users from mutating those fields.

A privileged verification/admin operation may use `service_role`, but only after the trusted server endpoint establishes that the caller is authorized to perform that exact operation.

Auth-trigger-based identity creation may use database-level privileges as appropriate, but this is a separate controlled mechanism and should not be used as evidence that arbitrary application routes may use `service_role`.

## 9. IDOR / Cross-User Attack Analysis

The main failure mode is:

```text
Attacker changes driver_id / trip_id
        ↓
User-facing route uses service_role
        ↓
RLS is bypassed
        ↓
Attacker may access another user's row
```

Preferred protection:

```text
User request
   ↓
user-scoped Supabase client
   ↓
RLS evaluates auth.uid()
   ↓
Cross-user row rejected/filtered
```

If a privileged route legitimately requires service-role access, it must not trust arbitrary request IDs. It must explicitly authorize the requested resource/action before using the privileged database capability.

## 10. Recommended MVP Policy

1. **RLS by default:** Enable RLS on protected Freight business tables and define explicit policies for each operation.
2. **User-scoped access by default:** Browser and user-facing Next.js routes use authenticated user-scoped Supabase clients whenever the operation can be performed under RLS.
3. **Privileged access by exception:** `service_role` is reserved for explicitly documented server-side operations that genuinely require RLS bypass.
4. **Privileged endpoints require authorization:** A service-role route must establish the caller's trusted identity and exact authorization before using the credential.
5. **Server-only secret:** `service_role` must never be exposed to browser/client code, public environment variables, logs, or client responses.
6. **No client-controlled trust:** A privileged route must not equate possession of a user-supplied ID with permission to act on that object.
7. **Mandatory auditability for security-sensitive privileged mutations:** Any privileged operation that mutates `verification_status`, `trusted_role`, or other administrative approval state **MUST** be logged with actor identity, target resource, and timestamp. Other privileged operations should be logged where appropriate.
8. **No architecture shortcut:** Do not use `service_role` merely to avoid fixing an RLS policy.
9. **Service-role compromise response:** If exposure of the service-role credential is suspected, rotate/revoke the credential immediately through the Supabase project controls, then audit recent privileged-operation logs for the exposure window and investigate affected resources.

## 11. Rejected Alternatives

### Application-layer-only security

Rejected. Disabling RLS and relying entirely on Next.js checks creates a dangerous single point of failure for IDOR and cross-user access.

### Service-role for all API routes

Rejected. This bypasses the database row-security boundary and makes every application authorization mistake potentially catastrophic.

### One universal RLS policy

Rejected. Different Freight tables and operations have different ownership/relationship requirements. Policies must be explicit per table and operation.

## 12. Acceptance-Test Matrix

At minimum, the eventual implementation must prove:

1. User A cannot `SELECT` User B's protected rows through a normal authenticated client.
2. User A cannot `UPDATE` User B's protected rows through a normal authenticated client.
3. User A cannot modify their own `verification_status` or `trusted_role` through ordinary user access.
4. A correctly authorized privileged verification workflow can update security-sensitive state.
5. An unauthorized caller cannot invoke a privileged service-role operation successfully.
6. A privileged endpoint rejects client-controlled IDs when the caller lacks authorization for the referenced resource/action.
7. `service_role` is absent from browser/client bundles and client-visible responses.
8. No public/user-facing route uses `service_role` merely for ordinary CRUD that RLS can safely enforce.
9. Protected tables have RLS enabled and expected SELECT/INSERT/UPDATE/DELETE policies.
10a. A row that passes RLS (for example, User A reading User A's permitted row) can still be rejected by Node 1 authorization when the business operation is forbidden by role, trip state, or other locked business rules.
10b. A row blocked by RLS is never reachable even if Node 1 authorization would otherwise allow the business operation.
11. A protected table with RLS enabled but no applicable policy denies access for ordinary non-owner roles; this default-deny behavior is verified explicitly.
12. Protected table owner roles and `FORCE ROW LEVEL SECURITY` status are verified so that an untrusted application path cannot bypass RLS through table ownership.
13. Privileged background/webhook operations are constrained to their explicitly approved scope.
14. Privileged mutations of `verification_status`, `trusted_role`, or administrative approval state produce an audit record containing actor identity, target resource, and timestamp.
15. If service-role credential exposure is simulated/confirmed, the documented rotation and post-exposure audit procedure is followed.

## 13. Remaining Unknowns / Verification Items

Q6 policy is decision-ready, but these implementation-time facts must be verified before coding/acceptance:

1. Exact current Freight table list and ownership relationships.
2. Exact RLS policy coverage for each protected table and CRUD operation.
3. Exact list of privileged workflows that genuinely require RLS bypass.
4. Exact authentication/authorization mechanism for each privileged endpoint or webhook.
5. Actual source locations where `service_role` is currently imported/used.
6. Whether any existing trigger/function already performs an equivalent privileged identity operation.
7. `FORCE ROW LEVEL SECURITY` status and table-owner role for each protected table.

These are verification items, not permission to add new privileged paths.

## 14. Proposed Q6 Decision

```text
Q6 = Strict RLS + Privileged Server Boundary

Normal user operations
→ authenticated user-scoped client
→ RLS

Privileged operations that genuinely require RLS bypass
→ trusted server-only path
→ explicit authentication + authorization
→ narrowly scoped service_role use
→ mandatory audit logging for security-sensitive privileged mutations

service_role
≠ authorization
≠ normal user database client
≠ browser credential
```

## 15. Final Status

```text
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🔒 LOCKED
Q6 = 🟡 INVESTIGATED / READY FOR FINAL INDEPENDENT REVIEW
Implementation = ⏸️ PAUSED
```

**No code changes or database-policy changes are authorized by this investigation.**

Next:

```text
Q6 corrected report
        ↓
Independent Claude final review
        ↓
Ayush approval
        ↓
Q6 LOCK
        ↓
Move to Q7
```