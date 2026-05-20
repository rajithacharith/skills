---
name: integrate-tanstack-router
description: Add ThunderID authentication to a React app using TanStack Router, with the official @thunderid/react and @thunderid/tanstack-router SDKs. Use when asked to "add ThunderID to my TanStack Router app" or when package.json contains @tanstack/react-router.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — React + TanStack Router Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

1. Open `https://localhost:8090/console` and sign in as `admin` / `admin`
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: your app name
   - **Type**: Single Page Application
   - **Authorized Redirect URL**: `http://localhost:5173/callback`
4. Copy the **Client ID**

### Via the API

First obtain a system API token from the ThunderID console, then:

```bash
curl -kL -X POST https://localhost:8090/applications \
  -H 'Authorization: Bearer <your-system-token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-tanstack-router-app",
    "inboundAuthConfig": [{
      "type": "oauth2",
      "config": {
        "grantTypes": ["authorization_code", "refresh_token"],
        "responseTypes": ["code"],
        "redirectUris": ["http://localhost:5173/callback"],
        "tokenEndpointAuthMethod": "none",
        "publicClient": true,
        "pkceRequired": true
      }
    }]
  }'
```

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/react @thunderid/tanstack-router
```

## Step 3 — Configure Provider and Callback Route

Edit `src/main.tsx`:

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { RouterProvider } from '@tanstack/react-router'
import { ThunderIDProvider } from '@thunderid/react'
import { router } from './router'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <ThunderIDProvider
      clientId="<your-client-id>"
      baseUrl="https://localhost:8090"
    >
      <RouterProvider router={router} />
    </ThunderIDProvider>
  </StrictMode>
)
```

Add the callback route in your router definition (`src/router.tsx`):

```tsx
import { createRouter, createRoute, createRootRoute } from '@tanstack/react-router'
import { ThunderIDCallback } from '@thunderid/tanstack-router'

const rootRoute = createRootRoute()

const callbackRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/callback',
  component: ThunderIDCallback,
})

export const router = createRouter({
  routeTree: rootRoute.addChildren([callbackRoute /*, ...other routes */]),
})
```

## Step 4 — Protect Routes

Use `beforeLoad` to guard protected routes:

```tsx
import { createRoute, redirect } from '@tanstack/react-router'
import { getThunderIDAuth } from '@thunderid/tanstack-router'

const protectedRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/dashboard',
  beforeLoad: ({ context }) => {
    const auth = getThunderIDAuth(context)
    if (!auth.isSignedIn) {
      throw redirect({ to: '/' })
    }
  },
  component: Dashboard,
})
```

## Step 5 — Add Auth UI

```tsx
import {
  SignedIn, SignedOut, SignInButton, SignOutButton, Loading, User,
} from '@thunderid/react'

function App() {
  return (
    <>
      <Loading><div>Loading...</div></Loading>
      <SignedIn><SignOutButton>Sign Out</SignOutButton></SignedIn>
      <SignedOut><SignInButton>Sign In</SignInButton></SignedOut>
      <SignedIn>
        <User>
          {(user) => user && <p>Welcome, {user.name || user.username}!</p>}
        </User>
      </SignedIn>
    </>
  )
}
```

## Step 6 — Run and Verify

```bash
npm run dev
```

Click **Sign In** — you should be redirected to `https://localhost:8090` and returned to your app after login.

## Troubleshooting

**Certificate error** — Visit `https://localhost:8090` in your browser and accept the warning once.

**Callback not working** — Ensure the redirect URL registered in the ThunderID console exactly matches `http://localhost:5173/callback`.
