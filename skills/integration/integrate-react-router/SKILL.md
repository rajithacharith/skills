---
name: integrate-react-router
description: Add ThunderID authentication to a React app using React Router, with the official @thunderid/react and @thunderid/react-router SDKs. Use when asked to "add ThunderID to my React Router app" or when package.json contains react-router.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — React + React Router Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

Ask the developer to create an application in ThunderID and share the **Client ID** before continuing.

Guide them through these steps:

1. Open `https://localhost:8090/console` and sign in (default: `admin` / `secret`)
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: their app name (e.g. `my-react-router-app`)
   - **Type**: Single Page Application
   - **Authorized Redirect URL**: `http://localhost:5173/callback`
4. Click **Create** and copy the **Client ID** shown on the next screen

The `/callback` path matches the `ThunderIDCallback` route added in Step 3 — they must match exactly.

Once they paste the Client ID, use it in all subsequent steps. Do **not** use a placeholder — wait for the real value.

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/react @thunderid/react-router
```

## Step 3 — Configure Provider and Callback Route

Edit `src/main.tsx`:

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { createBrowserRouter, RouterProvider } from 'react-router-dom'
import { ThunderIDProvider } from '@thunderid/react'
import { ThunderIDCallback } from '@thunderid/react-router'
import App from './App'

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
  {
    path: '/callback',
    element: <ThunderIDCallback />,
  },
])

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

## Step 4 — Protect Routes

```tsx
import { useNavigate } from 'react-router-dom'
import { useThunderID } from '@thunderid/react'
import { useEffect } from 'react'

function ProtectedPage() {
  const { isSignedIn, isLoading } = useThunderID()
  const navigate = useNavigate()

  useEffect(() => {
    if (!isLoading && !isSignedIn) {
      navigate('/')
    }
  }, [isSignedIn, isLoading, navigate])

  if (isLoading) return <div>Loading...</div>
  if (!isSignedIn) return null

  return <div>Protected content</div>
}
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
