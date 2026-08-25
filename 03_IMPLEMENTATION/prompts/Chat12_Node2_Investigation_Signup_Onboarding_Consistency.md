# Chat12 — Node 2 Signup / Onboarding Consistency Investigation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Type:** Investigation only  
**Status:** Investigation requested — NO IMPLEMENTATION AUTHORIZED

## Objective

Investigate the unresolved Node 2 signup/onboarding consistency problem using the current source repository, migrations, configuration/evidence available to the agent, and existing project records.

The purpose is to collect evidence needed to make the Node 2 contract decisions. Do not implement or modify the application to fix any issue found.

## Authoritative context

Node 1 is already FINAL LOCKED / COMPLETE. Do not reopen or redesign Node 1 rules.

Node 2 authentication implementation is PAUSED until the Node 2 contract is resolved, independently reviewed, and locked.

Existing investigation already verified that current signup performs Supabase Auth User creation and application identity creation as separate operations, with a possible orphan state when identity creation fails. Use the existing records as starting evidence, but re-check relevant facts against the current source state because evidence can become stale.

Relevant existing record:

`03_IMPLEMENTATION/implementation_reports/Chat11_Node2_Report_Signup_Onboarding_Consistency.md`

Relevant draft contract:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

## Investigation questions

Collect evidence for these areas without deciding or implementing the final architecture:

### 1. Transaction capability

Determine whether the current Supabase/Next.js architecture can make Auth User creation and application identity creation atomic in one database transaction.

Establish the actual boundary between Supabase Auth and application database/PostgREST operations.

### 2. Failure modes

Identify concrete failure points between Auth User creation and application identity creation.

Determine which failures are:
- deterministic;
- temporary/retryable;
- unknown-outcome, such as timeout after a possible database commit.

Do not invent an error taxonomy if the source/evidence does not support it.

### 3. Retry behavior

Determine what the current implementation does on retry.

Establish whether safe bounded retry is technically possible and what evidence exists for retryable vs non-retryable operations.

Do not implement retry logic.

### 4. Idempotency / duplicate prevention

Inspect current schema constraints, unique keys, indexes, and signup behavior relevant to one Auth User → one application identity.

Determine whether a repeated identity-creation attempt can safely detect an already-created identity after an unknown-outcome failure.

Identify any remaining duplicate/race risks.

### 5. Compensation

Determine what server-side capabilities currently exist to compensate for a successfully-created Auth User when application identity creation fails.

Establish whether Auth User deletion/cleanup is available from the current server boundary and what constraints or risks apply.

Do not perform cleanup on real accounts as part of this investigation.

### 6. Recovery

Determine what recovery mechanisms currently exist for an orphaned Auth User or orphaned application identity.

Check specifically for:
- retry/reconciliation jobs;
- queues;
- webhooks/triggers;
- admin recovery tooling;
- manual recovery procedures;
- existing database constraints that help recovery.

If none exist, record that as VERIFIED from the inspected evidence.

### 7. Concurrency

Investigate what happens if multiple signup/retry requests for the same user/email arrive concurrently.

Inspect database constraints and current API behavior for race conditions involving Auth User creation and application identity creation.

Do not add concurrency controls; investigation only.

### 8. Email confirmation / onboarding timing

Inspect the current Supabase Auth configuration/evidence available to the agent and the current signup implementation.

Determine:
- whether email confirmation is currently required;
- what `signUp` returns when confirmation is required;
- when the application identity is currently created relative to confirmation;
- whether the current implementation checks confirmation state;
- what evidence exists for the current behavior.

Do not choose the final product policy yet.

## Additional evidence to collect

Also verify, where relevant:

- current signup route and UI flow;
- current auth/identity database migrations;
- `auth_id` uniqueness and foreign-key behavior;
- service-role usage and server-only boundary;
- current authentication error handling;
- current tests or absence of tests;
- any local-only changes that affect the investigation, clearly separated from committed/pushed state.

## Required investigation discipline

Follow:

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION`

For this task, stop after evidence/root-cause analysis. Do not implement a fix.

Tag every substantive finding:

- **VERIFIED** — directly confirmed by code, migration, logs, configuration, or other evidence.
- **INFERRED** — reasonable conclusion not directly confirmed.
- **UNKNOWN** — evidence is insufficient.

Treat prior evidence as potentially stale if the relevant source code changed.

## Required output

Create an implementation report in:

`03_IMPLEMENTATION/implementation_reports/`

The report must contain:

1. Preflight state.
2. Exact source/config/record locations inspected.
3. Findings for questions 1–8.
4. Evidence supporting each finding.
5. VERIFIED / INFERRED / UNKNOWN classification.
6. Current failure-state matrix.
7. Current retry/idempotency/concurrency behavior.
8. Current compensation/recovery capabilities.
9. Email-confirmation evidence.
10. Remaining unknowns.
11. Recommended decision inputs for the Node 2 contract — recommendations only, not implementation instructions.

## Hard boundaries

- Do NOT modify application source code.
- Do NOT modify database schema or migrations.
- Do NOT change Supabase configuration.
- Do NOT implement retry, compensation, reconciliation, or signup fixes.
- Do NOT lock the Node 2 contract.
- Do NOT reopen Node 1.
- Do NOT claim a decision is approved.

This is an evidence-gathering investigation only.