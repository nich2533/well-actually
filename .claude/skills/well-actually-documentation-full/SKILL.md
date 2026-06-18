---
name: well-actually-documentation-full
description: Reads the entire repository and writes documentation/ from scratch — one doc per subsystem. Use for initial seeding of a project, or when the docs have rotted too far for an incremental pass to recover. For the steady-state loop that only syncs the last week of changes, use well-actually-documentation-recent instead. Dispatches one subagent per subsystem so the docs draft in parallel. Trigger phrases include "document the whole repo", "full documentation pass", "seed the docs", or running the skill directly.
---

# well-actually-documentation-full — document the whole repo from scratch

`documentation/` is cached understanding — the non-obvious, expensive-to-reverse-engineer parts of the system written down so nobody has to grep the codebase to recover them. This skill builds that cache from nothing: it reads the entire repo, finds the subsystems worth documenting, and writes one doc per subsystem in parallel.

Use it once, up front (or as a reset). The day-to-day after that is `well-actually-documentation-recent`.

## What this skill does

1. Survey the whole repository and identify the subsystems.
2. Dispatch one subagent per subsystem, in parallel, to read that subsystem and write its doc.
3. Reconcile the set and report coverage.

## What this skill does NOT do

- It does not document everything. A doc earns its place by capturing what's *non-obvious* — a flow, an invariant, a gotcha. A file whose behavior is self-evident from its name doesn't get a doc. Exhaustive ≠ useful; the goal is a small set of high-value docs, not one per file.
- It does not invent behavior. Each doc is grounded in code the subagent actually read. Where the code is unclear, the doc flags the gap rather than guessing.
- It does not touch `.claude/rules/`. Standards belong in the rules tree; this skill writes only `documentation/` (how the system *is*).

## Workflow

1. **Survey the repo.** Map the codebase before splitting it up:
   - List the top-level structure and the main source directories (`git ls-files` or a directory walk under `src/`).
   - Identify the subsystems — the natural units someone would need to understand independently: auth, the ingestion pipeline, billing/webhooks, the job queue, the public API, etc. Let them emerge from the structure; don't force a fixed list.
   - Produce a subsystem → source-paths map and show it to the user before fanning out. Aim for a handful of high-value docs, not one per folder.

2. **Dispatch subagents — one per subsystem, all in a single message so they run concurrently.** Give each subagent:
   - The subsystem name and the source paths that make it up — plus explicit permission to read adjacent files (config, schema, callers) when needed to verify how the code actually behaves. The path list is a starting point, not a fence.
   - The target path: `documentation/<subsystem>.md`.
   - The house format every doc follows (give it verbatim so the set comes back consistent):
     - An H1 title and a one-line `>` note marking this as system knowledge (*what is*, not *what should be*).
     - An `## Overview` — the flow or responsibility in a few numbered steps.
     - A `## The non-obvious parts` — the invariants, gotchas, ordering assumptions, and footguns that cost time to rediscover. **This is the highest-value section; spend the most effort here.**
     - A `## Where it lives` — the source paths a reader jumps to.
   - The instruction to **read the actual source** for its subsystem (not just filenames), **write the file directly** at `documentation/<subsystem>.md`, and return a one-line summary plus any place the code was too unclear to document confidently.
   - A grounding fence: document only what the code shows. Unclear behavior gets a `> TODO:` line naming what's unknown — never a guessed paragraph. **The code is the ground truth.** If the subsystem description you were handed (or the project's own docs) contradicts what the code does, document the actual behavior and flag the divergence in a `> TODO:` — don't bend the doc to match a stale claim.

   Each subagent owns a distinct file, so the concurrent writes never collide.

3. **Reconcile the set.** When the subagents return, verify every promised doc exists and follows the format. Check for overlap (two docs describing the same boundary — merge or cross-link), for standards that drifted in from `.claude/rules/` territory (move them out), and for subsystems that got missed.

4. **Replace the placeholders.** If the repo still has the template's placeholder docs, the real docs supersede them — remove any placeholder that no longer maps to a real subsystem so the tree reflects the actual system.

5. **Establish the sync marker.** Write `HEAD`'s SHA to `documentation/.last-synced` (`git rev-parse HEAD`). This pass just brought the docs in line with the whole repo, so HEAD is the point everything is now synced to — and it's the baseline `well-actually-documentation-recent` diffs from next time. Without it, the recent pass has nothing to anchor on and will send the user back here. Commit it alongside the new docs.

6. **Report.** List each doc written with its one-line summary, the subsystems deliberately left undocumented (and why — usually "self-evident"), anything flagged as too unclear to document, the sync marker you wrote, and the next step: from here on, run `well-actually-documentation-recent` to keep these current.

## Principle

The first full pass is the expensive one — it's where the cache gets built. Spend the effort on the non-obvious parts, because that's the understanding that's costly to rebuild later. Everything self-evident from the code is wasted ink; leave it out and the docs that remain are the ones worth trusting.
