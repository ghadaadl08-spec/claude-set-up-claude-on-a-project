# claude-course-starter

A small Express REST API (users and a health check) with in-memory data, used as the Claude Code course starter project.

## Commands

- `npm run dev`: start the API with auto-reload on http://localhost:3000
- `npm test`: run all tests (Node's built-in test runner)
- `npm run lint`: run ESLint. CI runs lint, then tests; both must pass before a change is done.

## Architecture

- `server.js` builds the Express app, mounts each router at `/<resource>`, and exports `app`. It only calls `listen` when run directly (`require.main === module`) so tests can import `app` without opening a port. Keep that guard.
- `routes/`: one router file per resource (`users.js`, `health.js`). A new resource gets its own file and is mounted in `server.js`.
- `db/store.js`: the only data layer. In-memory, resets on every restart.
- `tests/`: `<resource>.test.js` files that hit the exported `app` through `supertest`.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules. ESLint is set to `sourceType: "script"` and rejects `import`.
- Write tests with `node:test`, `node:assert` and `supertest`, not Jest or Mocha.
- Routes never touch data directly: add a function to `db/store.js` and call it from the route.
- Return errors as `res.status(code).json({ error: "message" })`: 400 for missing or invalid input, 404 for not found. Return 201 for a successful create.
- Convert URL params with `Number(req.params.id)` before looking anything up; the store compares ids with `===`.
- Never read or edit `.env`. Document new config keys in `.env.example` with placeholder values.
