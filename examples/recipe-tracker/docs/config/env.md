----
name: env
type: config
----

# env

Single env file at project root: `.env`. Copy from `.env.example` on first setup.

## Variables

- `DATABASE_URL` — SQLite path for V1. Default `sqlite:///./recipes.db`. V2 may switch to postgres.
- `PORT` — backend port. Default `8000`.
- `WEB_PORT` — Next.js port. Default `3000`.

No secrets in V1 — single-user local-only app. When auth lands in V2, add a pair md at `docs/backend/config/auth.md` and the matching `docs/ui/common/auth.md`.

## Pairs with

Nothing in V1. The V2 auth pair will reference this file for the session-secret env key.
