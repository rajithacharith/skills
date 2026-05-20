---
name: integrate-express
description: Add ThunderID authentication to an Express application using the official @thunderid/express SDK. Use when asked to "add ThunderID to my Express app", "integrate ThunderID with Express", or "protect Express routes with ThunderID".
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Express Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

1. Open `https://localhost:8090/console` and sign in as `admin` / `admin`
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: your app name
   - **Type**: Web Application
   - **Authorized Redirect URL**: `http://localhost:3000/callback`
4. Copy the **Client ID** and **Client Secret**

### Via the API

First obtain a system API token from the ThunderID console, then:

```bash
curl -kL -X POST https://localhost:8090/applications \
  -H 'Authorization: Bearer <your-system-token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-express-app",
    "inboundAuthConfig": [{
      "type": "oauth2",
      "config": {
        "grantTypes": ["authorization_code", "refresh_token"],
        "responseTypes": ["code"],
        "redirectUris": ["http://localhost:3000/callback"],
        "tokenEndpointAuthMethod": "client_secret_basic",
        "publicClient": false,
        "pkceRequired": true
      }
    }]
  }'
```

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/express
```

## Step 3 — Register Middleware

```ts
import express from 'express'
import { thunderID, requireAuth } from '@thunderid/express'

const app = express()

app.use(thunderID({
  clientId: '<your-client-id>',
  clientSecret: '<your-client-secret>',
  baseUrl: 'https://localhost:8090',
  redirectUri: 'http://localhost:3000/callback',
  sessionSecret: '<random-string-at-least-32-chars>',
}))
```

## Step 4 — Add Auth Routes

The middleware automatically mounts `/login`, `/callback`, and `/logout` routes. You can customise the paths via options, or wire them manually:

```ts
// Manual login trigger
app.get('/login', (req, res) => {
  res.thunderID.signIn()
})

// Manual logout trigger
app.get('/logout', (req, res) => {
  res.thunderID.signOut()
})
```

## Step 5 — Protect Routes

```ts
// Redirect to login if not signed in (web routes)
app.get('/dashboard', requireAuth(), (req, res) => {
  res.send(`Welcome, ${req.thunderID.user.name}`)
})

// Return 401 for API routes
app.get('/api/profile', requireAuth({ redirect: false }), (req, res) => {
  res.json({ user: req.thunderID.user })
})
```

## Step 6 — Run and Verify

```bash
npm run dev   # or node index.js
```

Visit `http://localhost:3000/login` — you should be redirected to `https://localhost:8090` and returned after login.

## Troubleshooting

**Certificate error** — Set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` for local development (remove before deploying).

**Session not persisting** — Ensure `sessionSecret` is set and that your Express app has a session store configured for production.

**`invalid_client`** — Double-check the Client ID and Client Secret.
