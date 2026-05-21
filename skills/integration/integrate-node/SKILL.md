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

`@thunderid/node` is the generic Node.js SDK — use it with the built-in `http` module, Fastify, Hono, Koa, or any other Node.js framework. For Express, use `/integrate-express` instead.

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

## Step 3 — Initialize the Client

Create `index.js` and initialize `ThunderIDNodeClient` with your application credentials:

```js
const http = require('http');
const { URL } = require('url');
const { randomUUID } = require('crypto');
const { ThunderIDNodeClient } = require('@thunderid/node');

const PORT = 3000;
const SESSION_COOKIE = 'tid_session';

const auth = new ThunderIDNodeClient();

function getSessionId(req) {
  const cookieHeader = req.headers.cookie ?? '';
  for (const part of cookieHeader.split(';')) {
    const [name, value] = part.trim().split('=');
    if (name === SESSION_COOKIE) return decodeURIComponent(value);
  }
  return null;
}

async function main() {
  await auth.initialize({
    clientId: '<your-client-id>',
    clientSecret: '<your-client-secret>',
    baseUrl: 'https://localhost:8090',
    afterSignInUrl: 'http://localhost:3000/callback',
    afterSignOutUrl: 'http://localhost:3000',
  });

  const server = http.createServer(async (req, res) => {
    const url = new URL(req.url, `http://localhost:${PORT}`);

    try {
      if (url.pathname === '/') {
        const sessionId = getSessionId(req);
        const signedIn = sessionId && (await auth.isSignedIn(sessionId));
        res.writeHead(200, { 'Content-Type': 'text/html' });
        res.end(signedIn
          ? '<a href="/profile">View profile</a> | <a href="/logout">Sign out</a>'
          : '<a href="/login">Sign in</a>'
        );

      } else if (url.pathname === '/profile') {
        const sessionId = getSessionId(req);
        if (!sessionId || !(await auth.isSignedIn(sessionId))) {
          res.writeHead(302, { Location: '/login' });
          return res.end();
        }
        const user = await auth.getUser(sessionId);
        res.writeHead(200, { 'Content-Type': 'text/html' });
        res.end(`
          <h1>Welcome, ${user.name || user.username}!</h1>
          <p><strong>Email:</strong> ${user.email}</p>
          <a href="/logout">Sign out</a>
        `);

      } else if (url.pathname === '/login') {
        let sessionId = getSessionId(req);
        const extraHeaders = {};
        if (!sessionId) {
          sessionId = randomUUID();
          extraHeaders['Set-Cookie'] =
            `${SESSION_COOKIE}=${sessionId}; HttpOnly; SameSite=Lax; Path=/`;
        }
        await auth.signIn((authUrl) => {
          res.writeHead(302, { ...extraHeaders, Location: authUrl });
          res.end();
        }, sessionId);

      } else if (url.pathname === '/callback') {
        const code = url.searchParams.get('code');
        const state = url.searchParams.get('state');
        const sessionState = url.searchParams.get('session_state');
        const sessionId = getSessionId(req);

        if (!sessionId || !code || !state) {
          res.writeHead(400);
          return res.end('Bad request');
        }

        await auth.signIn(() => {}, sessionId, code, sessionState, state);
        res.writeHead(302, { Location: '/profile' });
        res.end();

      } else if (url.pathname === '/logout') {
        const sessionId = getSessionId(req);
        if (!sessionId) {
          res.writeHead(302, { Location: '/' });
          return res.end();
        }
        const signOutUrl = await auth.signOut(sessionId);
        res.writeHead(302, {
          Location: signOutUrl,
          'Set-Cookie': `${SESSION_COOKIE}=; HttpOnly; SameSite=Lax; Path=/; Max-Age=0`,
        });
        res.end();

      } else {
        res.writeHead(404);
        res.end('Not found');
      }
    } catch {
      res.writeHead(500);
      res.end('Internal server error');
    }
  });

  server.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
  });
}

main();
```

**How the sign-in flow works:**

1. `GET /login` — generates a session ID cookie and calls `signIn` with an `authUrlCallback` that redirects to ThunderID.
2. `GET /callback` — ThunderID redirects back with `code` and `state`; calling `signIn` again exchanges the code for tokens.
3. `GET /logout` — calls `signOut` to get the OIDC end-session URL, clears the cookie, redirects to complete logout at ThunderID.

## Step 4 — Run and Verify

```bash
node index.js
```

Visit `http://localhost:3000` — click the sign-in link to authenticate, then view your profile at `/profile`.

## Troubleshooting

**Certificate error** — Set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` for local development (remove before deploying).

**`invalid_client`** — Double-check the Client ID and Client Secret.
