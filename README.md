# BuildTrack — Construction Management MVP

A presentation-ready construction operations prototype for contractors, site managers and clients. It focuses on dated site updates, milestone evidence, claims, client responses, material records, workforce visibility and payment tracking.

## What's included

- React + Vite frontend with responsive light/dark premium UI.
- Three role-based demo workspaces (role switching is for presentation only, not authentication).
- Projects, milestones, claims, payments, site updates, materials, workforce and issues.
- Floating BuildTrack AI assistant with project-aware prompts.
- Express API with health, state read/write and AI chat endpoints.
- PostgreSQL persistence when `DATABASE_URL` is configured; in-memory fallback for local API-only testing.
- Browser localStorage fallback if the API is unavailable.
- Render blueprint for the API + PostgreSQL and Vercel-ready frontend build.

## Architecture

```text
Browser (React/Vite)
  ├─ localStorage fallback
  └─ /api/* ───────────────► Express API (Render)
                                ├─ PostgreSQL (Render)
                                └─ OpenAI Responses API (optional)
```

The frontend only talks to `/api/state` and `/api/ai/chat`. The OpenAI key is read only by the backend and must never be added to a `VITE_*` variable.

## Run locally

Requirements: Node.js 20+.

Terminal 1 — API:

```bash
cd server
cp .env.example .env
npm install
npm run dev
```

The API listens on `http://localhost:3001`. Without `DATABASE_URL`, it uses in-memory state (which resets when the API restarts). Configure PostgreSQL for persistent local data.

Terminal 2 — frontend:

```bash
npm install
npm run dev
```

Open the Vite URL, normally `http://localhost:5173`. Vite proxies `/api` to `http://localhost:3001`.

## Configure the AI assistant

Add these variables to the **backend** environment (for local development, `server/.env`):

```env
OPENAI_API_KEY=your_secret_key
OPENAI_MODEL=gpt-5.5
```

Restart the API. Without an API key, the assistant uses a small deterministic demo helper for common project questions; it is not generative AI. The API key is never sent to the browser. API usage may incur charges under your OpenAI account.

## Deploy the backend to Render

1. Push this repository to GitHub.
2. In Render, create a Blueprint from the repository and select the root `render.yaml`.
3. Render provisions the `buildtrack-api` web service and `buildtrack-db` PostgreSQL database.
4. Set `API_ALLOWED_ORIGINS` to the exact Vercel frontend origin, e.g. `https://buildtrack-demo.vercel.app` (comma-separated if needed).
5. Add `OPENAI_API_KEY` as a secret if you want generative AI. Leave it unset for demo-helper mode.
6. Deploy and verify `https://YOUR-API.onrender.com/api/health` returns `ok: true` and `database: connected`.

The blueprint uses Render's private database connection and `PGSSL=false`. If you manually use an external PostgreSQL URL, set `PGSSL=true` if required by your provider.

## Deploy the frontend to Vercel

1. Import the same GitHub repository into Vercel.
2. Framework preset: Vite; root directory: repository root.
3. Build command: `npm run build`; output directory: `dist`.
4. Add environment variable `VITE_API_URL=https://YOUR-API.onrender.com`.
5. Deploy. Copy the deployed Vercel origin into the backend's `API_ALLOWED_ORIGINS` and redeploy/restart the API.

For local development, leave `VITE_API_URL` unset so Vite's `/api` proxy is used.

## API

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/health` | API, database and AI configuration status |
| GET | `/api/state` | Load the demo project state |
| PUT | `/api/state` | Validate and persist the demo project state |
| POST | `/api/ai/chat` | Project-aware AI assistant (or deterministic fallback) |

`PUT /api/state` expects `{ "state": { projects, milestones, claims, payments, updates, materials, workers, issues } }`.

## Important deployment and security limits

This package is **deployment-ready for a controlled presentation/demo**, not yet a production multi-tenant SaaS. The current MVP stores one shared demo workspace in one PostgreSQL row. There is no user login, tenant isolation, per-record authorization, audit log, object storage for actual photo files, or concurrent-edit conflict resolution. The role selector is a demo control and must not be treated as a security boundary. Do not put real client personal data, contracts, wage details or financial records into a public deployment until authentication and authorization are implemented.

The chat endpoint is rate-limited and the API key remains server-side, but production use should additionally add authenticated users, project-level access checks, stricter request validation, durable audit trails, backups, monitoring and privacy controls. AI output is advisory; it must not certify engineering safety, contractual entitlement, or payment approval.

## Before presenting

- Open all three roles and rehearse the same project workflow.
- Submit a sample daily update, submit a milestone claim, respond as the client and record a payment.
- Check `/api/health` before the presentation.
- Keep the OpenAI key and database credentials only in backend environment variables.
- Use demo data only.


## Assistant fallback
The assistant first tries the configured backend AI endpoint. If it is unavailable, it switches to a local demo assistant that answers common project, payment, claim, material, workforce, issue and site-update questions using the workspace data. This fallback does not require an API key.
