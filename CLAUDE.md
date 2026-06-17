# Project map

This file is a map, not a manual. It tells you where things live so the right context loads for the task in front of you — and nothing else does.

## Where things are

- `src/` — application code.
- `rules/` — how we do things. Read the file that matches your task:
  - `rules/database.md` — schema, migrations, query conventions.
  - `rules/api-design.md` — endpoint shape, status codes, versioning.
  - `rules/testing.md` — what we test, how, and what "done" means.
  - `rules/security.md` — secrets, input handling, authz boundaries.
- `documentation/` — how the system already works. Read on demand:
  - `documentation/auth-flow.md`
  - `documentation/ingestion-pipeline.md`
  - `documentation/billing-webhooks.md`

## How to use this

Pull in the one or two files relevant to the change you're making — not all of them. If you're fixing a label, you don't need the migration policy.

When a feature lands or before a PR merges, run `/documentation` to keep `documentation/` in sync with the code.

## Keep this file lean

If this map grows past a screen, something that belongs in `rules/` or `documentation/` has leaked into it. Move it back out.
