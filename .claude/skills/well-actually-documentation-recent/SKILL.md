---
name: well-actually-documentation-recent
description: Reads the last week of commits and updates the affected files in documentation/ to match. Use as the steady-state maintenance loop — run it after a feature lands or before a PR merges to keep system docs in sync with what just changed. For a from-scratch pass over the whole repo, use well-actually-documentation-full instead. Trigger phrases include "update the docs", "sync documentation", "docs are stale", or running the skill directly.
---

# well-actually-documentation-recent — sync docs with the last week of changes

`documentation/` describes how the system *actually works*. The moment code and docs drift, the docs become a trap — confidently wrong is worse than absent. This skill closes that gap on a tight loop: it reads the **last week of commits**, sees what changed, and updates only the affected docs.

## What this skill does

1. Pull the last week of commits and the diffs behind them.
2. Map each change to the doc it affects (or decide none is affected).
3. Update only the affected docs, preserving their structure and voice.
4. Report what changed and why — never edit silently.

## What this skill does NOT do

- It does not re-document the whole repo. It only touches what moved in the last week. For a full pass — new project, or docs that have rotted past the point a week of commits explains — use `well-actually-documentation-full`.
- It does not invent system behavior. If the diff doesn't show how something works, the doc says what's known and flags the gap — it does not guess.
- It does not touch `rules/`. Rules are how you *want* things done; this skill only maintains `documentation/` (how things *are*).

## Workflow

1. **Pull the window.** Get the last week's commits and their diffs:
   - `git log --since="1 week ago" --oneline` — the commits in scope. If this is empty, nothing changed in the last week; say so and stop.
   - Derive a baseline commit from that window rather than a reflog shorthand. `git "@{1 week ago}"` depends on local reflog and resolves unreliably on a fresh clone or in CI — don't use it. Instead, take the oldest commit in the window and diff its parent against `HEAD`:
     - `BASE=$(git log --since="1 week ago" --format=%H | tail -1)` — the oldest in-window commit.
     - `git diff "$BASE^" HEAD --stat` for the changed-file overview, then `git diff "$BASE^" HEAD -- <path>` per file that moved.
     - If `$BASE` is the repo's first commit (no parent), diff against the empty tree instead: `git diff 4b825dc642cb6eb9a060e54bf8d69288fbee4904 HEAD` (the well-known empty-tree hash, portable across platforms).
   - If the user named a tighter scope ("just the billing change"), honor it.
   - List the changed source files before going further.

2. **Map changes to docs — fan out.** Dispatch one subagent per doc in `documentation/`, in a single message so they run concurrently. Give each subagent its doc plus the week's diff, and ask: does anything in this window change what this doc describes? A change under `src/billing/handlers` affects `billing-webhooks.md`; a CSS-only change affects nothing here. Each subagent returns either "no change needed" (with why) or a concrete edit to its file.

3. **Verify before editing.** Each subagent confirms the doc's current description is now wrong or incomplete *against the diff* — not against memory. The "non-obvious parts" sections get the most scrutiny; those are the claims most likely to have silently rotted.

4. **Make the smallest true edit.** Update only the lines that drifted, preserving the doc's structure, headings, and voice. New non-obvious behavior (a new idempotency key, a new ordering assumption) goes into that doc's "non-obvious parts." Removed behavior gets deleted — no tombstones.

5. **Flag gaps, don't fill them.** If a change clearly needs documentation but the diff doesn't reveal the behavior, add a short `> TODO:` line naming exactly what's unknown, and surface it in the report. A flagged gap is honest; a guessed paragraph is the trap this skill exists to prevent.

6. **Report.** For each doc: updated (with a one-line summary), unchanged (with why), or flagged (with what's missing). If the week introduced a whole new subsystem with no doc at all, recommend running `well-actually-documentation-full` or creating the doc — don't create it unprompted unless asked.

## Principle

Good documentation is cached understanding. A cache that lies is worse than a cache miss. When unsure whether an edit makes the doc *more* true, prefer flagging the uncertainty to writing a confident sentence you can't back with the diff.
