---
name: setup-thunderid
description: Install and start the ThunderID server. Use when asked to "set up ThunderID", "install ThunderID", "run ThunderID", or "start ThunderID server".
license: Apache-2.0
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(curl:*), Bash(docker:*)
metadata:
  author: thunderid
  version: 0.1.0
---

# ThunderID Server Setup

Ask the developer which option they prefer, then run the command.

## Option A — npx (recommended)

```bash
npx thunderid
```

Downloads the latest ThunderID release for the current OS/arch, runs first-time setup, and starts the server — no manual steps required.

- Server starts at **https://localhost:8090**
- Console at **https://localhost:8090/console** — default credentials: `admin` / `admin`
- During setup, note the `[INFO] Sample App ID: <id>` line — you will need this Client ID when integrating an app

## Option B — Docker Compose

```bash
curl -o docker-compose.yml https://raw.githubusercontent.com/thunder-id/thunderid/v0.39.0/install/quick-start/docker-compose.yml
docker compose up
```

Wait for `ThunderID started on https://localhost:8090` in the logs. On first run the setup container may take 30–60 seconds to initialize the database.

- Server starts at **https://localhost:8090**
- Console at **https://localhost:8090/console** — default credentials: `admin` / `admin`
- Note the `[INFO] Sample App ID: <id>` from the setup container output

## Troubleshooting

**`docker compose up` hangs** — Run `docker compose logs setup` to check database initialization progress.

**Certificate warnings** — ThunderID uses a self-signed HTTPS certificate on `localhost:8090`. Visit `https://localhost:8090` in your browser and accept the warning before running your app.

## What's Next

Choose the integration skill for your framework:

| Framework | Skill |
|-----------|-------|
| Next.js | `/integrate-nextjs` |
| Nuxt | `/integrate-nuxt` |
| React | `/integrate-react` |
| React + React Router | `/integrate-react-router` |
| React + TanStack Router | `/integrate-tanstack-router` |
| Vue | `/integrate-vue` |
| Express | `/integrate-express` |
| Node.js (other) | `/integrate-node` |
| Browser / vanilla JS | `/integrate-browser` |
| Universal JS | `/integrate-javascript` |
| Angular, Svelte, Python, Go, etc. | `/integrate-oidc` |
