# Chat12 Node 2 — Q7 Investigation Prompt: Final Acceptance-Test Matrix

Review the authoritative Node 2 records and build the final acceptance-test matrix for Node 2.

Q1–Q6 are locked and must not be reopened unless new evidence creates a genuine conflict.

Cover at minimum:
- signup atomicity;
- one-user → one-identity and DB uniqueness;
- requested role vs trusted role;
- verification boundaries and PENDING/REJECTED access;
- Q2 email-confirmation + Active gate;
- Q3 session/refresh/logout/CSRF/live DB Active enforcement;
- Q5 Auth rate limiting and 429/client-IP behavior;
- Q6 RLS/service_role boundary, privileged authorization, audit logging, FORCE RLS/table-owner checks, and service_role allowlist;
- role enforcement and Node 1 authorization handoff;
- authenticated identity-context derivation;
- cross-user/IDOR and wrong-role cases;
- stale-session/state-change cases;
- security-sensitive failure cases;
- required implementation evidence and Ayush verification.

For every test, define:
1. Test ID
2. Scenario/precondition
3. Action
4. Expected result
5. Security boundary being verified
6. Evidence required
7. Source/locked decision

Do not implement anything.
Do not invent new architecture.
Do not reopen Q1–Q6.

Return:
- final matrix;
- gaps or contradictions, if any;
- minimum acceptance gate for Node 2;
- recommendation: READY FOR REVIEW / CHANGES REQUIRED.