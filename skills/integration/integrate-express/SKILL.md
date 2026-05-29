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

Ask the developer to create an application in ThunderID and share the **Client ID** and **Client Secret** before continuing.

Guide them through these steps:

1. Open `https://localhost:8090/console` and sign in (default: `admin` / `secret`)
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: their app name (e.g. `my-express-app`)
   - **Type**: Web Application
   - **Authorized Redirect URL**: `http://localhost:3000/login`
4. Click **Create** and copy the **Client ID** and **Client Secret** shown on the next screen

Once they paste both values, use them in all subsequent steps. Do **not** use placeholders — wait for the real values.

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install express cookie-parser @thunderid/express
```

## Step 3 — Add Middleware and Routes

Create `index.js`:

```js
const express = require('express');
const cookieParser = require('cookie-parser');
const { thunderID, handleSignIn, handleSignOut, protect } = require('@thunderid/express');

const app = express();
const port = 3000;

app.use(cookieParser());
app.use(express.json());

app.use(
  thunderID({
    baseUrl: 'https://localhost:8090',
    clientId: '<your-client-id>',
    clientSecret: '<your-client-secret>',
    afterSignInUrl: 'http://localhost:3000/login',
    afterSignOutUrl: 'http://localhost:3000/logout',
  }),
);

app.get('/', (_req, res) => {
  res.send('<a href="/protected">Go to protected page</a>');
});

app.get('/login', handleSignIn());
app.get('/logout', handleSignOut());

app.get(
  '/protected',
  protect((res) => res.redirect('/login')),
  (_req, res) => {
    res.send('You are signed in and can access this protected route.');
  },
);

app.get('/me', protect(), async (req, res) => {
  const user = await req.thunderIDAuth.getUserFromRequest(req);
  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

## Step 4 — Run and Verify

```bash
node index.js
```

Visit `http://localhost:3000/protected` — you should be redirected to `https://localhost:8090`. After login, you'll return to the protected route. Visit `http://localhost:3000/me` to inspect the signed-in user profile.

## Troubleshooting

**Certificate error** — Set `NODE_TLS_REJECT_UNAUTHORIZED=0` in `.env` for local development (remove before deploying).

**`invalid_client`** — Double-check the Client ID and Client Secret.
