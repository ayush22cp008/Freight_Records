# Chat5 Node3 — Investigation Report: Authentication Foundation

## Objective
Provide a factual audit of the current authentication implementation, based on inspecting the actual source code and local project state, prior to any decisions about replacing the driver-code login.

---

## 1. Current Authentication Implementation

- **Login route/API:** `/api/auth/login` (Server API Route at `src/app/api/auth/login/route.ts`).
- **Login page:** `/login` (Client Component at `src/app/login/page.tsx`).
- **Session mechanism:** An HttpOnly cookie named `driver_id`.
- **Cookies:** Created by the API route with properties: `httpOnly: true`, `secure: process.env.NODE_ENV === 'production'`, `sameSite: 'lax'`, `path: '/'`, `maxAge: 604800` (1 week).
- **Supabase usage:** The login API queries the `drivers` table server-side (`supabaseServer.from('drivers').select('id, name').eq('driver_code', driver_code).single()`). Protected routes query `trips` and `events` using the `driver_id` from the cookie.
- **Server/client boundaries:** The login UI is a client component that sends credentials via `fetch` to a server API route. Protected routes are React Server Components that read the cookie via `next/headers`.
- **Authentication checks:** Protected server components check `const driverId = cookieStore.get('driver_id')?.value`. If falsy, they invoke Next.js `redirect('/login')`.
- **Logout/sign-out behavior:** **None currently exists.** There is no `/api/auth/logout` endpoint, and no UI button to clear the session cookie.
- **Redirects:** 
  - Successful login triggers a client-side `router.push('/')`.
  - Unauthenticated access to protected routes triggers a server-side `redirect('/login')`.
- **Middleware:** **None.** There is no `src/middleware.ts` intercepting requests.

---

## 2. Current Login Flow

1. User enters driver code on `/login` UI.
2. Client `onSubmit` handler calls `fetch('/api/auth/login', { method: 'POST', body: JSON.stringify({ driver_code }) })`.
3. Server API validates the driver code against the Supabase `drivers` table.
4. If valid, Server API creates a 1-week HttpOnly `driver_id` cookie set to the driver's UUID.
5. Server API responds with `200 OK` and the driver object.
6. Client receives the successful response and executes `router.push('/')`.
7. User arrives at the Trip Hub (`/`).

---

## 3. Session Persistence

- **What creates the session?** `cookieStore.set('driver_id', ...)` inside `src/app/api/auth/login/route.ts`.
- **How long does it persist?** 1 week (`maxAge: 60 * 60 * 24 * 7`).
- **Is it HttpOnly?** Yes.
- **Is it secure/same-site configured?** Yes (`sameSite: 'lax'`, secure only in production).
- **Does refresh preserve it?** Yes, the browser persists the cookie.
- **Does navigating back to `/login` preserve it?** Yes, the cookie remains valid in the browser.
- **Does `/login` detect an existing authenticated session and redirect to `/`?** **No.** `src/app/login/page.tsx` is a client component without session-checking logic. There is no server-side or middleware check on `/login`.
- **Manual Flow (Login → Hub → browser Back → /login → refresh):** The user's session remains fully valid (the cookie is still there), but because `/login` does not check for existing sessions, the user will see the login form again instead of being automatically redirected to the Hub.

---

## 4. Protected-Route Behavior

- **How unauthenticated users are blocked:** Server Components (like `src/app/page.tsx` and `src/app/events/arrival/page.tsx`) explicitly read `cookieStore.get('driver_id')`. If the cookie is absent, they execute `redirect('/login')`.
- **What authentication context is exposed:** Only the `driver_id` (a UUID string) is exposed and utilized by the application logic. The application uses this ID directly in backend Supabase queries to retrieve the driver's active trip and associated events.
