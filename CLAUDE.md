# Project map

This file is a map, not a manual. It tells you where things live so the right context loads for the task in front of you — and nothing else does.

## Where things are

- `src/` — application code.
- `.claude/rules/` — how we do things. **You don't need to load these by hand.** Each rule file carries a `paths:` glob and Claude Code loads it automatically when you touch a matching file — `database.md` when you open a migration, `api-design.md` when you touch an endpoint, and nothing at all for a typo fix. A truly universal standard ("never log secrets") can live here in this file instead, so it's always in context.
- `documentation/` — how the system already works. These are *read on demand* — there's no auto-load, so this map is how you find the right one:
  - `documentation/auth-flow.md` — how a request becomes an authenticated, authorized caller.
  - `documentation/ingestion-pipeline.md` — how external data enters the system and becomes queryable.
  - `documentation/billing-webhooks.md` — how the payment provider's events drive billing state.

## How to use this

Pull in the one or two docs relevant to the change you're making — not all of them. The `.claude/rules/` files arrive on their own when they apply; the `documentation/` files you reach for from the list above.

When a feature lands or before a PR merges, run `well-actually-documentation-recent` to keep `documentation/` in sync with the code.

## Keep this file lean

If this map grows past a screen, something that belongs in `.claude/rules/` or `documentation/` has leaked into it. Move it back out. A standard that applies to a specific kind of file belongs in a path-scoped rule, not here.
