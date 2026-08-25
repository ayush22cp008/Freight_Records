# Chat12 — Node 2 Investigation: Pending Identity vs Verification Timing

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Type:** Investigation only  
**Status:** NO IMPLEMENTATION AUTHORIZED

## Objective

Determine when the Freight application identity should be created relative to Company/Driver verification.

Compare:

- **Model A:** Auth User → Application Identity PENDING → verification → VERIFIED role.
- **Model B:** Auth User → verification → Application Identity created after verification.
- **Model C:** Auth User → one pending Freight Identity with requested role + verification state → trusted Company/Driver role only after verification.

Do not select or implement the final model. Collect evidence and decision inputs only.

## Existing context

Review these records:

`02_ARCHITECTURE/Chat12_Node2_Signup_Onboarding_Consistency_Decision.md`

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md`

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Independent_Review_Claude.md`

Also inspect the locked Node 1 records and current project-control records relevant to identity/role/authorization.

## Questions

### 1. Node 1 invariant compatibility

Determine whether each model can preserve:

```text
1 Auth User → exactly 1 application identity → exactly 1 role
Role = Company OR Driver
```

Identify any model that creates a conflict with Node 1.

### 2. Role establishment

Determine how the system should represent:

- requested role: Driver or Company;
- verification status: PENDING / VERIFIED / REJECTED / other evidence-supported states;
- trusted application role;
- active/usable account state.

Do not assume a user-selected role is trusted.

### 3. Trigger compatibility

Determine whether the proposed `auth.users` trigger architecture remains viable under each model.

Specifically investigate whether the trigger can safely create:

- a generic pending identity;
- a role-specific identity;
- no identity until verification;

without creating a role/authorization security gap.

### 4. Unverified identity risk

For Model A/C, investigate risks created by an identity existing before verification, including:

- Driver code creation/enumeration;
- visibility in Driver marketplace or company-facing views;
- login behavior;
- protected API access;
- resource ownership;
- abandoned/unverified identities;
- duplicate signup/retry behavior.

Determine which controls are required to ensure PENDING identities cannot perform protected business operations.

### 5. Verification workflow

Investigate the proposed hackathon verification model:

```text
User selects requested role
        ↓
Uploads official document
        ↓
PENDING
        ↓
OCR/AI extraction and inspection
        ↓
Controlled/manual reviewer decision
        ↓
VERIFIED / REJECTED / NEEDS_INFO
```

Determine what identity/role state should exist at each stage.

Do not implement OCR, document storage, or review tooling.

### 6. Security boundary

Determine whether the user can ever directly set:

- trusted role;
- verification status;
- active status;
- authorization state.

The final model must make these server-controlled state, not client-controlled fields.

### 7. Authentication and authorization

For each model, determine what happens when a PENDING user:

- logs in;
- refreshes a session;
- calls a protected API;
- attempts Driver-only operations;
- attempts Company-only operations;
- submits a client-supplied role.

### 8. Email confirmation interaction

Determine how these states interact:

```text
email confirmation
identity existence
verification status
trusted role
active/usable account
```

Identify whether any ordering creates a security or consistency problem.

### 9. Data/privacy implications

Compare what data would need to be stored before and after verification.

Identify whether document copies, extracted fields, verification evidence, or reviewer decisions need different retention/access treatment.

Do not design a final retention policy; identify requirements and unknowns.

### 10. Acceptance criteria

For each model, propose the minimum acceptance tests required to prove:

- one Auth User cannot receive two identities;
- one Auth User cannot become both Company and Driver;
- PENDING users cannot perform protected role-specific operations;
- verification cannot be self-approved by the user;
- rejected users cannot gain trusted role access;
- concurrent signup/retry cannot create duplicate identities;
- email confirmation cannot bypass verification/authorization requirements.

## Required comparison

Provide an evidence-based comparison table for Model A, Model B, and Model C covering:

- Node 1 compatibility;
- trigger compatibility;
- security;
- authorization clarity;
- abandoned-account handling;
- email-confirmation interaction;
- verification workflow fit;
- implementation complexity;
- major risks;
- remaining UNKNOWNs.

Do not choose a winner.

## Evidence discipline

Use:

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION INPUTS`

Classify findings as:

- VERIFIED
- INFERRED
- UNKNOWN

Do not turn assumptions into verified facts.

## Hard boundaries

- NO source-code changes.
- NO migration changes.
- NO trigger creation.
- NO Supabase configuration changes.
- NO document-verification implementation.
- NO role/identity implementation.
- NO contract lock.
- NO final architecture selection.

## Required report

Create:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Pending_Identity_vs_Verification.md`

The report must contain:

1. Preflight state.
2. Records/source/configuration inspected.
3. Findings for questions 1–10.
4. A/B/C comparison.
5. Security and authorization implications.
6. Remaining UNKNOWNs.
7. Decision inputs for the Node 2 contract.
