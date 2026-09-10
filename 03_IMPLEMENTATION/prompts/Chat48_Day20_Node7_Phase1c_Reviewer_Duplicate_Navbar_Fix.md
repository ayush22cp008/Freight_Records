# Chat48 / Day20 / Node7 / Phase1c
# Reviewer Duplicate Navbar Fix — Antigravity Implementation Instruction

**Status:** APPROVED FOR IMPLEMENTATION  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day20  
**Executor:** Antigravity  
**Scope:** Remove the duplicate general white navbar from Reviewer pages only

## 1. Objective

Reviewer pages currently render two navigation bars:

```text
General white Freight Navbar
        ↓
Dark Freight Reviewer Navbar
```

The required final structure is:

```text
Reviewer pages
        ↓
Dark Freight Reviewer Navbar only
        ↓
Reviewer content
```

Driver and Company pages must continue using the existing general white Freight Navbar.

## 2. Verified Source Evidence

The authenticated parent layout currently renders the general `Navbar` and passes its content through `RejectedGuard`. The Reviewer layout separately renders `ReviewerNavbar`.

Relevant source files:

```text
src/app/(authenticated)/layout.tsx
src/app/(authenticated)/Navbar.tsx
src/app/(authenticated)/reviewer/layout.tsx
src/app/(authenticated)/reviewer/ReviewerNavbar.tsx
```

The parent layout currently contains:

```tsx
<Navbar userEmail={data.user.email} role={userRole} />
```

and the Reviewer layout renders:

```tsx
<ReviewerNavbar userEmail={data.user.email} />
```

Therefore the duplicate visual navigation is caused by the parent authenticated layout rendering the general Navbar even when `userRole === 'REVIEWER'`.

## 3. Required Behavior

Final role-specific navigation must be:

```text
DRIVER
→ General white Navbar only

COMPANY
→ General white Navbar only

REVIEWER
→ ReviewerNavbar (dark) only
→ No general white Navbar
```

Do not make Driver or Company navigation darker or otherwise redesign them.

Do not remove `Navbar.tsx`; it remains required by Driver and Company.

## 4. Minimal Implementation

Modify the parent authenticated layout so the general `Navbar` is not rendered for Reviewer users.

The implementation should preserve the existing Reviewer layout and `ReviewerNavbar` unchanged unless a strictly necessary compatibility adjustment is proven.

Preferred behavior is equivalent to:

```tsx
{userRole !== 'REVIEWER' && (
  <Navbar userEmail={data.user.email} role={userRole} />
)}
```

Use the repository's existing coding conventions and verify the exact surrounding layout before editing.

Do not alter reviewer authorization, identity resolution, routing, rejection handling, onboarding state logic, Reviewer Queue, Reviewer Verify, or Verification History data behavior.

## 5. Preflight — Mandatory

Before editing source, report:

- project root / current working directory;
- source repository;
- branch;
- current commit SHA;
- working-tree status;
- exact files inspected;
- exact target file to modify.

Stop if repository boundaries are incorrect.

## 6. Allowed Source Scope

Primary target:

```text
src/app/(authenticated)/layout.tsx
```

Do not modify `Navbar.tsx` or `ReviewerNavbar.tsx` unless required by a proven compatibility issue.

Do not modify unrelated source files.

## 7. Validation Requirements

After implementation:

1. run the repository's appropriate type-check/static validation;
2. run `npm run build`;
3. inspect `git diff`;
4. confirm only the intended parent-layout navigation condition changed;
5. confirm there are no unintended changes;
6. report exact command outputs/results.

Do not claim browser/manual verification unless actually performed and documented. Ayush is the final manual verifier.

## 8. Manual Verification Handoff

Ayush should verify at minimum:

```text
A. Reviewer Verification Queue
   → dark Reviewer navbar only
   → no white Freight navbar above it

B. Reviewer Verification page
   → dark Reviewer navbar only

C. Reviewer Verification History
   → dark Reviewer navbar only

D. Driver Dashboard
   → white Freight navbar remains

E. Company Dashboard
   → white Freight navbar remains

F. Reviewer functionality
   → Queue / Verify / History links still work
   → Sign out still works
```

## 9. Implementation Report

After validation, create:

```text
03_IMPLEMENTATION/implementation_reports/Chat48_Day20_Node7_Phase1c_Reviewer_Duplicate_Navbar_Fix_Implementation_Report.md
```

The report must include:

- preflight state;
- exact source file modified;
- concise explanation of the role-based Navbar condition;
- type-check result;
- `npm run build` result;
- final diff/scope verification;
- manual verification status as `PENDING` unless independently documented by Ayush.

## 10. Push Boundary

Do not push automatically.

After implementation, validation, and report creation, wait for the normal project push approval and Ayush's explicit permission.

## 11. Completion Definition

```text
Reviewer duplicate white navbar removed
        ↓
Reviewer dark navbar preserved
        ↓
Driver/Company white navbar preserved
        ↓
Type-check passes
        ↓
Production build passes
        ↓
Diff scope confirmed
        ↓
Implementation report saved
        ↓
Ayush manual verification pending
```
