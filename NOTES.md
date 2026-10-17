# NOTES — `PUT /users/:id`

## Plan

Before any edits I asked for a read-only plan. It covered:

- **Files:** add `updateUser(id, { name, email })` to `db/store.js` and a `PUT /:id` route to `routes/users.js`. Leave `server.js` alone (it already mounts `express.json()` and `/users`) and leave `tests/update-user.test.js` alone.
- **Validation:** PUT replaces the whole user, so `name` and `email` are both required. Each must be a string that isn't empty after trimming. Anything else gets `400 { error }`. Only `name` and `email` are read from the body, so a client can't change `id` or add fields.
- **Not found:** `404 { error: "User not found" }`, the same as `GET /users/:id`. The id goes through `Number()` like it does in GET, so a non-numeric id is a 404.
- **Order:** validation runs before the lookup, so a bad body is a 400 even when the id doesn't exist.
- **Verification:** `npm test` and `npm run lint` after each commit, a few extra edge-case requests, then a review of the full diff.

Before approving I added some clarifications: create a feature branch rather than working on `main`, don't modify any test files, record the exact model name, give the store helper, the route and the notes separate commits, and stop before pushing or opening the PR. I didn't change the technical approach.

## Model

This was done with **Claude Opus 5.5** (model ID `claude-opus-5-5`) in Claude Code. I used the model already active in the session and didn't switch. The name and ID come from Claude Code's session environment information, not a guess. A smaller model such as Sonnet would probably have handled a change this size. Opus was kept because it was already active and the course asked for a careful plan and review.

## Commits

1. `feat(store): add updateUser helper`: the data-access layer, on its own.
2. `feat(users): add PUT /users/:id with validation and 404 handling`: the HTTP layer that uses it.
3. `docs: add NOTES.md for the update-user change`: this file.

The commits follow the layers of the app. The store helper works and can be reviewed without the route. The route commit is where the update-user tests go from red to green. The notes are kept out of the code commits. Lint passed after every commit, and the full suite passed at the end.

## Review

All the planned checks passed: `npm test` shows 9/9 passing (including the existing users and health tests), and `npm run lint` is clean. I reviewed the full diff against `main` and ran extra requests the provided tests don't cover:

| Request | Result |
|---|---|
| `PUT /users/abc` | 404 |
| empty, whitespace-only, or numeric `name`/`email` | 400 |
| missing field on an unknown id (`/users/9999`) | 400 (validation runs first) |
| no body / JSON array body | 400 |
| malformed JSON | 400 (Express's built-in error handler) |
| body containing `id` and `role` | 200; id unchanged, extra fields ignored |

The review found no bugs. It noted these minor behaviours, all of which POST and GET already share, so I left them alone to keep this change in scope:

- `name` and `email` are validated after trimming but stored as sent, so surrounding spaces are kept.
- Emails aren't checked for format or uniqueness.
- `Number()` accepts ids like `0x1` or `1e0`, which resolve to user 1, the same as in GET.
- The new PUT validation is stricter than POST, which only checks that the fields are truthy (so it accepts `name: 123`). Making them match would be a sensible follow-up.
