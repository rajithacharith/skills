---
name: integrate-nextjs
description: Add ThunderID authentication to a Next.js application using the official @thunderid/nextjs SDK. Use when asked to "integrate ThunderID into Next.js", "add auth to my Next.js app", or "connect ThunderID with Next.js".
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Next.js Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

Ask the developer to create an application in ThunderID and share the **Client ID** and **Client Secret** before continuing.

Guide them through these steps:

1. Open `https://localhost:8090/console` and sign in (default: `admin` / `secret`)
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: their app name (e.g. `my-nextjs-app`)
   - **Type**: Web Application
   - **Authorized Redirect URL**: `http://localhost:3000`
4. Click **Create** and copy the **Client ID** and **Client Secret** shown on the next screen

Once they paste both values, use them in all subsequent steps. Do **not** use placeholders — wait for the real values.

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/nextjs
```

## Step 3 — Set Environment Variables

Create `.env.local`:

```env
NEXT_PUBLIC_THUNDERID_BASE_URL=https://localhost:8090
NEXT_PUBLIC_THUNDERID_CLIENT_ID=<your-client-id>
THUNDERID_CLIENT_SECRET=<your-client-secret>
THUNDERID_SECRET=<a-random-secret-for-session-signing>
```

Generate `THUNDERID_SECRET` with `openssl rand -base64 32` (at least 32 characters).

## Step 4 — Add ThunderIDProvider to Layout

Edit `app/layout.tsx` — `ThunderIDProvider` handles the OAuth callback automatically; no manual callback route needed:

```tsx
import { ThunderIDProvider } from '@thunderid/nextjs/server'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <ThunderIDProvider>
          {children}
        </ThunderIDProvider>
      </body>
    </html>
  )
}
```

## Step 5 — Add Middleware for Route Protection

Create `middleware.ts` at the project root:

```ts
import {
  thunderIDMiddleware,
  createRouteMatcher,
} from '@thunderid/nextjs/middleware'

const isProtectedRoute = createRouteMatcher(['/dashboard(.*)'])

export default thunderIDMiddleware(async (thunderid, request) => {
  if (isProtectedRoute(request)) {
    await thunderid.protectRoute()
  }
})

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

## Step 6 — Add Auth UI

```tsx
import {
  SignedIn,
  SignedOut,
  SignInButton,
  SignOutButton,
  UserDropdown,
} from '@thunderid/nextjs'

export default function Home() {
  return (
    <main>
      <h1>Next.js Auth Demo</h1>
      <SignedOut>
        <SignInButton>Sign In</SignInButton>
      </SignedOut>
      <SignedIn>
        <UserDropdown />
        <SignOutButton>Sign Out</SignOutButton>
      </SignedIn>
    </main>
  )
}
```

## Step 7 — Run and Verify

```bash
npm run dev
```

Click **Sign In** — you should be redirected to `https://localhost:8090` and returned after login.

## Troubleshooting

**Certificate error** — Visit `https://localhost:8090` in your browser and accept the warning once.

**`invalid_client`** — Double-check the Client ID and Client Secret in `.env.local`.
