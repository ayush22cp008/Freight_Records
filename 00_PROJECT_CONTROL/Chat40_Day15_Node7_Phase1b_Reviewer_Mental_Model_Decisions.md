# Chat40 Day15 — Node 7 Phase 1b Reviewer Mental Model Decisions

## Purpose

This record captures the Reviewer Portal Mental Model decisions locked during Chat40 / Day15 after completion of the Existing Reviewer System Investigation.

The Reviewer Mental Model is a reasoning/architecture stage. It is not a second Existing-System Investigation, UI blueprint, interaction map, or implementation plan.

## Evidence Basis

The Mental Model is grounded in the completed Reviewer Existing-System Investigation and Completion Report under:

`05_DEBUGGING/investigations/`

The existing-system evidence establishes a Reviewer workflow centered on pending onboarding verification, submitted evidence, applicant/requested-role context, and Approve/Reject decisions. Existing Reviewer investigation findings remain authoritative for current-system facts and scope boundaries.

## Reviewer Mental Model — COMPLETE / LOCKED

### 1. Primary Job

**Identity & Evidence Verifier**

The Reviewer verifies whether the applicant actually represents the claimed identity/role (Driver or Company) using submitted evidence, then makes the verification decision.

### 2. Primary Object

**Evidence**

The Reviewer primarily thinks in terms of evidence that must be evaluated to determine whether the claimed Driver/Company identity is genuine.

### 3. Information Model

**Evidence + Applicant + Requested Role**

The minimum required verification context is:

```text
WHO?
Applicant

WHAT ARE THEY CLAIMING?
Requested Role

WHAT PROVES IT?
Evidence
```

Evidence is primary; Applicant and Requested Role provide the necessary context.

### 4. Verification / Decision Model

**Applicant + Role + Evidence → Evaluate → Verify → Approve/Reject**

The Reviewer evaluates the relationship between the applicant, claimed role, and submitted evidence to determine whether the identity/role is verified.

No scoring system, AI decision mechanism, or new automated verification mechanism is introduced by this mental model.

### 5. State Model

**Pending Verification → Verified / Rejected**

The mental model expresses the existing underlying pending → verified/rejected lifecycle as an evidence-verification lifecycle without introducing a new persistent `under_review` state.

```text
Pending Verification
        ↓
     Evaluate
        ↓
 ┌──────┴──────┐
 ↓             ↓
Verified     Rejected
```

### 6. Reviewer Mental Journey

**Verification-first**

```text
Pending Verification
        ↓
Applicant + Claimed Role
        ↓
Evidence
        ↓
Evaluate
        ↓
Verify Identity / Role
        ↓
Approve / Reject
```

### 7. Trust & Evidence Model

**Evidence must support the claimed identity/role**

The Reviewer evaluates whether the submitted evidence corresponds to the applicant and supports the claimed Driver/Company role.

The model does not impose mandatory multiple-evidence requirements or automated confidence scoring unless separately verified and approved in a later scope decision.

### 8. Responsibility Boundary

**Narrow verification boundary**

Reviewer responsibilities:
- examine submitted evidence;
- verify the claimed Driver/Company identity;
- make the verification decision;
- approve or reject.

Outside Reviewer responsibility:
- trip operations;
- delivery operations;
- claims;
- Driver operations;
- Company operations;
- general platform administration.

### 9. Current Mental-Model Problem

**One coherent Reviewer verification-workflow problem**

The current system has both navigation/access problems and evidence → verification → decision clarity problems. These are treated as parts of one larger issue: the Reviewer’s actual verification work is not represented as one sufficiently clear, coherent, role-specific workflow.

This problem is to be solved later through Interaction Mapping, Blueprint, and implementation. It is not a reason to add new Reviewer functionality now.

### 10. Mental-Model Principles

**Evidence-centered, identity-aware, decision-driven**

```text
Applicant
    +
Claimed Role
    +
EVIDENCE
    ↓
Evaluation
    ↓
Identity / Role Verification
    ↓
Approve / Reject
```

Core principles:
- Evidence is central.
- Applicant and Claimed Role provide context.
- Verification is the Reviewer’s core reasoning task.
- Approve/Reject is the resulting human verification decision.
- The Reviewer remains the decision-maker.
- Navigation and interaction should support the complete verification workflow.
- No unnecessary Reviewer responsibilities are added.

## Locked Scope Boundary

This Mental Model does not authorize implementation or introduce new backend business functionality, new authorization rules, new verification states, new evidence types, automated/AI verification, scoring, trip/delivery review, or general administration.

If a later design requires information or behavior not established by the existing system evidence, it must be treated as UNKNOWN and verified before scope expansion.

## Status

**Reviewer Portal Mental Model → COMPLETE / LOCKED**

Next stage:

```text
Reviewer Mental Model → COMPLETE / LOCKED
        ↓
Reviewer Interaction Mapping → NEXT
        ↓
Reviewer Final Blueprint
        ↓
Implementation-Boundary Review
        ↓
Implementation Preparation
```

No implementation should begin from this record alone.
