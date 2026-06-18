---
name: well-actually-get-started
description: One-shot initial seeding for a project adopting the budget-not-backpack layout. Runs well-actually-create-rules (CLAUDE.md → .claude/rules/) and well-actually-documentation-full (code → documentation/) together, fanning out subagents so both trees seed in parallel. Use right after dropping this template into an existing codebase, or when .claude/rules/ and documentation/ are still placeholders. Trigger phrases include "get started", "seed the project", "initialize the docs and rules", or running well-actually-get-started.
---

# well-actually-get-started — seed .claude/rules/ and documentation/ from an existing codebase

The template ships with placeholder rules and docs. This skill replaces them with the real thing in one pass: it decomposes your `CLAUDE.md` into path-scoped `.claude/rules/` files and reads your code into `documentation/` — at the same time, because the two halves don't depend on each other.

## What this skill does

1. Confirm there's something to seed from.
2. Kick off both seeding jobs concurrently as subagents — the rules half and the docs half.
3. Wait for both, reconcile any overlap.
4. Write the slimmed `CLAUDE.md` map as a reviewable proposal, and give one combined report.

## What this skill does NOT do

- It does not re-implement `well-actually-create-rules` or `well-actually-documentation-full`. Those two skills are the source of truth for *how* each tree is seeded; this skill orchestrates them and owns only the parallelism and the combined report. If you change how rules or docs are generated, change it there, not here.
- It does not overwrite without approval. Anything destructive (replacing `CLAUDE.md`, overwriting non-placeholder files) is proposed first.

## Workflow

1. **Pre-flight.** Confirm the inputs exist:
   - A `CLAUDE.md` or convention files for the rules half. (None → skip that half and say so.)
   - Source code under `src/` (or wherever the project lives) for the docs half. (Empty → skip that half and say so.)
   If both are missing, there's nothing to seed — stop and tell the user what to add.

2. **Fan out both halves at once.** In a single message, dispatch two independent lines of work so they run concurrently:

   - **Rules subagent(s).** Run the `well-actually-create-rules` workflow: gather CLAUDE.md + config, cluster into concerns, and — because that workflow itself fans out one subagent per concern — this branch expands into several parallel drafters. Each writes its own `.claude/rules/<concern>.md`, complete with the `paths:` frontmatter that makes it auto-load only on matching files.
   - **Docs subagent(s).** Run the `well-actually-documentation-full` workflow: survey the codebase, identify the subsystems, and write each `documentation/<subsystem>.md` from scratch. One subagent per subsystem, in parallel.

   Every subagent owns a distinct output file, so the concurrent writes never collide. Send all dispatches in one message — don't await one branch before starting the other.

3. **Reconcile overlap.** When both halves return, check the seam between them: a standard that drifted into a doc ("we always validate at the boundary") belongs in `.claude/rules/security.md`; a piece of system behavior that landed in a rule ("the webhook retries on non-200") belongs in `documentation/billing-webhooks.md`. Move misfiled content to the correct tree. This is the one step that *must* happen after the barrier — it needs both halves' output.

4. **Write the lean CLAUDE.md — the seed isn't done until this exists.** Seeding `.claude/rules/` and `documentation/` without slimming `CLAUDE.md` leaves you carrying the backpack *and* the budget. So produce the actual file, every run:
   - Take the map the rules half drafted and confirm it also points at the freshly created `documentation/` docs. Remember the map does *not* list the `.claude/rules/` files individually — those auto-load by path; the map's job is the `documentation/` pointers and any universal standard.
   - **Write it to `CLAUDE.md.proposed`** next to the original — a real artifact, not a chat suggestion.
   - Show the user a diff of `CLAUDE.md` → `CLAUDE.md.proposed` and ask for approval.
   - **On approval, replace the original** (`CLAUDE.md.proposed` → `CLAUDE.md`) and remove the `.proposed` file. On decline, leave both for hand-editing.
   - Never overwrite `CLAUDE.md` in place without approval — but never finish the seed without having written the `.proposed` file.

5. **Combined report.** One summary covering:
   - `.claude/rules/` files created (with one-line summaries and their `paths:` globs).
   - `documentation/` files created (with one-line summaries).
   - Anything reconciled across the seam.
   - Anything flagged or skipped (ungrounded rules, subsystems too unclear to document, missing inputs).
   - Next step: review the proposed `CLAUDE.md`, then run `well-actually-documentation-recent` after the next feature lands to keep it all current.

## Why subagents

Seeding is embarrassingly parallel: each rule file and each doc file is independent. Drafting them one at a time wastes the structure. Fan out, let each subagent own one file, and the whole initial seed finishes in roughly the time of the slowest single file instead of the sum of all of them.

## Principle

This is a one-time accelerator, not the maintenance loop. Once both trees are seeded and the lean `CLAUDE.md` is in place, the steady state is small and incremental: edit the relevant `.claude/rules/` file by hand, and run `well-actually-documentation-recent` when code changes. Don't reach for `well-actually-get-started` again unless you're seeding a fresh project.
