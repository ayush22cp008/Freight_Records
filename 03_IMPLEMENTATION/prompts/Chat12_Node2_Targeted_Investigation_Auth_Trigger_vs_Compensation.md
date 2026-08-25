# Chat12 — Node 2 Targeted Investigation: Auth Trigger vs Compensation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Type:** Targeted investigation  
**Status:** Investigation only — NO IMPLEMENTATION AUTHORIZED

## Objective

The previous Chat12 investigation established that current signup is not atomic and can leave an orphaned Auth User when application identity creation fails.

This investigation must collect enough evidence to make the Node 2 architecture decision. Do not implement, prototype, modify schema, change configuration, or apply a fix.

## Existing evidence to use

Read and validate:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Signup_Onboarding_Consistency.md`

Also inspect the current Node 2 contract draft and relevant earlier investigation records.

## Questions to answer

### 1. Auth `users` trigger feasibility

The previous report proposed a Postgres trigger on `auth.users` as a possible alternative to the current two-request flow.

Investigate this claim rigorously.

Determine from the actual current Supabase/project architecture and available source/configuration evidence:

- Whether a trigger on `auth.users` is technically available in this project.
- Whether such a trigger can safely create the required application identity.
- Whether the trigger execution is part of the Auth User creation transaction in a way that can enforce the required consistency invariant.
- What happens if the trigger fails.
- What happens if application identity creation inside the trigger fails.
- Whether the trigger can safely use the required identity/role information.
- Whether it would create the application identity before email confirmation.
- Whether it introduces security, recursion, privilege, operational, migration, or observability concerns.
- Whether this approach actually satisfies the Node 2 product/identity contract rather than merely moving the failure boundary.

Classify each conclusion as VERIFIED / INFERRED / UNKNOWN.

Do not create or modify the trigger.

### 2. Compensation feasibility

Investigate the proposed service-role compensation path.

Determine:

- Whether the current server boundary can call Supabase Auth admin deletion for the newly-created Auth User.
- What exact permissions/client are involved.
- Whether compensation is safe immediately after application identity creation fails.
- What happens if the application identity insert has an unknown outcome.
- What happens if compensation itself fails.
- Whether compensation can race with another signup/login/recovery operation.
- Whether compensation could accidentally delete an Auth User that should be preserved.
- What evidence exists for distinguishing safe compensation from unsafe compensation.

Do not implement compensation.

### 3. Unknown-outcome / timeout scenario

Investigate this exact case:

Auth User created → application identity request sent → response times out → actual database outcome is unknown.

Determine how the current system can establish whether the application identity already exists before retrying or compensating.

Inspect:

- `drivers.auth_id` uniqueness;
- relevant indexes/constraints;
- current lookup capabilities;
- current API behavior;
- race conditions;
- duplicate prevention.

Determine whether an idempotent retry/reconciliation operation is technically possible with current primitives.

Do not implement it.

### 4. Concurrency / race conditions

Investigate these cases separately:

A. Two signup requests for the same email at the same time.

B. Original signup is still running while a retry/recovery operation begins.

C. Auth User exists and two identity-creation attempts happen concurrently.

D. Identity creation succeeds but the original request reports failure.

E. Compensation begins while another operation is using the Auth User.

Determine what Supabase Auth and current database constraints guarantee, and what remains unprotected at application level.

### 5. Retry and recovery feasibility

Determine what a safe recovery strategy would require from the current architecture, without designing the final solution yet.

Specifically establish evidence for:

- retryable vs non-retryable failure classes;
- bounded retry feasibility;
- idempotent identity creation feasibility;
- compensation feasibility;
- compensation-failure recovery;
- manual recovery requirements;
- whether a reconciliation mechanism would require new infrastructure.

Do not choose the final architecture.

### 6. Email-confirmation dependency

The previous report established that the current code creates the Driver identity without checking confirmation state, but Dashboard configuration is UNKNOWN.

Investigate what can be established from the current project/source/configuration evidence about:

- email confirmation requirement;
- `signUp()` behavior when confirmation is required;
- returned Auth User/session state;
- whether identity creation should logically depend on confirmation under the current product model;
- whether trigger-based identity creation would change this behavior.

Do not make the final product policy decision.

## Required comparison

At the end, provide an evidence-based comparison of these architecture families:

1. Auth-trigger-based identity creation.
2. Server-side compensation after sequential creation.
3. Sequential creation with safe idempotent retry/recovery.
4. Combination of the above, only if evidence supports it.

For each option report:

- What is VERIFIED to be possible.
- What is UNKNOWN.
- Failure behavior.
- Unknown-outcome behavior.
- Concurrency behavior.
- Recovery requirements.
- Security/authorization implications.
- Additional infrastructure required.
- Major risks.

Do not select a winner.

## Evidence discipline

Use:

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION INPUTS`

Stop before final architecture decision.

Every substantive finding must be labeled:

- VERIFIED
- INFERRED
- UNKNOWN

Do not turn an inference into a verified capability.

## Required output

Create:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md`

The report must include:

1. Preflight state.
2. Exact source/configuration/record locations inspected.
3. Auth trigger feasibility findings.
4. Compensation findings.
5. Unknown-outcome findings.
6. Concurrency findings.
7. Retry/recovery findings.
8. Email-confirmation findings.
9. Architecture comparison.
10. Remaining UNKNOWNs.
11. Decision inputs for Node 2 contract.

## Hard boundaries

- NO application code changes.
- NO migration changes.
- NO database trigger creation.
- NO Supabase Dashboard changes.
- NO compensation execution against real users.
- NO retry implementation.
- NO contract locking.
- NO Node 1 changes.
- NO final architecture selection.

This is evidence collection only.