# Notes: setting up Claude Code on this project

## What's in CLAUDE.md, and what I left out

I kept `CLAUDE.md` to what Claude can't easily work out from the code, or would get wrong without being told. It has a one-line description, the three commands (`dev`, `test`, `lint`) plus the fact that CI runs lint and then tests, and a short architecture note: `server.js` mounts one router per resource from `routes/`, and all data goes through `db/store.js`. I also called out the `require.main === module` guard, because it looks removable but is what lets the tests import `app` without opening a port.

Each convention is written as "do X, not Y" and tied to something real in the repo. The project uses CommonJS because ESLint is set to `sourceType: "script"`. Tests use `node:test` with `supertest`, not Jest. Routes call the store instead of touching data. Errors use the `{ error }` shape with the right status code. URL ids are converted with `Number()` because the store compares with `===`. Claude must stay out of `.env`.

I deliberately left out a list of endpoints and file contents, because Claude can read `routes/` itself and a copy would go stale. I left out install and setup steps, which are in the README and don't help with code changes. I left out course instructions and one-off tasks, because `CLAUDE.md` loads in every session, so anything temporary keeps costing space. I left out anything from `.env`, because secrets don't belong in a committed file.

## Permission rules, and what the deny rules prevent

- **Allow:** `npm test` and `npm run lint`. Both only read code and report results, and Claude runs them after almost every change, so approving them each time would just be noise.
- **Ask:** `git push`, because pushing makes work visible to others. `npm install`, because it downloads packages and changes the lockfile.
- **Deny:** reading or editing `.env`, and `git push --force` / `git push -f`.

Without the `.env` deny rule, Claude could read the file while exploring the project or debugging config. Any real secrets in it, such as a database URL with a password, would then end up in the conversation and be sent off my machine. Without the force-push deny rule, one mistaken command could overwrite commits on the remote with no undo. I listed both `--force` and `-f` because rules match the start of the command as text, so one spelling doesn't cover the other.

One limit I'm aware of: `Read(./.env)` blocks Claude's Read tool, not every possible shell command. The rules are a guardrail, and the real protection is keeping real secrets out of the project folder.

## Verification

<!-- Fill this in after checking in a fresh Claude session -->
- `/memory`: CLAUDE.md shows as loaded:
- `/permissions`: allow / ask / deny rules are listed:
- Asked "How do I run the tests here?" and Claude answered:
