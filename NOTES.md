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

## Wire Claude into your stack

**Server.** I connected the filesystem MCP server (@modelcontextprotocol/server-filesystem) at project scope in .mcp.json, limited to the project folder, with no credentials. It gives structured access (directory_tree, search_files, reading several files at once). The permission rule in .claude/settings.json allows only mcp__filesystem__read_text_file; the write, edit and move tools still ask for permission. On Windows the server is started through cmd /c npx.

**Skill.** .claude/skills/error-responses/SKILL.md captures how this project returns errors: JSON { error: "..." } sent with return, 400 for validation (required fields first, then format), 404 with "<Thing> not found", and tests that check both the status and the exact message. The description is narrow: it fires only when someone adds or changes an Express route or its validation. I confirmed it by asking for a new GET /orders/:id route without naming the skill; the answer used the project's error format.

**Command.** I added /test <file path> and /scaffold <type> <name> in .claude/commands/. /test is worth a shortcut because the same short command writes tests that match the project's style for any file (it produced tests for health, store and users). /scaffold uses $1 and $2, so a two-word name has to be wrapped in quotes.

**Hook.** A PostToolUse hook (it reacts after the edit, it does not prevent) with matcher Edit|Write runs "npx eslint --max-warnings 0 routes tests db server.js 1>&2 || exit 2". Exit code 2 sends the lint output back to Claude. I added --max-warnings 0 because an unused variable is only a warning in this ESLint config. I triggered it on purpose with an unused variable and saw the blocking error.

**Headless.** I ran: claude -p "List all Express routes in the routes folder: method and path. Change nothing." --allowedTools "Read,Glob,Grep". It is read-only: Edit, Write and Bash are not allowed, so it is safe to run unattended.
