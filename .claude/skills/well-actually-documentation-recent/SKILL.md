---
name: well-actually-documentation-recent
description: Reads every commit since the last documentation sync (tracked by a committed marker file) and updates the affected files in documentation/ to match. Use as the steady-state maintenance loop — run it after a feature lands or before a PR merges to keep system docs in sync with what just changed. For a from-scratch pass over the whole repo, use well-actually-documentation-full instead. Trigger phrases include "update the docs", "sync documentation", "docs are stale", or running the skill directly.
---

# well-actually-documentation-recent — sync docs with the last week of changes

`documentation/` describes how the system *actually works*. The moment code and docs drift, the docs become a trap — confidently wrong is worse than absent. This skill closes that gap on a tight loop: it reads **every commit since the docs were last synced** — not a fixed calendar window — sees what changed, and updates only the affected docs. The sync point is a committed marker, so two runs in a day don't reprocess the same commits and a quiet week doesn't let changes fall out of a window.

## What this skill does

1. Read the marker, then pull every commit since it and the diffs behind them.
2. Map each change to the doc it affects (or decide none is affected).
3. Update only the affected docs, preserving their structure and voice.
4. Report what changed and why — never edit silently.

## What this skill does NOT do

- It does not re-document the whole repo. It only touches what moved since the last sync. For a full pass — new project, or docs that have rotted past the point the diff explains — use `well-actually-documentation-full`.
- It does not invent system behavior. If the diff doesn't show how something works, the doc says what's known and flags the gap — it does not guess.
- It does not touch `.claude/rules/`. Rules are how you *want* things done; this skill only maintains `documentation/` (how things *are*).

## Workflow

1. **Find the sync point, then pull everything since it.** Docs are synced *up to a commit*, not *for a calendar period* — so the window is "everything since the last successful sync," tracked by a marker file rather than wall-clock time:
   - The marker lives at `documentation/.last-synced` and holds the commit SHA the docs were last brought in line with. It is committed alongside the doc edits, so the team, CI, and a fresh clone all share one honest sync point — no dependence on local reflog or `--since` dates.
   - **Marker exists and is reachable from HEAD** — verify with `git merge-base --is-ancestor "$(cat documentation/.last-synced)" HEAD`. That SHA is your `BASE`. If `BASE` already equals `HEAD`, nothing has changed since the last sync — say so and stop.
   - **Marker missing** (first run): there's no prior sync to diff from. Recommend `well-actually-documentation-full` to seed the docs — that pass establishes the first marker. Don't reconstruct a window from a guessed date.
   - **Marker exists but is NOT an ancestor of HEAD** (history was rebased or squashed, or it came from another branch): don't trust it. Tell the user the marker is stale and offer either a full pass (`well-actually-documentation-full`) or an explicit base they name; don't silently diff from a bad point.
   - With a valid `BASE`: `git log "$BASE"..HEAD --oneline` for the commits in scope, `git diff "$BASE" HEAD --stat` for the changed-file overview, then `git diff "$BASE" HEAD -- <path>` per file that moved.
   - If the user named a tighter scope ("just the billing change"), honor it for *which docs you touch*, but still advance the marker only once the docs are actually in sync.
   - List the changed source files before going further.

2. **Map changes to docs — fan out.** Dispatch one subagent per doc in `documentation/`, in a single message so they run concurrently. Give each subagent its doc plus the week's diff, and ask: does anything in this window change what this doc describes? A change under `src/billing/handlers` affects `billing-webhooks.md`; a CSS-only change affects nothing here. Each subagent returns either "no change needed" (with why) or a concrete edit to its file.

3. **Verify before editing.** Each subagent confirms the doc's current description is now wrong or incomplete *against the diff* — not against memory. The "non-obvious parts" sections get the most scrutiny; those are the claims most likely to have silently rotted.

4. **Make the smallest true edit.** Update only the lines that drifted, preserving the doc's structure, headings, and voice. New non-obvious behavior (a new idempotency key, a new ordering assumption) goes into that doc's "non-obvious parts." Removed behavior gets deleted — no tombstones.

5. **Flag gaps, don't fill them.** If a change clearly needs documentation but the diff doesn't reveal the behavior, add a short `> TODO:` line naming exactly what's unknown, and surface it in the report. A flagged gap is honest; a guessed paragraph is the trap this skill exists to prevent.

6. **Advance the marker — but only as far as you actually synced.** Once the doc edits are made, write `HEAD`'s SHA to `documentation/.last-synced` (`git rev-parse HEAD`) so the next run starts where this one ended. Two rules keep the marker honest:
   - Only advance to the commit the docs now reflect. If you flagged gaps with `> TODO:` rather than fully documenting a change, the docs are still in sync *as a known-incomplete record up to HEAD* — advancing is correct, because the TODO is the record of what's owed. But if you deliberately skipped commits (user scoped you to "just billing"), do **not** jump the marker past the commits you didn't address — leave it where those changes still register as pending.
   - The marker change is part of the same commit as the doc edits, never its own commit. Propose it alongside them; don't commit on the user's behalf unless asked.

7. **Report.** For each doc: updated (with a one-line summary), unchanged (with why), or flagged (with what's missing). State the old and new marker SHAs so the advance is visible. If the window introduced a whole new subsystem with no doc at all, recommend running `well-actually-documentation-full` or creating the doc — don't create it unprompted unless asked.

## Principle

Good documentation is cached understanding. A cache that lies is worse than a cache miss. When unsure whether an edit makes the doc *more* true, prefer flagging the uncertainty to writing a confident sentence you can't back with the diff.
