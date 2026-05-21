---
name: integrate-browser
description: Add ThunderID authentication to a vanilla browser app using the official @thunderid/browser SDK. Use when asked to "add ThunderID to my vanilla JS app", "integrate ThunderID without a framework", or when there is no frontend framework but the app runs in the browser. For Node.js environments, use /integrate-javascript instead.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Browser SDK Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

`@thunderid/browser` is the browser-specific ThunderID SDK. It builds on `@thunderid/javascript` and adds DOM/window integrations — use it for apps running directly in the browser without a framework.

## Step 1 — Register an Application

1. Open `https://localhost:8090/console` and sign in as `admin` / `admin`
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: your app name
   - **Type**: Single Page Application
   - **Authorized Redirect URL**: `http://localhost:5173/callback` (or your app's callback URL)
4. Copy the **Client ID**

### Via the API

First obtain a system API token from the ThunderID console, then:

```bash
curl -kL -X POST https://localhost:8090/applications \
  -H 'Authorization: Bearer <your-system-token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-browser-app",
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
npm install @thunderid/browser
```

## Step 3 — Initialize the SDK

Create `src/auth.js`:

```js
import { ThunderIDBrowserClient } from '@thunderid/browser'

const auth = new ThunderIDBrowserClient()

await auth.initialize({
  clientId: '<your-client-id>',
  baseUrl: 'https://localhost:8090',
  afterSignInUrl: window.location.origin,
  afterSignOutUrl: window.location.origin,
})

export default auth
```

## Step 4 — Add Sign-In, Sign-Out, and User Display

Replace `src/main.js`:

```js
import './style.css'
import auth from './auth.js'

async function renderApp() {
  const isSignedIn = await auth.isSignedIn()

  if (isSignedIn) {
    const user = await auth.getUser()

    document.querySelector('#app').innerHTML = `
      <div>
        <h2>Welcome, ${user.displayName || user.username}!</h2>
        <p><strong>Email:</strong> ${user.email || 'N/A'}</p>
        <button id="sign-out-btn" type="button">Sign Out</button>
      </div>
    `
    document.querySelector('#sign-out-btn')
      .addEventListener('click', () => auth.signOut())
  } else {
    document.querySelector('#app').innerHTML = `
      <div>
        <p>You are not signed in.</p>
        <button id="sign-in-btn" type="button">Sign In</button>
      </div>
    `
    document.querySelector('#sign-in-btn')
      .addEventListener('click', () => auth.signIn())
  }
}

renderApp()
```

## Step 5 — Run and Verify

```bash
npm run dev
```

Click **Sign In** — you should be redirected to `https://localhost:8090` and returned after login.

## Troubleshooting

**Certificate error** — Visit `https://localhost:8090` in your browser and accept the warning once.

**Not signed in after redirect** — Ensure `auth.initialize()` is awaited on every page load; it detects the `code` query parameter automatically and completes the callback exchange.
