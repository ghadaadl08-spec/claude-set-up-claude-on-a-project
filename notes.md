# Notes: How Claude Code is set up on this project

These notes explain the two files that set up Claude Code for this repo: `CLAUDE.md` and `.claude/settings.json`.

## CLAUDE.md

Claude Code loads `CLAUDE.md` at the start of every session. It tells Claude things about the project that it can't easily work out from the code, or that it would otherwise have to rediscover each time.

### Commands

The file lists the exact commands to run the app, the tests, a single test file, a filtered set of tests, and the linter.

- **Why:** Claude can run the right command straight away instead of guessing (for example `npx jest` or `npm start`) or reading `package.json` first.
- The note that CI runs `lint` and then `test` on Node 22 means Claude knows a change isn't finished until both pass.

### Architecture

- **`server.js` exports `app` and only calls `listen()` when run directly.** This is the main reason the tests work. Without this note, Claude might "fix" the server by calling `listen()` all the time, which would break tests that import the app.
- **One router per resource, mounted in `server.js`.** This shows Claude where new endpoints belong and how to add a new resource.
- **`db/store.js` is the only data-access layer.** Routes call store functions and never change data directly. This keeps that boundary in place when Claude adds features.
- **The store is shared across tests in the same process.** This prevents tests that assert exact record counts or IDs and then fail depending on which test ran first.

### Conventions

Each convention is there because Claude would plausibly get it wrong without being told:

| Convention | What it prevents |
|---|---|
| CommonJS, not ES modules | `import`/`export` syntax, which breaks under ESLint's `sourceType: "script"` |
| `node:test` + `node:assert` | Claude pulling in Jest or Mocha |
| Errors as `{ "error": "..." }`, 400 for bad input, 404 for not found, 201 for creates | Responses with inconsistent shapes or status codes |
| Convert params with `Number(req.params.id)` | String IDs that never match, because lookups use `===` |
| Unused args named `req`, `res`, `next`, `_` | Lint failures in CI |
| Never read or commit `.env` | Secrets leaking into the conversation or into git |

### What I left out on purpose

Anything Claude can find easily by reading the code (such as a file-by-file tour or every route) stays out. A short `CLAUDE.md` gets followed. A long one gets skimmed.

## .claude/settings.json (permissions)

```json
{
  "permissions": {
    "allow": ["Bash(npm test:*)"],
    "ask":   ["Bash(git push:*)"],
    "deny":  ["Read(./.env)", "Read(./**/.env)"]
  }
}
```

Claude checks rules in this order: **deny → ask → allow**. The most restrictive rule that matches wins.

### Allow: `npm test`

- Running tests is safe, has no side effects, and happens all the time. Asking for approval every time would just slow things down.
- The `:*` suffix is a prefix match, so `npm test -- --test-name-pattern="..."` is covered too.

### Ask: `git push`

- Pushing is outward-facing and hard to take back. It changes the shared remote and starts CI.
- `ask` instead of `deny`: I still want Claude to be able to push, but only after I've checked what's going out.

### Deny: reading `.env`

- `.env` holds real config and secrets. Anything Claude reads ends up in the conversation context.
- There are two rules: `./.env` covers the project root and `./**/.env` covers nested folders.
- **Limitation:** these are `Read` rules, so they block Claude's file-reading tool but not a shell command like `cat .env`. The `CLAUDE.md` instruction ("Never read or commit `.env`") adds a second layer, and a `Bash` deny rule or sandboxing could make this stricter.

### Shared vs. personal settings

`.claude/settings.json` is meant to be committed, so these rules apply to everyone on the project. Personal tweaks go in `.claude/settings.local.json`, which git ignores.
