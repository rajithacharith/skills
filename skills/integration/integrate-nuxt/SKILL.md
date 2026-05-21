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
   - **Authorized Redirect URL**: `http://localhost:3000/api/auth/callback`
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
        "redirectUris": ["http://localhost:3000/api/auth/callback"],
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

## Step 3 — Register the Module

Edit `nuxt.config.ts`:

```ts
export default defineNuxtConfig({
  modules: ['@thunderid/nuxt'],
})
```

The module auto-registers `/api/auth/callback` as a Nitro server route — no manual callback route needed. It also auto-imports all components and composables.

## Step 4 — Set Environment Variables

Create `.env`:

```env
NUXT_PUBLIC_THUNDERID_BASE_URL=https://localhost:8090
NUXT_PUBLIC_THUNDERID_CLIENT_ID=<your-client-id>
THUNDERID_CLIENT_SECRET=<your-client-secret>
THUNDERID_SESSION_SECRET=<random-string-at-least-32-chars>
```

Generate `THUNDERID_SESSION_SECRET` with `openssl rand -base64 32`. `THUNDERID_CLIENT_SECRET` and `THUNDERID_SESSION_SECRET` have no `NUXT_PUBLIC_` prefix — Nuxt keeps them server-side only.

## Step 5 — Wrap App with ThunderIDRoot

Edit `app.vue`:

```vue
<template>
  <ThunderIDRoot>
    <NuxtPage />
  </ThunderIDRoot>
</template>
```

## Step 6 — Add Auth UI

All components and composables are auto-imported. Create `pages/index.vue`:

```vue
<template>
  <main>
    <header>
      <h1>ThunderID Auth Demo</h1>
      <SignedIn>
        <SignOutButton>Sign Out</SignOutButton>
      </SignedIn>
      <SignedOut>
        <SignInButton>Sign In</SignInButton>
      </SignedOut>
    </header>
    <section>
      <SignedIn>
        <User>
          <template #default="{ user }">
            <p>Welcome, {{ user.name || user.username }}!</p>
          </template>
        </User>
      </SignedIn>
    </section>
  </main>
</template>
```

## Step 7 — Protect a Page

```vue
<script setup>
definePageMeta({ middleware: ['thunderIDMiddleware'] });
</script>
```

Unauthenticated users are automatically redirected to the sign-in page.

## Step 8 — Run and Verify

```bash
npm run dev
```

Click **Sign In** — you should be redirected to `https://localhost:8090` and returned after login.

## Troubleshooting

**Certificate error** — Visit `https://localhost:8090` in your browser and accept the warning once.

**`invalid_client`** — Double-check the Client ID and Client Secret in `.env`.
