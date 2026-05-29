---
name: integrate-vue
description: Add ThunderID authentication to a Vue application using the official @thunderid/vue SDK. Use when asked to "integrate ThunderID into Vue", "add auth to my Vue app", or "connect ThunderID with Vue".
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Vue Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

## Step 1 — Register an Application

Ask the developer to create an application in ThunderID and share the **Client ID** before continuing.

Guide them through these steps:

1. Open `https://localhost:8090/console` and sign in (default: `admin` / `secret`)
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Name**: their app name (e.g. `my-vue-app`)
   - **Type**: Single Page Application
   - **Authorized Redirect URL**: `http://localhost:5173`
4. Click **Create** and copy the **Client ID** shown on the next screen

Once they paste the Client ID, use it in all subsequent steps. Do **not** use a placeholder — wait for the real value.

## Step 2 — Install

Detect the package manager from lockfiles: `pnpm-lock.yaml` → `pnpm add`, `yarn.lock` → `yarn add`, `bun.lockb` → `bun add`, else `npm install`.

```bash
npm install @thunderid/vue
```

## Step 3 — Register Plugin and Wrap with Provider

**`src/main.js`**

```js
import { createApp } from 'vue'
import { ThunderIDPlugin } from '@thunderid/vue'
import App from './App.vue'
import './style.css'

const app = createApp(App)
app.use(ThunderIDPlugin)
app.mount('#app')
```

**`src/App.vue`**

```vue
<template>
  <ThunderIDProvider client-id="<your-client-id>" base-url="https://localhost:8090">
    <RouterView />
  </ThunderIDProvider>
</template>
```

## Step 4 — Add Auth UI

```vue
<script setup>
import { SignedIn, SignedOut, SignInButton, SignOutButton, Loading, User } from '@thunderid/vue'
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

**Composables fail outside `setup()`** — `useThunderID()` and `useUser()` use Vue's `inject` and must be called inside a component's `setup()`.
