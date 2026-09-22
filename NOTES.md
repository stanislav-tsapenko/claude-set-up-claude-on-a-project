# NOTES

## CLAUDE.md

**What's in it:** a one-line project description, the three commands I actually run (`npm run dev`, `npm test`, `npm run lint`), two conventions that aren't obvious from a single file read (one-route-file-per-resource, and "all data access goes through `db/store.js`"), and a short architecture section describing `server.js` / `routes/` / `db/store.js` / `tests/`.

**What I left out, and why:**
- Anything derivable from opening the file it describes (e.g. the exact shape of the `User` object, or that `health.js` has one route) — Claude can read the code itself.
- Any mention of `.env` contents — there's nothing sensitive in `.env.example` here, but I kept the habit of not restating config values in `CLAUDE.md`.
- Git workflow / PR process — that's already in this repo's `README.md` and would just go stale if repeated here.

## Permissions (`.claude/settings.json`)

- **Allow:** `Bash(npm test:*)` and `Bash(npm run lint:*)` — both are safe, side-effect-free, and I run them constantly; no reason to be prompted every time.
- **Deny:** `Read(./.env)` — even though this starter's `.env` only holds `PORT`, the habit matters more than this specific file: if a real `DATABASE_URL` or API key ever lands there, Claude should never be able to read it into context (and from there potentially into a commit message, a pasted error, or a suggestion). Also denied `Bash(git push --force:*)` — a force-push can silently discard commits (mine or a teammate's) on a shared branch; there's no legitimate reason for Claude to do this without me typing it myself.
- **Ask:** `Bash(git push:*)` — pushing is visible to others (opens/updates a PR), so I want a confirmation prompt each time rather than blanket-allowing it, but it's common enough that a hard deny would be annoying.

**What could go wrong without the deny rule:** without `Read(./.env)` denied, an agent debugging a config issue could read `.env`, and once a secret is in context it can leak into a suggestion, a commit message, or a paste into an issue/PR description. Without the `git push --force` deny, an "let me clean this up" moment could overwrite a colleague's unpushed work with no easy recovery.

## Verification

- `/memory` shows `CLAUDE.md` loaded from the project root.
- `/permissions` shows the allow/ask/deny rules from `.claude/settings.json`.
- Could not run `npm install` / `npm test` / `npm run lint` in this sandbox (`node`/`npm` are not installed here), so the commands in `CLAUDE.md` are verified by reading `package.json`'s `scripts` block rather than by executing them.
