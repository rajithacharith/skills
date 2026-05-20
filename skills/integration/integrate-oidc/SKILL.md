---
name: integrate-oidc
description: Integrate ThunderID using a generic OIDC library for frameworks without an official ThunderID SDK — Angular, SvelteKit, Python, Go, .NET, and others. Use when asked to "add ThunderID to Angular", "integrate ThunderID with SvelteKit", or when no official SDK covers the target framework or language.
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(bun:*), Bash(pip:*), Bash(go:*), Read, Write, Edit
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID — Generic OIDC Integration

Assumes ThunderID is running at `https://localhost:8090`. If not, run `/setup-thunderid` first.

ThunderID is a standard OpenID Connect provider. Its discovery document is at:

```
https://localhost:8090/.well-known/openid-configuration
```

OIDC endpoints:

| Endpoint | URL |
|----------|-----|
| Discovery | `https://localhost:8090/.well-known/openid-configuration` |
| Authorization | `https://localhost:8090/oauth2/authorize` |
| Token | `https://localhost:8090/oauth2/token` |
| Userinfo | `https://localhost:8090/oauth2/userinfo` |
| JWKS | `https://localhost:8090/oauth2/jwks` |
| End session | `https://localhost:8090/oidc/logout` |

## Step 1 — Register an Application

1. Open `https://localhost:8090/console` and sign in as `admin` / `admin`
2. Navigate to **Applications → New Application**
3. Fill in:
   - **Type**: Single Page Application for browser apps; Web Application for server-side
   - **Authorized Redirect URL**: your app's callback URL
4. Copy the **Client ID** (and **Client Secret** for server-side apps)

---

## Angular — `oidc-client-ts`

```bash
npm install oidc-client-ts
```

Create `src/auth.ts`:

```ts
import { UserManager, WebStorageStateStore } from 'oidc-client-ts'

export const userManager = new UserManager({
  authority: 'https://localhost:8090',
  client_id: '<your-client-id>',
  redirect_uri: `${window.location.origin}/callback`,
  post_logout_redirect_uri: `${window.location.origin}`,
  response_type: 'code',
  scope: 'openid profile email',
  userStore: new WebStorageStateStore({ store: window.localStorage }),
})

export const signIn = () => userManager.signinRedirect()
export const signOut = () => userManager.signoutRedirect()
export const getUser = () => userManager.getUser()
```

Callback handler at the `/callback` route:

```ts
import { userManager } from './auth'
await userManager.signinRedirectCallback()
window.location.replace('/')
```

Route guard:

```ts
import { inject } from '@angular/core'
import { CanActivateFn, Router } from '@angular/router'
import { getUser } from './auth'

export const authGuard: CanActivateFn = async () => {
  const user = await getUser()
  if (!user || user.expired) {
    inject(Router).navigate(['/'])
    return false
  }
  return true
}
```

---

## SvelteKit — Auth.js

```bash
npm install @auth/sveltekit
```

`src/auth.ts`:

```ts
import { SvelteKitAuth } from '@auth/sveltekit'

export const { handle, signIn, signOut } = SvelteKitAuth({
  providers: [
    {
      id: 'thunderid',
      name: 'ThunderID',
      type: 'oidc',
      issuer: 'https://localhost:8090',
      clientId: '<your-client-id>',
      clientSecret: '<your-client-secret>',
    },
  ],
})
```

`src/hooks.server.ts`:

```ts
export { handle } from './auth'
```

Protect a route in `+page.server.ts`:

```ts
import { redirect } from '@sveltejs/kit'
import type { PageServerLoad } from './$types'

export const load: PageServerLoad = async (event) => {
  const session = await event.locals.auth()
  if (!session) throw redirect(303, '/')
  return { session }
}
```

Sign-in / sign-out in a Svelte component:

```svelte
<script>
  import { signIn, signOut } from '@auth/sveltekit/client'
  import { page } from '$app/stores'
</script>

{#if $page.data.session}
  <button on:click={() => signOut()}>Sign Out</button>
  <p>Signed in as {$page.data.session.user?.email}</p>
{:else}
  <button on:click={() => signIn('thunderid')}>Sign In</button>
{/if}
```

---

## Python — `authlib`

```bash
pip install authlib httpx
```

```python
from authlib.integrations.httpx_client import AsyncOAuth2Client

client = AsyncOAuth2Client(
    client_id='<your-client-id>',
    client_secret='<your-client-secret>',
    redirect_uri='http://localhost:8000/callback',
)

# Discover endpoints
metadata = await client.load_server_metadata('https://localhost:8090/.well-known/openid-configuration')

# Start login
uri, state = client.create_authorization_url(metadata['authorization_endpoint'], scope='openid profile email')

# In the callback handler
token = await client.fetch_token(metadata['token_endpoint'], authorization_response=str(request.url))
userinfo = await client.get(metadata['userinfo_endpoint'])
```

---

## Go — `coreos/go-oidc`

```bash
go get github.com/coreos/go-oidc/v3/oidc golang.org/x/oauth2
```

```go
provider, err := oidc.NewProvider(ctx, "https://localhost:8090")

config := oauth2.Config{
    ClientID:     "<your-client-id>",
    ClientSecret: "<your-client-secret>",
    RedirectURL:  "http://localhost:8080/callback",
    Endpoint:     provider.Endpoint(),
    Scopes:       []string{oidc.ScopeOpenID, "profile", "email"},
}

// Start login
http.Redirect(w, r, config.AuthCodeURL(state), http.StatusFound)

// In /callback
token, err := config.Exchange(ctx, r.URL.Query().Get("code"))
idToken, err := provider.Verifier(&oidc.Config{ClientID: "<your-client-id>"}).Verify(ctx, token.Extra("id_token").(string))

var claims struct {
    Email string `json:"email"`
    Name  string `json:"name"`
}
idToken.Claims(&claims)
```

---

## Troubleshooting

**Certificate error** — For browsers: visit `https://localhost:8090` and accept the warning. For Node.js: add `NODE_TLS_REJECT_UNAUTHORIZED=0` to `.env.local` (dev only). For Python: pass `verify=False` to the HTTP client (dev only). For Go: configure a custom TLS client.

**CORS error** — Ensure the redirect URI registered in the ThunderID console exactly matches the one used in code, including trailing slashes.

**`invalid_client`** — Double-check the Client ID and Client Secret.
