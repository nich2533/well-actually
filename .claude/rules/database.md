---
paths:
  - "**/migrations/**"
  - "**/*.sql"
  - "src/db/**"
  - "src/**/models/**"
---

# Database rules

> Auto-loads only when you touch a migration, a model, or SQL — that's what the `paths:` globs above do. Tune them to your layout. (For how the data actually flows, see `documentation/`, not here.)

Replace the examples below with your project's real conventions. Keep it to the rules that have actually bitten you — a rules file earns its place by preventing a specific mistake, not by being thorough.

## Migrations

- Every schema change ships as a migration. No editing the schema by hand.
- Migrations are forward-only and reversible. If a migration can't be reversed cleanly, write the down path explicitly.
- One logical change per migration. Don't bundle a column rename with a data backfill.

## Queries

- No raw string interpolation into SQL. Use parameterized queries or the query builder.
- Reads that can grow unbounded need a limit and an index to back the filter.

## Naming

- Tables are plural snake_case (`billing_accounts`). Columns are singular snake_case.
- Timestamps end in `_at` (`created_at`). Booleans read as a question (`is_active`).

## When in doubt

If a change affects how data is written or migrated, run `well-actually-documentation-recent` so `documentation/` reflects the new shape.
