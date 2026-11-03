# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express (Node.js) REST API for users, with an in-memory data store. It's the starter project for a Claude Code course.

## Commands

- `npm run dev` — start the API on http://localhost:3000 (restarts on file changes via `node --watch`; set `PORT` to change it)
- `npm test` — run all tests with the built-in Node test runner (`node --test`)
- `node --test tests/users.test.js` — run a single test file
- `node --test --test-name-pattern="POST /users"` — run tests whose name matches
- `npm run lint` — ESLint (`eslint:recommended`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` and then `npm test` on Node 22, so both have to pass.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`), and **exports `app`**. It only calls `listen()` when run directly (`require.main === module`), so tests can import the app without opening a port.
- `routes/<resource>.js` holds one `express.Router()` per resource. Mount new routers in `server.js`.
- `db/store.js` is the only data-access layer: a module-level array that resets when the process restarts. Routes call store functions and never change data directly.
- `tests/` uses `node:test` + `node:assert` + `supertest` against the exported `app`. The store is shared across tests in the same process, so don't assert exact record counts or IDs that an earlier test could change.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules. ESLint is set to `sourceType: "script"`.
- Use `node:test` and `node:assert` for tests, not Jest or Mocha.
- Send errors as JSON `{ "error": "<message>" }` with the right status code (400 for bad input, 404 when something isn't found). Return successful creates with 201.
- Convert route params yourself (e.g. `Number(req.params.id)`). Store IDs are numbers and lookups use `===`.
- Unused-variable lint warnings skip arguments named `req`, `res`, `next` or `_`. Use those names for handler arguments you don't need.
- Real config goes in `.env` (see `.env.example`). Never read or commit `.env`.
