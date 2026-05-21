![ThunderID Skills](./assets/images/repo-banner.png)

## Install

### via Open Agent Skills CLI

```bash
npx skills add thunder-id/skills
```

### via Codex

```bash
codex plugin marketplace add thunder-id/skills
```

After adding the marketplace, restart Codex, open `/plugins`, select
**ThunderID Skills**, install and enable `thunder-id-skills`, then start a new thread.

### via Claude Code

```bash
/plugin marketplace add thunder-id/skills
```

## Skills

### Core Skills

| Skill | Purpose | When to Use |
| --- | --- | --- |
| `setup-thunderid` | Install and start the ThunderID server | New projects, initial setup |

### Integration Skills

| Skill | Framework / SDK | When to Use |
| --- | --- | --- |
| `integrate-nextjs` | `@thunderid/nextjs` | Next.js apps |
| `integrate-nuxt` | `@thunderid/nuxt` | Nuxt apps |
| `integrate-react` | `@thunderid/react` | React apps |
| `integrate-react-router` | `@thunderid/react` + `@thunderid/react-router` | React Router apps |
| `integrate-tanstack-router` | `@thunderid/react` + `@thunderid/tanstack-router` | TanStack Router apps |
| `integrate-vue` | `@thunderid/vue` | Vue apps |
| `integrate-express` | `@thunderid/express` | Express servers |
| `integrate-node` | `@thunderid/node` | Node.js / Fastify / Hono servers |
| `integrate-browser` | `@thunderid/browser` | Vanilla browser apps |
| `integrate-javascript` | `@thunderid/javascript` | Node.js, edge runtimes, custom integrations |
| `integrate-oidc` | Generic OIDC | Angular, SvelteKit, Python, Go, .NET |

## Usage

### Ask Your Agent

| You Say | Skill Used |
| --- | --- |
| "Set up ThunderID on my machine" | `setup-thunderid` |
| "Add ThunderID to my Next.js app" | `integrate-nextjs` |
| "Add ThunderID to my Nuxt app" | `integrate-nuxt` |
| "Integrate ThunderID into my React app" | `integrate-react` |
| "Add ThunderID to my React Router app" | `integrate-react-router` |
| "Add ThunderID to my TanStack Router app" | `integrate-tanstack-router` |
| "Add ThunderID to my Vue app" | `integrate-vue` |
| "Protect routes in my Express app" | `integrate-express` |
| "Add ThunderID to my Node.js / Fastify / Hono app" | `integrate-node` |
| "Add ThunderID to my vanilla JS app" | `integrate-browser` |
| "Integrate ThunderID without a framework" | `integrate-javascript` |
| "Add ThunderID to my Angular / SvelteKit / Python / Go app" | `integrate-oidc` |

## License

Licenses this source under the Apache License, Version 2.0 ([LICENSE](LICENSE)), You may not use this file except in compliance with the License.

---------------------------------------------------------------------------
(c) Copyright 2026 WSO2 LLC.
