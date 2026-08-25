# Chat12 Node 2 — Q6 Lock Record

## Q6 — RLS / Service-Role Boundary

**Status: 🔒 LOCKED**

### Locked Decision

Freight adopts the **Strict RLS + Privileged Server Boundary Pattern** for the MVP.

```text
Normal user operations
→ authenticated user-scoped client
→ RLS

Privileged operations that genuinely require RLS bypass
→ trusted server-only path
→ explicit authentication + authorization
→ narrowly scoped service_role use
→ mandatory audit logging for security-sensitive privileged mutations
→ approved service_role import allowlist enforced by automated lint/CI
```

### Core Rules

1. RLS is the normal database row-isolation boundary.
2. Protected tables must have intentional RLS policies.
3. Default-deny behavior must be preserved for protected data.
4. `FORCE ROW LEVEL SECURITY` / table-owner bypass must be explicitly verified for protected tables.
5. `service_role` is server-only and is a privileged database capability, not authorization.
6. `service_role` is disallowed for ordinary user CRUD except through an explicitly approved exception.
7. Privileged operations require explicit authentication and authorization before using `service_role`.
8. Security-sensitive privileged mutations of `verification_status`, `trusted_role`, or administrative approval state require mandatory audit records containing actor identity, target resource, and timestamp.
9. Suspected `service_role` compromise requires immediate rotation/restriction and incident auditing.
10. `SECURITY DEFINER` functions/triggers must be explicitly audited for execution role, `search_path`, privileges, callable surface, and function-body writes before implementation acceptance.
11. Future `service_role` imports must be restricted to an approved server-side allowlist enforced by lint/CI.
12. RLS and Node 1 authorization remain separate security layers; passing one does not bypass the other.

### Independent Review Evidence

Grok independently reviewed the Q6 report and returned:

```text
Verdict: APPROVE
Remaining corrections: None
Final recommendation: Lock Q6 as Strict RLS + Privileged Server Boundary Pattern.
```

Claude's repeated review output was inconclusive because it continued reporting an older/stale version that did not match the corrected repository record. Claude is therefore recorded as **temporarily unavailable/inconclusive**, not as an approval.

### Scope of Lock

This lock is an **architecture/policy lock only**.

```text
Q6 policy = 🔒 LOCKED
Implementation = ⏸️ PAUSED
```

The remaining table-by-table RLS audit, service-role import audit, `FORCE ROW LEVEL SECURITY` verification, trigger/function audit, and acceptance testing are implementation-time verification requirements. They do not reopen Q6 unless new evidence creates a genuine architectural conflict.

### Node 2 Status

```text
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🔒 LOCKED
Q6 = 🔒 LOCKED
```

### Next

Proceed to the next unresolved Node 2 decision/question under the same project-control workflow.

**No implementation is authorized by this lock.**
