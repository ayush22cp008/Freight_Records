# Chat36 — Node 7 Phase 1a Remediation Execution Prompt

Execute the approved Chat36 remediation plan using the updated plan as the source of truth.

## Required Sources
- `03_IMPLEMENTATION/plans/Chat36_Node7_Phase1a_Remediation_Verification_Updated_Plan.md`
- `03_IMPLEMENTATION/plans/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Reconciled_Implementation_Plan.md`
- `03_IMPLEMENTATION/prompts/Chat36_Node7_Phase1a_Remediation_Verification_Antigravity_Prompt.md`

## Execution
1. Inspect the current implementation before changing anything.
2. Execute only the remaining Node 7 Phase 1a remediation defined by the Chat36 plan.
3. For AI, use the existing authoritative event/evidence source and shared summary logic; preserve freshness and ensure rate limiting occurs before expensive AI work. Do not invent a new AI storage architecture.
4. Complete the concurrency verification and address any verified implementation defect.
5. Preserve the locked privacy, token-security, authorization, rate-limit, audit, and Phase 1a scope requirements. If an authoritative audit mechanism is genuinely unavailable, keep it explicitly `UNKNOWN` rather than inventing a parallel audit system.
6. Run the relevant tests and manual verification checks. Do not mark anything VERIFIED without evidence.
7. Update the existing Chat35 implementation report with the actual results, including VERIFIED / UNKNOWN status and any remaining limitations.
8. Commit and push all implementation/report changes to the repository.

## Stop Conditions
- If implementation reality conflicts with the locked architecture or plan, STOP and report the exact conflict.
- Do not start Phase 1b, Phase 3, or unrelated refactoring.
- Do not create Chat37 or any new numbered workstream.
- After successful implementation, verification, report update, commit, and push, STOP for Ayush manual verification/approval.
