# Chat46 — Day 19 — Node 7 — Phase 1b
# Reviewer + Driver + Company Truth Audit — Evidence Questionnaire Follow-up

**Status:** INVESTIGATION ONLY — NO IMPLEMENTATION AUTHORIZED  
**Investigator:** Antigravity  
**Purpose:** Ask and answer the exact questions required to establish what actually exists in the repository, live database, runtime, RLS, and storage.  

---

## 1. Why This Follow-up Exists

The previous truth-audit report found real source-code defects, especially the mismatch between preserved 1:N `onboarding_evidence` history and frontend `.single()` queries.

However, that report also marked some production facts as `VERIFIED` while explicitly relying on assumptions such as:

```text
"assuming migration 011 was manually run"
```

That is not sufficient evidence for production truth.

This follow-up is therefore a **questionnaire for Antigravity to answer with direct evidence**.

The goal is not to produce another broad opinion. The goal is to answer:

```text
WHAT EXISTS?
WHAT IS MISSING?
WHAT IS WRONG?
WHAT ACTUALLY HAPPENS?
WHAT IS ONLY ASSUMED?
```

---

## 2. Hard Rules

### No implementation

Do NOT change:

- source code;
- migrations;
- database schema;
- RLS policies;
- storage policies;
- production data;
- Reviewer behavior;
- Driver behavior;
- Company behavior.

### Evidence labels

Every answer MUST use one of:

```text
VERIFIED
INFERRED
UNKNOWN
BLOCKED
```

### Critical distinction

For every question, explicitly identify whether the answer comes from:

```text
SOURCE CODE
REPOSITORY MIGRATION
LIVE DATABASE
LIVE RUNTIME
RLS POLICY INSPECTION
STORAGE INSPECTION
HISTORICAL RECORD
INFERENCE
```

Never call an assumption `VERIFIED`.

---

# 3. Environment Questions

### Q1. What exact application/runtime are you investigating?

Answer:

```text
Repository:
Branch:
Commit SHA:
Application URL/environment:
Supabase project/environment identifier if safely available:
Investigation timestamp:
```

Classification:

```text
VERIFIED / UNKNOWN / BLOCKED
```

Evidence:

```text
Exact command/output/source:
```

### Q2. Did you inspect the actual live database, or only repository migrations/source code?

Answer exactly one:

```text
LIVE DATABASE INSPECTED
REPOSITORY ONLY
PARTIAL LIVE ACCESS
BLOCKED
```

Provide evidence.

### Q3. Did you inspect the actual live storage bucket and objects?

Answer with evidence.

---

# 4. `freight_identities` Questions

### Q4. Does `freight_identities` actually exist in the live database?

Do not infer from migrations.

Provide:

```text
Table existence:
Columns:
Constraints:
Indexes relevant to auth_id/role/status:
Live inspection evidence:
Classification:
```

### Q5. For the actual Driver test applicant, what is the exact live row?

Return:

```text
auth_id
identity id
email (mask if appropriate)
requested_role
trusted_role
verification_status
created_at
updated_at
reviewed_at if present
```

Do not substitute source-code expectations for row values.

### Q6. For the actual Company test applicant, provide the same live values.

### Q7. Does `requested_role` ever differ from `trusted_role` in an expected pending/rejected state?

Show an actual example or say UNKNOWN.

---

# 5. `onboarding_evidence` Questions

### Q8. Does `onboarding_evidence` actually exist in production?

Provide direct evidence.

### Q9. What is the actual live schema?

Return exact:

```text
columns
primary key
foreign keys
unique constraints
indexes
```

### Q10. Is `auth_id` unique in production?

This question is critical.

Answer:

```text
YES / NO / UNKNOWN
```

Provide the exact constraint/index evidence.

### Q11. For the Driver test applicant, list ALL evidence rows.

For every row return:

```text
id
auth_id
role_type
document_type
status
version
storage_path
mime_type
size_bytes
rejection_reason
created_at
```

Do not return only the latest row.

### Q12. For the Company test applicant, list ALL evidence rows using the same fields.

### Q13. How many evidence rows exist for each applicant?

Explicitly state:

```text
Driver count:
Company count:
```

### Q14. Are the live Driver evidence rows actually `DRIVING_LICENCE`?

Are the live Company evidence rows actually `GST`?

If any row differs, show the exact value.

---

# 6. The “Driver + GST Document” Mystery

### Q15. For the screenshot/observation where a Driver appeared as “GST Document”, what exact live evidence row was associated with that Driver?

Provide:

```text
identity.auth_id
identity.requested_role
identity.trusted_role
identity.verification_status
evidence.id
evidence.role_type
evidence.document_type
evidence.status
evidence.created_at
```

### Q16. What exactly caused the displayed “GST Document” label?

Choose the strongest evidence-backed root cause:

```text
A. Wrong live database document_type
B. Wrong frontend mapping
C. Wrong query/join association
D. Multiple evidence rows + wrong row selected
E. Test-data contamination
F. Configuration/environment mismatch
G. UNKNOWN
```

Do not answer from source code alone if a live row can be inspected.

### Q17. Show the exact source line and exact live row together.

The answer must make it possible to reproduce the mapping from:

```text
live row → query result → label
```

---

# 7. Reviewer Queue Questions

### Q18. Exactly what database rows qualify an applicant for Queue?

Answer using actual query conditions.

### Q19. Exactly which evidence row is selected for a Queue card when an applicant has multiple evidence rows?

Explain:

```text
SQL ordering
array ordering
JavaScript selection
status filtering
```

### Q20. Can Queue select an old row instead of the newest current pending evidence?

Provide either:

```text
controlled runtime reproduction
```

or clearly classify as:

```text
INFERRED from source
```

### Q21. Does Queue use the same document type vocabulary as onboarding writes?

Compare exact values:

```text
onboarding write value:
Queue expected/mapped values:
Result:
```

---

# 8. Reviewer Verify Questions

### Q22. When Reviewer opens `/reviewer/verify/[id]`, exactly how many evidence rows can the page receive?

### Q23. Is the Verify query constrained to:

```text
PENDING
newest
one row
correct applicant
```

Answer each separately.

### Q24. What happens when an applicant has two or more evidence rows?

Need actual:

```text
query result
HTTP/error behavior
rendered UI
```

If not runtime-tested, say UNKNOWN/INFERRED.

### Q25. Does the evidence row displayed in Verify match the evidence row whose ID is persisted into `reviewer_decisions`?

Demonstrate with a concrete review if possible.

---

# 9. Approve / Reject Questions

### Q26. On APPROVE, enumerate every database write in exact order.

For example:

```text
identity update
 evidence update
history insert
profile insert
```

Provide actual source evidence.

### Q27. On REJECT, enumerate every database write in exact order.

### Q28. Which write errors are checked?

For every write answer:

```text
checked / ignored / unknown
```

### Q29. Can APPROVE leave a partial state?

Investigate specifically:

```text
identity VERIFIED + no history
identity VERIFIED + no profile
history + no profile
history + missing evidence
```

Do not answer “atomic” unless transaction evidence exists.

### Q30. Can REJECT leave a partial state?

Use the same standard.

### Q31. Is there an actual database transaction/RPC/server-side transaction covering the whole decision?

Answer:

```text
YES / NO / UNKNOWN
```

Provide exact evidence.

---

# 10. `reviewer_decisions` Questions

### Q32. Does `reviewer_decisions` actually exist in the live database?

This must be directly inspected.

### Q33. What is its exact live schema?

Return:

```text
columns
PK
FKs
unique constraints
indexes
RLS enabled
```

### Q34. What RLS policies actually exist in production on `reviewer_decisions`?

List exact policy names and operations:

```text
SELECT
INSERT
UPDATE
DELETE
```

### Q35. Does the live table contain the Driver's historical rejection decision?

Provide the exact row.

### Q36. Does the live table contain the Company history, if a Company rejection/review was actually performed?

### Q37. Does the historical `evidence_id` point to a real evidence row?

For each decision row verify:

```text
evidence_id exists?
matching auth_id?
matching identity_id?
storage_path exists?
```

---

# 11. Recovery Questions

### Q38. For the actual rejected Driver, what is the exact pre-recovery state?

Return:

```text
identity status
old evidence rows
old reviewer_decision rows
old rejection reason
```

### Q39. What exactly happens during Driver recovery?

Capture the real sequence:

```text
upload
INSERT evidence
UPDATE identity
API response
frontend response
navigation/refresh
```

### Q40. What is the exact live post-recovery state?

Return:

```text
identity status
all evidence rows
all reviewer_decisions
```

### Q41. Does the old rejected evidence remain preserved?

### Q42. Does the new current evidence become distinguishable from the old evidence?

Use:

```text
id
version
status
created_at
```

### Q43. Can recovery partially fail?

Test or reason from exact code, but label correctly.

Examples:

```text
new evidence + identity REJECTED
new evidence + identity PENDING
old evidence only
new evidence + no response
```

---

# 12. Company Recovery Questions

Repeat Questions 38–43 for Company.

Do not reuse Driver evidence as proof for Company.

---

# 13. History Questions

### Q44. Does Reviewer History actually show historical decisions from live data?

### Q45. Is newest-first ordering verified against actual timestamps?

### Q46. Can two decisions for the same applicant coexist?

Show actual rows if available.

### Q47. Does History detail show the exact linked evidence used by the original decision?

### Q48. Can an applicant access another applicant’s history?

Provide a negative authorization test or direct RLS evidence.

---

# 14. Applicant Authorization Questions

### Q49. Can a Driver modify another user's `freight_identities` row?

### Q50. Can a Driver read another user's onboarding evidence?

### Q51. Can a Company read another user's onboarding evidence?

### Q52. Can a non-reviewer read Reviewer History?

### Q53. Can an applicant insert/update/delete `reviewer_decisions` directly?

These must be evaluated from **live RLS policies where possible**, not only migrations.

---

# 15. Service-Role Security Questions

For every API route using a privileged/service-role client:

### Q54. Is authentication verified before service-role access?

### Q55. Is Reviewer authorization verified where reviewer authority is required?

### Q56. Is the target identity constrained to the intended applicant/reviewer context?

### Q57. Can a client submit an arbitrary `identity_id` and cause unauthorized mutation?

### Q58. Can an authenticated non-reviewer invoke the decision API successfully?

Provide runtime evidence if possible.

---

# 16. Storage Questions

### Q59. Is the onboarding evidence bucket actually private in production?

### Q60. What storage policies actually exist?

### Q61. Does the historical Driver evidence storage object still exist after recovery?

### Q62. Does the historical Company evidence storage object still exist after recovery?

### Q63. Can an authorized Reviewer generate/access the historical evidence URL?

### Q64. Can an unauthorized applicant access another applicant's object?

---

# 17. Production Drift Questions

### Q65. Which migrations are present in the repository but NOT proven executed in production?

List them explicitly.

### Q66. Are there any live columns not represented by repository migrations?

### Q67. Are there any live RLS policies not represented in repository migrations?

### Q68. Are there repository migrations that claim constraints/indexes that are absent in production?

### Q69. Is `reviewer_decisions` production parity actually proven?

Answer only:

```text
PROVEN / NOT PROVEN / BLOCKED
```

### Q70. Is `onboarding_evidence` production cardinality actually proven to be 1:N?

---

# 18. Runtime Reproduction Questions

Where safe and possible, perform controlled tests.

### Q71. Fresh Driver onboarding

Record actual observed result.

### Q72. Fresh Company onboarding

Record actual observed result.

### Q73. Driver rejection

Record:

```text
UI
identity
 evidence
history
```

### Q74. Company rejection

Same.

### Q75. Driver recovery

Record exact before/after database state.

### Q76. Company recovery

Record exact before/after database state.

### Q77. Re-review after recovery

Record whether:

```text
Queue
Verify
Decision
History
```

all work correctly.

### Q78. Multiple evidence row behavior

Create or use a safe test case and capture the actual UI/API result.

### Q79. Unauthorized access tests

Capture actual status/result and database effect.

---

# 19. Final Evidence Reconciliation Questions

### Q80. Which findings from the previous report are genuinely VERIFIED by live evidence?

List them.

### Q81. Which findings are only VERIFIED by source inspection?

List them separately.

### Q82. Which previous `VERIFIED` claims must be downgraded to `INFERRED`, `UNKNOWN`, or `BLOCKED`?

Be explicit.

### Q83. What is the strongest confirmed root cause for the current Reviewer/Driver/Company problem?

One sentence plus evidence.

### Q84. What is the second-order contributing cause?

### Q85. Are there additional defects that the previous report missed?

Do not limit the answer to the three previously reported issues.

---

# 20. Final “What Exists / What Is Missing / What Is Wrong” Questionnaire

### Q86. What exists in repository but is NOT proven in production?

### Q87. What exists in production but is NOT represented in repository?

### Q88. What exists in both repository and production but is wired incorrectly?

### Q89. What is definitely missing from the system?

### Q90. What is definitely broken?

### Q91. What is only a source-level risk rather than a demonstrated runtime failure?

### Q92. What remains completely unknown?

### Q93. What is blocked because Antigravity lacks access?

---

# 21. Mandatory Final Defect Table

Produce:

| ID | Finding | Evidence | Source / DB / Runtime | Classification | Severity | Root Cause | Impact | Implementation needed? |
|---|---|---|---|---|---|---|---|---|

No vague findings.

---

# 22. Mandatory Final Truth Table

Produce:

| Question Area | Answer | Evidence | Classification |
|---|---|---|---|
| Driver onboarding | | | |
| Company onboarding | | | |
| Evidence schema | | | |
| Evidence live rows | | | |
| Reviewer Queue | | | |
| Reviewer Verify | | | |
| Approve | | | |
| Reject | | | |
| Recovery | | | |
| Reviewer History | | | |
| Historical evidence | | | |
| RLS | | | |
| Service-role security | | | |
| Storage | | | |
| Production drift | | | |

---

# 23. Final Decision

End with exactly one:

```text
A. VERIFIED HEALTHY
B. VERIFIED DEFECTS — IMPLEMENTATION HANDOFF REQUIRED
C. PARTIALLY VERIFIED — MORE EVIDENCE REQUIRED
D. PRODUCTION DRIFT
E. BLOCKED — INSUFFICIENT ACCESS
```

Then answer:

```text
1. What should be fixed next?
2. What evidence proves that fix is necessary?
3. Does the fix require governance reopening?
4. What must NOT be changed?
```

### Important

Do not produce an implementation prompt from this questionnaire.

This file only asks Antigravity to establish the truth. Any fix must be authorized separately after the investigation is reviewed.
