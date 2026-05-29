---
name: integrate-javascript
description: Add ThunderID authentication using the universal @thunderid/javascript SDK — works in Node.js, edge runtimes, and bundled browser apps without DOM dependencies. Use when building a custom integration, working in a non-browser JS environment, or when you need direct access to ThunderID's auth primitives.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — JavaScript SDK Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

`@thunderid/javascript` is the universal ThunderID SDK with no DOM or Node.js-specific dependencies. It is the foundation for all other ThunderID SDKs — use it directly when you need low-level auth primitives, are building a custom integration, or are targeting edge runtimes.

## Step 1 — Register an Application

Ask the developer to create an application in ThunderID and share the **Client ID** (and **Client Secret** for confidential/server-side clients) before continuing.

Guide them through these steps:

1. Open `https://localhost:8090/console` and sign in (default: `admin` / `secret`)
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: their app name (e.g. `my-js-app`)
   - **Type**: Single Page Application for public clients; Web Application for confidential clients
   - **Authorized Redirect URL**: their app's redirect URL (e.g. `http://localhost:3000`)
4. Click **Create** and copy the **Client ID** (and **Client Secret** if it's a Web Application)

Once they paste the values, use them in all subsequent steps. Do **not** use placeholders — wait for the real values.

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/javascript
```

## Step 3 — Create a Client

```ts
import { createThunderID } from '@thunderid/javascript'

const thunderid = createThunderID({
  clientId: '<your-client-id>',
  baseUrl: 'https://localhost:8090',
  redirectUri: 'http://localhost:3000/callback',
})
```

## Step 4 — Implement the Auth Flow

**Start login — generate an authorization URL and redirect the user:**

```ts
const { url, codeVerifier, state } = await thunderid.createAuthorizationUrl({
  scope: 'openid profile email',
})

// Store codeVerifier and state (e.g. in sessionStorage or a cookie) for the callback
sessionStorage.setItem('code_verifier', codeVerifier)
sessionStorage.setItem('oauth_state', state)

// Redirect
window.location.href = url
```

**Handle the callback — exchange the code for tokens:**

```ts
const code = new URLSearchParams(window.location.search).get('code')
const codeVerifier = sessionStorage.getItem('code_verifier')!

const tokens = await thunderid.exchangeCode({
  code: code!,
  codeVerifier,
})

// tokens.accessToken, tokens.idToken, tokens.refreshToken
```

**Get user info:**

```ts
const user = await thunderid.getUserInfo(tokens.accessToken)
// { sub, email, name, ... }
```

**Refresh tokens:**

```ts
const refreshed = await thunderid.refreshTokens(tokens.refreshToken)
```

**Sign out:**

```ts
const logoutUrl = thunderid.createLogoutUrl({
  idTokenHint: tokens.idToken,
  postLogoutRedirectUri: 'http://localhost:3000',
})
window.location.href = logoutUrl
```

## Troubleshooting

**Certificate error** — For browsers: visit `https://localhost:8090` and accept the warning. For Node.js: set `NODE_TLS_REJECT_UNAUTHORIZED=0` in dev (never in production).

**`invalid_grant`** — The `codeVerifier` must match the one used to generate the authorization URL. Do not regenerate it between steps.
