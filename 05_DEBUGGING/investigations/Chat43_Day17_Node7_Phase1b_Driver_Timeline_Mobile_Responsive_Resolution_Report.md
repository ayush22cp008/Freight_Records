# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Timeline Mobile Responsive Resolution Report

## 1. Investigation Conclusion

The investigation into the horizontal overflow on the Driver Timeline when an event contains a photo has been completed. The root cause has been successfully identified as a CSS conflict on the `<img>` element rendering the photo evidence.

## 2. Root Cause Analysis

In `src/app/(authenticated)/timeline/page.tsx`, the photo evidence is rendered with the following classes:

```tsx
<img 
  src={event.photo_url} 
  alt={`${event.event_type} proof`} 
  className="max-w-xs rounded shadow-sm border border-gray-200" 
/>
```

**Why this causes overflow:**
1. Tailwind CSS normally applies a global Preflight rule `max-width: 100%` to all `<img>` elements to ensure they never overflow their parent container.
2. The explicit `max-w-xs` utility class applies `max-width: 20rem` (320px) to the image, which **overrides** the default `max-width: 100%` protection.
3. The Driver Timeline layout includes significant horizontal padding on mobile:
   - Page container: `p-8` (64px total)
   - Card container: `p-6` (48px total)
   - Image wrapper: `pl-4` (16px total)
   This means on a standard 375px mobile viewport, the available width for the image is roughly `375px - 128px = 247px`.
4. Because the intrinsic dimensions of captured photos are large, and the `max-width` is clamped at 320px instead of 100%, the image forces a minimum width of 320px. This breaks out of the 247px available space, pushing the entire page layout horizontally and creating the observed black unused region on the right side of the viewport.

## 3. Boundary Verification
- **Backend/API/DB:** The root cause is strictly a CSS frontend layout issue. No backend API, RLS, or evidence model logic is defective or involved.
- **Laptop/Desktop:** On larger viewports, the available card width easily exceeds 320px, so the image scales down correctly to 320px without overflowing, explaining why the defect is mobile-specific.

## 4. Recommendation for Fix
The CSS classes on the photo evidence `<img>` element in `src/app/(authenticated)/timeline/page.tsx` must be updated to ensure the image scales to the container width on mobile, while remaining capped on desktop.

**Proposed Fix:**
Replace `max-w-xs` with `w-full max-w-xs` (or `max-w-full sm:max-w-xs`) on the `<img>` element:

```tsx
<img 
  src={event.photo_url} 
  alt={`${event.event_type} proof`} 
  className="w-full max-w-xs rounded shadow-sm border border-gray-200" 
/>
```

Adding `w-full max-w-xs` explicitly forces the image to take up 100% of its available container width (preventing overflow on narrow screens) but caps its maximum growth to 320px (preserving the existing desktop design). 

## 5. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
The investigation confirmed the issue is isolated to frontend responsive rendering rules. A focused implementation prompt can now authorize this CSS modification without risking any protected backend boundaries.
