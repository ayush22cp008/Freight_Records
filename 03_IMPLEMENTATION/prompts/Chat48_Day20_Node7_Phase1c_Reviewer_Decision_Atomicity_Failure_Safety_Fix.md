# Chat48 / Day20 / Node7 / Phase1c
# Reviewer Final Decision — Atomicity & Failure-Safety Fix

**Status:** APPROVED FOR IMPLEMENTATION  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day20  
**Executor:** Antigravity  
**Scope:** Make Reviewer Approve/Reject final decision atomic and failure-safe

## 1. Objective

Fix the confirmed Reviewer decision atomicity gap identified in:

```text
05_DEBUGGING/investigations/Chat48_Day20_Node7_Reviewer_Decision_Atomicity_Failure_Safety_Investigation_Report.md
```

The current Reviewer decision endpoint performs multiple independent database operations. A failure between operations can leave partial or contradictory state.

The required outcome is:

```text
Final decision succeeds
→ all required related changes commit together

OR

Final decision fails
→ changes roll back as one unit
→ applicant remains PENDING
→ Reviewer can safely retry
```

## 2. Current Affected Endpoint

```text
src/app/api/admin/review/route.ts
```

Current operations include evidence mutation, identity mutation, reviewer decision insertion, and approval-time Driver/Company record creation.

## 3. Required Architecture

Use a PostgreSQL transaction boundary for the complete final decision operation.

The investigation recommends moving the decision mutation sequence into a PostgreSQL stored procedure called through Supabase RPC.

Inspect the existing migration/database conventions before implementation and choose the smallest architecture-consistent RPC design.

The transaction must cover the complete logical final decision, not merely one individual update.

## 4. Required Transaction Semantics

### Reject

The transaction must atomically handle:

```text
Validate target identity is PENDING
        ↓
Identify the current PENDING evidence row
        ↓
Mark the intended current evidence as REJECTED
        ↓
Mark identity as REJECTED
        ↓
Insert reviewer_decisions row with rejection reason
        ↓
Commit
```

If any required step fails:

```text
ROLLBACK
↓
Identity remains PENDING
↓
Evidence remains PENDING
↓
No misleading completed decision is committed
```

### Approve

The transaction must atomically handle:

```text
Validate target identity is PENDING
        ↓
Identify the current PENDING evidence row
        ↓
Mark the intended current evidence as APPROVED
        ↓
Mark identity as VERIFIED
        ↓
Set trusted_role appropriately
        ↓
Insert reviewer_decisions row
        ↓
Create the required Driver OR Company record
        ↓
Commit
```

If any required step fails:

```text
ROLLBACK
↓
Identity remains PENDING
↓
Evidence remains PENDING
↓
No misleading completed decision is committed
↓
No partial Driver/Company record remains
```

## 5. Critical Current-Evidence Requirement

Do not update every pending evidence row indiscriminately.

Use the deterministic current evidence selection already established in the project:

```text
same auth_id
+ status = PENDING
+ newest created_at
+ limit 1
```

The decision and evidence mutation must be tied to the intended current evidence record.

Historical `REJECTED` or `APPROVED` evidence must remain preserved.

## 6. Concurrency / State Protection

The implementation must safely handle a Reviewer attempting to decide an applicant that is no longer PENDING.

The transaction/RPC must validate current state inside the transactional operation and reject stale/concurrent decisions without producing a partial outcome.

Do not invent a new persistent `UNDER_REVIEW` state.

## 7. Route Responsibility

Keep the API route responsible for:

- authenticating the caller;
- verifying Reviewer authorization;
- validating request shape;
- calling the transactional database operation;
- translating RPC success/failure into the existing API response contract.

Do not duplicate the multi-step mutation logic in the route after introducing the RPC.

## 8. Frontend Preservation

Do not redesign `ApplicantVerificationClient.tsx`.

Preserve the existing frontend behavior:

```text
API success
→ show correct result state

API failure
→ show clear error
→ applicant remains Pending
→ retry remains possible
```

The backend transaction must make that statement true, rather than relying only on frontend messaging.

## 9. Security / Authorization

Reviewer authorization must remain enforced server-side.

Do not expose the RPC as an unrestricted public operation.

Ensure the route's existing Reviewer authorization remains in place and the RPC cannot be abused by a non-Reviewer caller through direct invocation.

Inspect current Supabase/RLS/function conventions before finalizing the function security model.

Do not weaken RLS or authentication to make the RPC work.

## 10. Allowed Source / Database Scope

Expected files may include:

```text
src/app/api/admin/review/route.ts
src/db/migrations/<new migration for transactional Reviewer decision RPC>
```

A new migration is expected if the repository's database changes are managed through migrations.

Do not modify unrelated Reviewer UI, Driver, Company, onboarding, navigation, or evidence-upload code.

If additional files are required, report exactly why before expanding scope.

## 11. Preflight — Mandatory

Before editing:

- project root / current working directory;
- source repository;
- branch;
- current commit SHA;
- working-tree status;
- current `src/app/api/admin/review/route.ts` behavior;
- existing migration naming/order conventions;
- available database/RPC conventions.

Stop if repository boundaries do not match the expected source repository.

## 12. Validation Requirements

After implementation:

1. Run the repository's type-check/static validation.
2. Run database/migration validation appropriate for the project.
3. Run `npm run build`.
4. Inspect `git diff`.
5. Confirm no unrelated changes.
6. Verify both Approve and Reject normal paths.
7. Verify controlled failure behavior where safely possible without corrupting production data.
8. Document exact commands and results.

Do not claim transaction safety solely because an RPC exists. Show the implemented transactional boundary and validation evidence.

## 13. Manual Verification Handoff for Ayush

Verify at minimum:

```text
A. Normal Reject
   → evidence becomes rejected
   → identity becomes rejected
   → rejection history exists
   → rejection reason preserved

B. Normal Approve
   → evidence becomes approved
   → identity becomes verified
   → reviewer history exists
   → Driver/Company record exists

C. Failure safety
   → induced/controlled intermediate failure, where safely testable
   → no partial final state
   → applicant remains Pending

D. Retry
   → after a failed attempt, a valid retry can still complete the decision

E. Regression
   → Application Rejected UI still works
   → re-upload recovery still works
   → Reviewer Queue still works
   → Verification History still works
   → Driver/Company flows remain unchanged
```

Manual browser verification remains `PENDING` until Ayush explicitly confirms it.

## 14. Implementation Report

After implementation and validation, create:

```text
03_IMPLEMENTATION/implementation_reports/Chat48_Day20_Node7_Phase1c_Reviewer_Decision_Atomicity_Failure_Safety_Fix_Implementation_Report.md
```

The report must include:

- preflight state;
- exact route/migration files changed;
- RPC/function name and transactional scope;
- exact Approve and Reject mutation sequence;
- authorization/security handling;
- type-check result;
- migration/database validation result;
- `npm run build` result;
- final diff/scope verification;
- failure-path evidence or a precise statement explaining what could and could not be safely tested;
- manual verification status.

## 15. Push Boundary

Do not push automatically.

After implementation, validation, and report creation, wait for the standard project push approval and Ayush's explicit permission.

## 16. Completion Definition

```text
Atomic Reviewer decision implemented
        ↓
Approve is all-or-nothing
        ↓
Reject is all-or-nothing
        ↓
Current evidence only is mutated
        ↓
Reviewer authorization preserved
        ↓
Type-check passes
        ↓
Database/migration validation passes
        ↓
Production build passes
        ↓
Diff scope confirmed
        ↓
Implementation report saved
        ↓
Ayush manual verification pending
```

This is a backend correctness hardening change required by the Reviewer Blueprint. Do not redesign the Reviewer UX or introduce unrelated product behavior.