# Notes

## The plan

The approved plan was to add a `PUT /users/:id` endpoint following the existing route patterns exactly: a `store.updateUser(id, { name, email })` helper in `db/store.js` (mirroring `getUserById`/`createUser`), and a route handler in `routes/users.js` that validates `name`/`email` are present (400 if not, checked before the not-found check), calls the store helper, and returns 404 if the user doesn't exist or 200 with the updated user otherwise. I didn't edit the plan before approving it — it matched what the tests in `tests/update-user.test.js` required, and reused the file's existing conventions closely enough that there wasn't anything to push back on.

## Model choice

Sonnet 5. The task is a small, well-scoped CRUD endpoint with a precise test contract (exact status codes, and validation has to run before the not-found check to satisfy the 400 test on a non-existent-but-unvalidated body). That called for a model that reliably gets edge-case ordering right without needing the extra cost/latency of a bigger model — Opus would have been overkill for a change this size.

## Commit split

Two commits: one adding `updateUser` to `db/store.js`, one adding the `PUT /:id` route in `routes/users.js`. Split along the store/route boundary that the rest of the file already follows (each existing route has a matching store function), so each commit is a single reviewable, self-contained unit — the store commit stands alone as "new data-layer capability," and the route commit is "wire it up to HTTP."

## What the review caught

I ran `/code-review` (medium effort) on the diff before pushing. It checked correctness (including the `Number(req.params.id)` → `NaN` → 404 path for non-numeric ids, confirming it matches the existing `GET /:id` behavior rather than being a regression), reuse, simplification, and convention-fit. It flagged one candidate — the `if (!name || !email)` validation block is duplicated between the new `PUT` handler and the existing `POST` handler — but judged it below the bar for a change worth making (two occurrences, three lines, in a small demo-scale file). No findings survived; nothing needed fixing.
