---
name: error-responses
description: Use when adding or modifying an Express route handler or request validation in this project (files under routes/ or their tests). Defines the required 400/404 error response format and message wording. Do not use for anything else.
---

# Error responses in this project

## Format

Every error is JSON with a single `error` string field, returned with `return` so the handler stops:

```js
return res.status(400).json({ error: "name and email are required" });
```

No other fields, no plain-text bodies, no `res.sendStatus`.

## 400 — validation failed

- Validate in the route handler, **before** calling the store.
- Check required fields first, with one combined message: `"<field> and <field> are required"`
  - users: `"name and email are required"` (`!name || !email`)
  - products: `"name and price are required"` (`!name || typeof price !== "number"`; price is checked by type, so `0` is valid)
- Then check format rules with a separate, specific message, e.g. `"name must be at least 2 characters"` (non-string or `trim().length < 2`).
- Lowercase field names, no trailing period.

## 404 — lookup missed

- `GET /<resource>/:id`: `const id = Number(req.params.id)`, then if the store returns nothing, respond `{ error: "<Resource> not found" }` (capitalized singular: `"User not found"`, `"Product not found"`).
- A non-numeric id becomes `NaN`, misses in the store, and so also gives 404 (not 400).
- Unknown paths (e.g. `/health/unknown`) fall through to Express's default 404; don't add custom handling.

## Success codes (for contrast)

`200` for reads, `201` for successful `POST` creation.

## Tests

Each error path gets a test in `tests/<resource>.test.js` that asserts both `res.status` and the exact `res.body.error` text, using `supertest` against `app` from `server.js`.
