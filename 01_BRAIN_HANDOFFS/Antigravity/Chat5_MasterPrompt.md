# Chat 5 Master Prompt - Handoff Brain

## 1. Current State of the Application (Node 2 V2 Complete)
- **Authentication & Identity**: The V2 Company and Driver Onboarding Identity flow is fully implemented and active.
- **Evidence Storage**: Onboarding evidence (driving licenses, company GSTs) is securely uploaded as files to a private Supabase Storage bucket (`onboarding_evidence`).
- **Reviewer Queue**: A dedicated Reviewer UI (`/reviewer/queue`) exists. Access is strictly governed by the `reviewer_authorizations` database table. Reviewers view evidence via secure 60-second signed URLs.
- **Atomic Operations**: Approving or rejecting an identity is handled atomically in the backend (`/api/admin/review`). Rejections require a mandatory reason which is displayed to the applicant on their dashboard.
- **UI Fixes**: The shared login screen was fixed to accurately read "Freight Login" instead of "Driver Login", preventing confusion during Company signups.

## 2. Recent Investigations & Findings
- **Signup Routing Bug**: Investigated a flow where Company signups landed on a "Driver Login" screen. Identified that database persistence was flawless (roles are correctly stored in `freight_identities`); the issue was entirely a frontend routing bounce (`/signup` -> `/` -> `/login`) combined with a hardcoded UI title, which has now been fixed.

## 3. Strict Operating Rules & Instructions for the Next Agent
- **Handoff Mechanism**: You are reading this file to gain context from the previous chat. Assume the architecture described in section 1 is locked and actively working.
- **Security First**: Always verify user roles via the database (e.g., `reviewer_authorizations`) before rendering sensitive UI or processing backend actions. Do NOT rely solely on `trusted_role` for admin tasks.
- **Reporting Protocol**: "Every time you build a report or plan file, please give me the path in a copy and paste block so I can tell any other agent, making it easy for me."
- **GitHub Sync Protocol**: "Push on GitHub in the respective folder (`03_IMPLEMENTATION/implementation_reports/`, `03_IMPLEMENTATION/plans/`, or `01_BRAIN_HANDOFFS/Antigravity/`) and remove the report/file from the local file system."
- **Source Code**: Source code changes belong in the `freight_hackathon` repo (`freight/` directory). Documentation/Reports belong in the `Freight_Records` repo.

## 4. Next Steps
- Begin working on the next task (Node 3, Node 4, or further testing) as requested by the user, keeping the established V2 architecture intact.
