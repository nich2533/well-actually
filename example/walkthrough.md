# Walkthrough: one small feature, start to finish

A concrete look at what this layout buys you. We'll ship a tiny change and watch the
two halves of the system do their jobs: a path-scoped rule loading *itself* at the right
moment, and `documentation/` re-syncing from the diff afterward.

This is a narrated example, not a runnable app — the paths and file contents below stand
in for a real repo so you can see the mechanism without cloning anything.

## The change

Add an `is_active` flag to billing accounts so cancelled accounts stop being billed.
Three files move:

```
src/db/migrations/0042_add_is_active.sql   # new column, defaults true
src/billing/handlers/subscription.ts       # set is_active = false on subscription.canceled
src/billing/handlers/subscription.test.ts  # cancelled account is skipped on the next cycle
```

## What happens while you work — rules load themselves

You open `0042_add_is_active.sql` to write the migration. That path matches a glob in
`.claude/rules/database.md`:

```yaml
---
paths:
  - "**/migrations/**"
  - "**/*.sql"
---
```

So Claude Code pulls `database.md` into context **automatically, at that moment** — and
Claude reminds you migrations are forward-only and one logical change each, before you've
written a line. You never mentioned the rule. You never loaded it. The glob did.

Open the `.test.ts` file and `testing.md` arrives the same way. Open the handler and
`api-design.md` and `security.md` (both scoped to `src/billing/**`-ish paths) arrive.
Touch none of those and **nothing loads** — a README typo fix pulls in zero rule files.
That is the budget working: you pay context only for the concern in front of you.

> Contrast with the old "CLAUDE.md mentions `rules/database.md`" approach: a mention
> loads nothing on its own — the model has to *choose* to go read it. `paths:` makes it
> mechanical instead of hopeful.

## What happens after you commit — docs re-sync from the diff

You commit the three files. Then you run `well-actually-documentation-recent`.

It reads `documentation/.last-synced` (say it holds `a1b2c3d`), confirms that commit is an
ancestor of `HEAD`, and diffs `a1b2c3d..HEAD`. It sees `src/billing/handlers/` moved, which
maps to `documentation/billing-webhooks.md`. A subagent compares that doc against the diff
and finds the "non-obvious parts" section is now incomplete — cancellation has a new
side effect. It makes the smallest true edit:

```diff
  ## The non-obvious parts

+ - **Cancellation flips `is_active`.** A `subscription.canceled` event now sets
+   `is_active = false` on the account; the billing cycle skips inactive accounts.
+   Re-activation is a separate `subscription.resumed` event, not a field flip.
```

It leaves `ingestion-pipeline.md` and `auth-flow.md` untouched (the diff didn't reach them)
and reports *why* for each. Finally it advances the marker — `documentation/.last-synced`
now holds the new `HEAD` SHA — and stages that change in the same commit as the doc edit.
Run it again with no new commits and it says "nothing changed since the last sync" and stops.

## The payoff in one line

The rules showed up exactly when the files they govern were in play, and the docs stayed
true to the code without anyone re-reading the whole subsystem. That's the whole point:
**context arrives by relevance, and cached understanding doesn't rot.**
