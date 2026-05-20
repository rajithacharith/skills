---
name: integrate-node
description: Add ThunderID authentication to a Node.js application using the official @thunderid/node SDK. Use when asked to "add ThunderID to my Node.js app", "integrate ThunderID with Fastify", "add ThunderID to Hono", or any Node.js server framework without a dedicated ThunderID SDK. For Express specifically, prefer /integrate-express.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Node.js Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

`@thunderid/node` is the generic Node.js SDK — use it with Fastify, Hono, Koa, or any other Node.js framework. For Express, use `/integrate-express` instead.

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
    "name": "my-node-app",
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
npm install @thunderid/node
```

## Step 3 — Create a Client

```ts
import { createThunderID } from '@thunderid/node'

export const thunderid = createThunderID({
  clientId: '<your-client-id>',
  clientSecret: '<your-client-secret>',
  baseUrl: 'https://localhost:8090',
  redirectUri: 'http://localhost:3000/callback',
})
```

## Step 4 — Wire Up Auth Routes

**Login route — redirect the user to ThunderID:**

```ts
app.get('/login', async (req, reply) => {
  const { url, codeVerifier, state } = await thunderid.createAuthorizationUrl()
  // Persist codeVerifier and state in the session
  req.session.codeVerifier = codeVerifier
  req.session.oauthState = state
  reply.redirect(url)
})
```

**Callback route — exchange the code for tokens:**

```ts
app.get('/callback', async (req, reply) => {
  const { code, state } = req.query as { code: string; state: string }
  const tokens = await thunderid.exchangeCode({
    code,
    codeVerifier: req.session.codeVerifier,
    state,
    expectedState: req.session.oauthState,
  })
  req.session.user = await thunderid.getUserInfo(tokens.accessToken)
  reply.redirect('/')
})
```

**Logout route:**

```ts
app.get('/logout', async (req, reply) => {
  req.session.destroy()
  const logoutUrl = thunderid.createLogoutUrl({ postLogoutRedirectUri: 'http://localhost:3000' })
  reply.redirect(logoutUrl)
})
```

## Step 5 — Protect Routes

**Verify a token from an `Authorization` header (API routes):**

```ts
app.get('/api/profile', async (req, reply) => {
  const token = req.headers.authorization?.replace('Bearer ', '')
  if (!token) return reply.code(401).send({ error: 'Unauthorized' })

  const user = await thunderid.verifyToken(token)
  reply.send({ user })
})
```

**Check session (web routes):**

```ts
app.get('/dashboard', async (req, reply) => {
  if (!req.session.user) return reply.redirect('/login')
  reply.send(`Welcome, ${req.session.user.name}`)
})
```

## Troubleshooting

**Certificate error** — Set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` for local development (remove before deploying).

**`invalid_grant`** — The `codeVerifier` in the callback must match what was stored during the login redirect.

**`invalid_client`** — Double-check the Client ID and Client Secret.
