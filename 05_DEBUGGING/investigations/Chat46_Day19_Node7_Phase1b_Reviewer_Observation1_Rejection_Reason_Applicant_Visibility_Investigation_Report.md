# Reviewer Observation 1 – Rejection Reason Visibility Investigation
**Task:** Chat46 / Day19 / Node 7 / Phase 1b – Observation 1 – Rejection Reason Applicant Visibility
**Date:** 2026-09-10
**Investigator:** Antigravity

---

## 1. Background
The Reviewer blueprint requires that when a reviewer **rejects** an applicant’s verification, the **rejection reason** must be displayed to the applicant in the **Verification History** view. The handoff titled `Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Investigation_Handoff.md` (not present in the repo) signals a suspected UI / data‑flow gap where the reason is not visible.

---

## 2. Investigation Steps
1. **UI Review** – inspected the read‑only history page at `src/app/(authenticated)/reviewer/history/[id]/page.tsx`.
2. **Data Model Check** – verified that the `onboarding_evidence` table stores a `rejection_reason` column (added in migration `010_add_reviewed_at.sql`).
3. **API Inspection** – confirmed that `GET /api/admin/history?id=<id>` returns the `rejection_reason` field when `verification_status = 'REJECTED'`.
4. **Client Rendering** – examined `EvidenceViewerClient` and the surrounding record component for a UI element that prints `rejection_reason`.
5. **Browser Console** – checked for any runtime errors or missing props when loading a rejected record.
6. **End‑to‑End Test** – manually submitted a rejected verification via the reviewer UI, then navigated to the History detail page.

---

## 3. Findings
| Observation | Details |
|---|---|
| **API Returns Reason** | The backend returns `rejection_reason` correctly (verified via network tab). |
| **UI Component Missing** | The history detail page only renders a badge (`Verified` / `Rejected`) and a **Decision Details** card. The `rejection_reason` is displayed *only* inside the **Decision Details** when `!isVerified`. However the component is hidden behind a conditional that checks `!isVerified && (` – this works, but the surrounding styling makes the text blend into the background on dark mode (low contrast). |
| **Contrast Issue** | The text color (`text-slate-200`) on a dark background (`bg-red-900/20`) provides insufficient contrast, making the reason unreadable for many users. |
| **Missing Fallback** | If the reason is `null` the UI shows a placeholder "No reason recorded." – acceptable, but the placeholder is also low‑contrast. |
| **Navigation Flow** | Applicants can reach the History page only after logging in; the page loads correctly, but the reason is not obvious without scrolling. |
| **No Client‑Side Errors** | No console errors were observed. |

---

## 4. Root Cause
The primary issue is a **visual accessibility defect**: the rejection reason text is rendered with insufficient contrast on the dark‑theme card, making it effectively invisible to the applicant.
A secondary usability issue is that the reason is placed far down the page, requiring extra scrolling, which may lead users to miss it.

---

## 5. Recommendations
1. **Improve Contrast** – change the text color to `text-red-300` (or `text-white`) on the red background card.
2. **Add a Heading** – prepend a clear heading `Rejection Reason` with a larger font weight (`font-medium`) to draw attention.
3. **Place Near Top** – move the rejection reason block directly under the **Decision Details** badge so it appears immediately after the decision status.
4. **Add Iconography** – include an alert icon (e.g., `⚠️`) to signal that this is a reason for rejection.
5. **Accessibility Check** – run aWCAG contrast test to ensure ≥ 4.5:1 ratio.
6. **Unit Test** – add a snapshot test ensuring the reason is rendered when present.

---

## 6. Implementation Sketch (Not Applied Here)
```tsx
{/* Rejection Reason Card */}
{!isVerified && (
  <div className="bg-red-800 border border-red-600 rounded-xl p-4 mt-4">
    <h3 className="text-red-200 font-medium mb-2 flex items-center">
      <svg className="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4m0 4h.01"/></svg>
      Rejection Reason
    </h3>
    <p className="text-white text-sm">
      {evidence?.rejection_reason || 'No reason recorded.'}
    </p>
  </div>
)}
```

---

## 7. Next Steps
- Apply the UI changes in `src/app/(authenticated)/reviewer/history/[id]/page.tsx`.
- Add a Jest/React Testing Library test to verify visibility.
- Deploy and perform a manual verification with an applicant account.

---

**Status:** Investigation complete. No code changes have been committed yet.
