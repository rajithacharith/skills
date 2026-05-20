---
name: integrate-nuxt
description: Add ThunderID authentication to a Nuxt application using the official @thunderid/nuxt module. Use when asked to "integrate ThunderID into Nuxt", "add auth to my Nuxt app", or "connect ThunderID with Nuxt".
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Nuxt Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

1. Open `https://localhost:8090/console` and sign in as `admin` / `admin`
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: your app name
   - **Type**: Web Application
   - **Authorized Redirect URL**: `http://localhost:3000/callback`
4. Copy the **Client ID**, **Client Secret**, and generate a session secret (random 32+ char string)

### Via the API

First obtain a system API token from the ThunderID console, then:

```bash
curl -kL -X POST https://localhost:8090/applications \
  -H 'Authorization: Bearer <your-system-token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-nuxt-app",
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
npm install @thunderid/nuxt
```

## Step 3 — Configure Module

Edit `nuxt.config.ts`:

```ts
export default defineNuxtConfig({
  modules: ['@thunderid/nuxt'],
})
```

Wrap `app.vue`:

```vue
<template>
  <ThunderIDRoot><NuxtPage /></ThunderIDRoot>
</template>
```

Add to `.env.local`:

```env
NUXT_PUBLIC_THUNDERID_BASE_URL=https://localhost:8090
NUXT_PUBLIC_THUNDERID_CLIENT_ID=<your-client-id>
THUNDERID_CLIENT_SECRET=<your-client-secret>
THUNDERID_SESSION_SECRET=<random-string-at-least-32-chars>
```

## Step 4 — Add Auth UI

```vue
<script setup>
import { SignedIn, SignedOut, SignInButton, SignOutButton, Loading, User } from '@thunderid/nuxt'
</script>

<template>
  <Loading><div>Loading...</div></Loading>
  <SignedIn><SignOutButton>Sign Out</SignOutButton></SignedIn>
  <SignedOut><SignInButton>Sign In</SignInButton></SignedOut>
  <SignedIn>
    <User>
      <template #default="{ user }">
        <p>Welcome, {{ user.name || user.username }}!</p>
      </template>
    </User>
  </SignedIn>
</template>
```

## Step 5 — Run and Verify

```bash
npm run dev
```

Click **Sign In** — you should be redirected to `https://localhost:8090` and returned after login.

## Troubleshooting

**Certificate error** — Visit `https://localhost:8090` in your browser and accept the warning once.

**`invalid_client`** — Double-check the Client ID and Client Secret in `.env.local`.
