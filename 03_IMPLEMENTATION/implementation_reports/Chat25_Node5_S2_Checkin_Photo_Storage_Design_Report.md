# Chat25 — Node 5 Subnode 5.S2 — Check-in Photo Storage Design Report

## 1. Subnode Status
READY FOR REVIEW

## 2. Parent Node / Subnode Rationale
During Node 5 development, the Chat25 Check-in bug investigation revealed that while the API validation bug could be fixed in source code, the photo upload feature was natively broken because the required Supabase Storage bucket (`event-photos`) and its policies were never created. Because adding database infrastructure and security policies is beyond the scope of a small bug fix, this Subnode 5.S2 was created to isolate the storage design.

## 3. Records Baseline
- Investigated `05_DEBUGGING/investigations/Chat25_Checkin_Bug_Investigation_Report.md`.
- Read `03_IMPLEMENTATION/implementation_reports/Chat25_Checkin_Bug_Implementation_Report.md`.

## 4. Source Baseline
- Examined `src/app/api/upload-photo/route.ts`.
- Examined `src/lib/supabase-server.ts`.
- Examined `src/db/migrations/005_v2_onboarding_evidence.sql`.

## 5. Existing Upload Flow
1. Client selects photo and calls `POST /api/upload-photo`.
2. The `upload-photo` API route generates a random filename: `${Date.now()}-${random}.${ext}`.
3. The API route uses `supabaseServer` to upload the file to `event-photos`.
4. The API route returns the public URL using `getPublicUrl`.

## 6. Existing Storage Configuration Evidence
- There is no SQL migration that creates the `event-photos` bucket in the `storage.buckets` table.

## 7. Bucket Design
- **Bucket ID**: `event-photos`
- **Bucket Name**: `event-photos`
- **Public**: `true` (The current application strictly relies on `getPublicUrl` to display images in the Timeline and provide evidence URLs to the AI Summary without signed-URL overhead).

## 8. Access-Control / RLS Design
- **INSERT Access**: The backend API route (`/api/upload-photo/route.ts`) uses `supabaseServer`, which is instantiated with the `SUPABASE_SERVICE_ROLE_KEY`. This service role key **completely bypasses Row Level Security (RLS)**.
- **Identity/Path Binding**: The backend route completely lacks identity binding. It does not check `supabase.auth.getUser()`, nor does it enforce trip-based paths. 
- **Storage Policy Conclusion**: Because the upload uses a service role proxy, a PostgreSQL `INSERT` Storage policy is neither required nor enforceable. 

## 9. URL/Read-Access Design
- Since the bucket is configured as `public: true`, the Supabase Storage API natively serves files without requiring an explicit `SELECT` policy in `storage.objects`.

## 10. Migration Design
The narrowest, safest fix to satisfy the current application architecture is a single SQL statement to instantiate the public bucket:
```sql
-- Create event-photos bucket for Check-in/Arrival/Departure evidence
INSERT INTO storage.buckets (id, name, public) 
VALUES ('event-photos', 'event-photos', true)
ON CONFLICT (id) DO NOTHING;
```

## 11. Security Assessment
**CRITICAL SECURITY GAP**: The current architecture is highly insecure. `/api/upload-photo/route.ts` is an unauthenticated public endpoint that uses a privileged service role key to upload arbitrary files to your database bucket. 

While the proposed migration above will successfully "fix the bug" and make the product work, it codifies this insecure architecture. A proper security design would:
1. Authenticate the driver in the API route.
2. Generate a secure, deterministic path (e.g., `{driver_id}/{trip_id}/{timestamp}.jpg`).
3. (Optional but recommended) Perform direct client-side uploads using standard RLS policies instead of a server-side proxy.

## 12. Node 5 / S1 Dependency Assessment
This bucket is required for legacy Node 3/4 check-ins and is fully decoupled from the Node 5 S1 event schema migrations. It can be safely implemented immediately.

## 13. Proposed Implementation Scope
Create `src/db/migrations/007_event_photos_bucket.sql` containing the bucket initialization script.

## 14. Decision Required from ChatGPT/Ayush
Please explicitly decide between:
- **Option A (Minimal Fix)**: Approve the proposed migration to simply create the bucket so the hackathon flow works, accepting the insecure unauthenticated upload endpoint.
- **Option B (Security Rewrite)**: Authorize expanding the scope to rewrite `/api/upload-photo/route.ts` to enforce authentication and secure path binding before creating the bucket.

## 15. VERIFIED / INFERRED / UNKNOWN Summary
- **VERIFIED**: The upload route uses the service role key.
- **VERIFIED**: The upload route does not check authentication.
- **VERIFIED**: The bucket must be public for Timeline display.

## 16. Explicit Non-Changes
```text
Application source modified = NO
Database schema modified = NO
Tests added = NO
Production/shared data changed = NO
Commit = NO
Push = NO
Implementation = NO
```
