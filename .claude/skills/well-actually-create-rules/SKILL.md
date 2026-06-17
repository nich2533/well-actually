---
name: well-actually-create-rules
description: Reads an existing CLAUDE.md (and other convention sources) and decomposes it into concern-scoped files under rules/. Use when migrating a monolithic CLAUDE.md to the budget-not-backpack layout, or seeding rules/ for the first time. Dispatches one subagent per concern so the files draft in parallel. Trigger phrases include "create rules", "split my CLAUDE.md", "infer rules from CLAUDE.md", or running well-actually-create-rules.
---

# well-actually-create-rules — turn a fat CLAUDE.md into a lean rules/ tree

A monolithic `CLAUDE.md` is the backpack: every standard you've ever written, read on every task. This skill unpacks it — it reads the existing context, clusters the standards by concern, and emits one short `rules/<concern>.md` per concern, so each task pulls in only what it needs.

## What this skill does

1. Gather the source conventions (CLAUDE.md and the obvious config-as-convention files).
2. Cluster them into concerns — the natural seams (database, api, testing, security, style, …).
3. Dispatch one subagent per concern, in parallel, to draft that concern's rule file.
4. Write a slimmed `CLAUDE.md` — a map pointing at the new files instead of containing them — as a reviewable proposal file.
5. Report what was extracted, what was left behind, and what couldn't be sourced.

## What this skill does NOT do

- It does not invent standards. A rule appears only if it's grounded in the source CLAUDE.md, a config file, or an explicit pattern in the code. No grounding → it gets flagged, not written.
- It does not touch `documentation/`. Rules are how you *want* things done; that's a different tree. (Seeding both at once is `well-actually-get-started`.)
- It does not delete the original CLAUDE.md. The slimmed version is *proposed* — the user approves before anything is overwritten.

## Workflow

1. **Gather sources.** Read `CLAUDE.md` (the source of most rules). Then scan for convention-bearing config that should inform the rules rather than be restated: linter/formatter configs, `tsconfig`, CI files, `.editorconfig`, test config, any existing `rules/` or `CONTRIBUTING.md`. List what you found before going further. If there's no `CLAUDE.md` and no convention files, say so and stop — there's nothing to infer from.

2. **Cluster into concerns.** Read the gathered material and group the standards by concern. Don't force a fixed list — let the seams emerge from the content. Common ones: `database`, `api-design`, `testing`, `security`, `style`, `git-workflow`, `accessibility`. Aim for 3–8 files; if you have one rule for a concern, fold it into the closest neighbor rather than spawning a one-line file. Produce an explicit concern → source-lines map and show it to the user.

   While clustering, watch for **sources that contradict each other** — a CLAUDE.md that disagrees with itself, or with a config or an existing rules file. Don't silently pick a winner. Surface the conflict to the user with both sides quoted and let them resolve it; a rule built on a guessed resolution is a rule that lies.

3. **Dispatch subagents — one per concern, all in a single message so they run concurrently.** Give each subagent:
   - The concern name and the exact slices of source text that belong to it.
   - The target path: `rules/<concern>.md`.
   - The house format every rule file follows (give it verbatim so the files come back consistent):
     - An H1 title.
     - A `> Load this when …` one-liner naming the task that should pull the file in.
     - Short, single-concern sections. Behavior, not prose. Each rule earns its place by preventing a specific mistake.
     - A closing `## When in doubt` line.
   - The instruction to **write the file directly** at `rules/<concern>.md` and return a one-line summary of what it captured plus anything in its slice it deliberately dropped.
   - A scope fence: stay within the provided slices; if the slice implies a rule but doesn't state it, flag it in the return rather than inventing it.

   Each subagent owns a distinct file, so parallel writes don't collide.

4. **Collect and sanity-check.** Once the subagents return, verify every promised file exists and follows the format. Re-read each briefly for contradictions across files (two rules disagreeing) and for anything that's actually system knowledge masquerading as a rule — flag those for the `documentation/` tree (`well-actually-documentation-full`) instead of keeping them here.

5. **Write the lean CLAUDE.md — don't just describe it.** This step is the whole point of the skill; do not skip it or leave it as prose. Produce the actual slimmed file:
   - Draft the replacement: a map — the project's one-paragraph orientation, then a "where things are" list pointing at each new `rules/<concern>.md` (and the `documentation/` tree if it exists), and nothing that now lives in those files.
   - **Write it to `CLAUDE.md.proposed`** next to the original. This is a real artifact the user can open, not a suggestion in the chat.
   - Show the user a diff of `CLAUDE.md` → `CLAUDE.md.proposed` (what's leaving, what's becoming a pointer) and ask for approval.
   - **On approval, replace the original** (`CLAUDE.md.proposed` → `CLAUDE.md`) and delete the `.proposed` file. If the user declines, leave both files in place so they can edit the proposal by hand.
   - Never overwrite `CLAUDE.md` in place without the approval step — but always produce the `.proposed` file, every run, so "the CLAUDE.md got slimmed" is a concrete deliverable rather than an intention.

6. **Report.** List each file written with its one-line summary, the proposed CLAUDE.md diff, anything dropped or flagged (ungrounded would-be rules, misfiled system knowledge), and the recommended next step (`well-actually-documentation-full` to seed the docs tree, or `well-actually-get-started` if both trees are still empty).

## Principle

The goal isn't to relocate every sentence — it's to make each task load only its concern. If a rule wouldn't change what Claude does for *some specific task*, it's noise; drop it rather than file it. A lean wrong rule is still a cost.
