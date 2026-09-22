# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A minimal Express.js REST API (users + health check) with an in-memory data store. Used as a starter project for the Claude Code course — no real database, no auth.

## Commands

- `npm run dev` — start the API with auto-reload (`node --watch server.js`) on `http://localhost:3000`
- `npm test` — run the test suite (`node --test`, using `supertest` against the exported `app`)
- `npm run lint` — run ESLint

## Conventions

- One route file per resource under `routes/` (e.g. `users.js`, `health.js`), mounted in `server.js` via `app.use("/<resource>", ...)`.
- All data access goes through `db/store.js` — route handlers never touch the `users` array directly.
- Validate required fields in the route handler and return `400` with `{ error: "..." }` before calling into the store; return `404` with `{ error: "..." }` when a lookup misses.

## Architecture

- `server.js` — app entry point; wires up middleware and routes, only calls `app.listen` when run directly (so tests can `require("../server")` without opening a port).
- `routes/` — one file per resource, each exporting an `express.Router()`.
- `db/store.js` — in-memory store (resets on restart); the only place that owns the `users` data.
- `tests/` — `node:test` + `supertest`, one file per resource, importing `app` from `server.js`.

## Notes

- `.env` is git-ignored; copy `.env.example` to `.env` for local config (currently just `PORT`).
- `.claude/settings.local.json` is git-ignored — personal permission overrides stay local.
