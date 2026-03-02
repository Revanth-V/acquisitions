# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Acquisitions is a Node.js REST API built with Express 5 and ES modules (`"type": "module"`). It uses Neon (serverless Postgres) via Drizzle ORM, with JWT-based auth stored in httpOnly cookies.

## Commands

- **Dev server:** `npm run dev` (nodemon with `--legacy-watch`)
- **Lint:** `npm run lint` / `npm run lint:fix`
- **Format:** `npm run format` / `npm run format:check`
- **Generate migration:** `npm run db:generate` (reads schema from `src/models/*.js`)
- **Run migration:** `npm run db:migrate`
- **DB GUI:** `npm run db:studio`

There is no test runner configured yet. The eslint config includes a `tests/**/*.js` glob with jest-style globals, but no test framework is installed.

## Architecture

### Entrypoint and Server Setup

`src/index.js` loads dotenv then imports `src/server.js`, which imports the Express app from `src/app.js` and listens on `PORT`. The app is exported separately from the server to allow testing without starting a listener.

### Request Flow

Routes → Controllers → Services → Database (Drizzle)

- **Routes** (`src/routes/`) — define endpoints and wire them to controller functions. Mounted under `/api` prefixes in `app.js`.
- **Controllers** (`src/controllers/`) — validate input using Zod schemas, call services, set cookies/tokens, and send responses.
- **Services** (`src/services/`) — contain business logic and database operations via Drizzle.
- **Validations** (`src/validations/`) — Zod schemas used by controllers via `safeParse`. Validation errors are formatted by `src/utils/format.js`.

### Database

- **ORM:** Drizzle ORM with Neon serverless driver (`@neondatabase/serverless`).
- **Connection:** `src/config/database.js` exports `db` (Drizzle instance) and `sql` (raw Neon query function).
- **Schema:** Table definitions live in `src/models/*.js` using Drizzle's `pgTable` builder. `drizzle.config.js` points at this glob for migration generation.
- **Migrations:** Generated SQL files live in `drizzle/`. Run `db:generate` after schema changes, then `db:migrate` to apply.

### Path Aliases

The project uses Node.js subpath imports (defined in `package.json` `"imports"` field):
`#config/*`, `#controllers/*`, `#middleware/*`, `#models/*`, `#routes/*`, `#services/*`, `#utils/*`, `#validations/*` — all resolve to `./src/<folder>/*`. Always use these aliases in imports.

### Auth and Security

- JWT tokens are signed/verified via `src/utils/jwt.js` and stored in httpOnly cookies managed by `src/utils/cookies.js` (15-minute expiry, secure in production, sameSite strict).
- Passwords are hashed with bcrypt (salt rounds: 10) in the auth service.
- Express middleware stack: helmet, cors, cookie-parser, morgan (logging to winston).

### Logging

Winston logger (`src/config/logger.js`) writes JSON to `logs/error.lg` and `logs/combined.log`. Console transport is added in non-production environments.

## Code Style

- ESLint + Prettier enforced: 2-space indent, single quotes, semicolons, trailing commas (es5), unix line endings.
- Prefix unused function parameters with `_` (enforced by `no-unused-vars` rule).
- Use `const`/`let` only (no `var`), arrow callbacks, and object shorthand.
