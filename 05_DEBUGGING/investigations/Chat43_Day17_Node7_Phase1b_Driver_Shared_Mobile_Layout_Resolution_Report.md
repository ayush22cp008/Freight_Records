# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Shared Mobile Layout Resolution Report

## 1. Investigation Conclusion

The investigation into the shared mobile layout defect (horizontal overflow leaving a black unused region on the right) has been completed. The root cause is a **shared CSS anti-pattern** applied to photo `<img>` elements across multiple route components, rather than a single global layout wrapper issue.

## 2. Root Cause Analysis

The application uses Tailwind CSS. By default, Tailwind's Preflight applies `max-width: 100%` to all `<img>` elements to prevent them from overflowing their containers.

However, in both affected routes, an explicit `max-w-*` class was applied to the photo evidence `<img>` elements:

**1. Timeline Route (`src/app/(authenticated)/timeline/page.tsx`):**
```tsx
className="max-w-xs rounded shadow-sm border border-gray-200" 
```
`max-w-xs` applies `max-width: 320px`. On a standard 375px mobile screen, after accounting for page and card padding (64px + 48px + 16px = 128px), the available width is ~247px.

**2. Arrival Recorded State (`src/app/(authenticated)/events/arrival/ArrivalClient.tsx`):**
```tsx
className="mt-4 max-w-sm rounded shadow-sm border border-gray-200"
```
`max-w-sm` applies `max-width: 384px`. After padding (64px + 48px = 112px), the available width is ~263px.

**Why this breaks the layout:**
In both cases, applying `max-w-xs` or `max-w-sm` **overrides** the default `max-width: 100%` protection. Because photos from device cameras have large intrinsic dimensions (e.g., 1920x1080), the browser falls back to the newly defined `max-width`. 
Since 320px and 384px are larger than the available container widths (247px and 263px), the images forcefully overflow their containers horizontally. This forces the browser viewport to widen beyond 100vw, producing the observed black/unused region on the right side of the screen.

## 3. Boundary Verification
- **Global Layout:** The parent wrappers and authenticated layout shell (`layout.tsx`) are correctly configured and do not have intrinsic width defects.
- **Other Pages:** Pages without photos (Active Trip, Dashboard, Available Trips, etc.) do not experience this overflow because they do not contain intrinsically wide elements overriding the 100% max-width.
- **Backend/API:** The defect is strictly visual and isolated to the frontend presentation layer.

## 4. Recommendation for Fix

To resolve this issue holistically, we must restore the `max-width: 100%` behavior on mobile viewports while preserving the intended caps on desktop viewports. This requires updating the `<img>` classes in both locations.

**Fix for Timeline (`timeline/page.tsx`):**
Replace `max-w-xs` with `w-full max-w-xs`:
```tsx
className="w-full max-w-xs rounded shadow-sm border border-gray-200" 
```

**Fix for Arrival Recorded (`events/arrival/ArrivalClient.tsx`):**
Replace `max-w-sm` with `w-full max-w-sm`:
```tsx
className="mt-4 w-full max-w-sm rounded shadow-sm border border-gray-200" 
```

By adding `w-full`, the image explicitly matches 100% of the parent container's width on narrow screens, preventing horizontal overflow, while smoothly stopping at 320px/384px on larger screens.

## 5. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
The investigation confirms that the shared symptom is caused by the identical CSS pattern in both files. A focused implementation prompt can now authorize this CSS modification for both the Timeline and the Arrival Client without risking any protected backend boundaries.
