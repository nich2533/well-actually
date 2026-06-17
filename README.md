# well-actually

A starter layout for keeping Claude Code sharp as your codebase grows.

The premise, in one line: **the context window is a budget, not a backpack.** Everything you load costs you twice — once in space, once in attention. So instead of cramming everything into `CLAUDE.md`, this template splits context into three pieces that load only when they're relevant.

Full write-up: [Context Is a Budget, Not a Backpack](https://banyanbusinessoutcomes.com/attention/context-is-a-budget-not-a-backpack/)

## The structure

```
.
├── CLAUDE.md            # lean map: what lives where, nothing more
├── rules/               # how we do things — loaded by relevance
│   ├── database.md
│   ├── api-design.md
│   ├── testing.md
│   └── security.md
├── documentation/       # how the system works — read on demand
│   ├── auth-flow.md
│   ├── ingestion-pipeline.md
│   └── billing-webhooks.md
└── .claude/
    └── skills/
        ├── well-actually-get-started/
        │   └── SKILL.md  # seeds rules/ + documentation/ in one pass
        ├── well-actually-create-rules/
        │   └── SKILL.md  # splits a fat CLAUDE.md into rules/
        ├── well-actually-documentation-full/
        │   └── SKILL.md  # documents the whole repo from scratch
        └── well-actually-documentation-recent/
            └── SKILL.md  # updates docs from the last week of commits
```

- **`CLAUDE.md`** is a table of contents, not an encyclopedia. It points Claude at the right file and gets out of the way.
- **`rules/`** holds your standards, split by concern. Working on an endpoint pulls in `api-design.md`; touching the database pulls in `database.md`. A typo fix pulls in nothing.
- **`documentation/`** holds system knowledge — the non-obvious, expensive-to-reverse-engineer parts. Good docs are cached understanding.
- **`.claude/skills/`** is what keeps the layout from being a chore. Four skills:
  - **`well-actually-get-started`** seeds both trees at once — splits your `CLAUDE.md` into `rules/` and reads your code into `documentation/`, fanning out subagents so it's fast.
  - **`well-actually-create-rules`** does just the rules half: reads an existing `CLAUDE.md` and decomposes it into concern-scoped `rules/` files.
  - **`well-actually-documentation-full`** reads the whole repo and writes `documentation/` from scratch — one doc per subsystem, in parallel. Use it to seed, or to reset docs that have rotted too far.
  - **`well-actually-documentation-recent`** is the maintenance loop: it reads the last week of commits and updates only the affected docs.

## Quickstart

1. Clone this repo, or copy the layout into an existing project.
2. Open the folder in Claude Code.
3. Run **`well-actually-get-started`**. It reads your existing `CLAUDE.md` and code and seeds `rules/` and `documentation/` for you — replacing the placeholders with the real thing. (Starting from scratch with no `CLAUDE.md`? Skip this and fill the placeholder files in by hand; each ships with a short prompt explaining what belongs there.)
4. Review the proposed lean `CLAUDE.md` and approve it.
5. From then on, after each feature lands, run **`well-actually-documentation-recent`** and watch the affected docs update themselves. If the docs ever drift too far, **`well-actually-documentation-full`** rebuilds them from the whole repo.

## Install into an existing project

The Quickstart above assumes you're starting fresh. To add this to an app you **already have**, copy only the skills — *not* the placeholder `CLAUDE.md`, `rules/`, `documentation/`, or `src/`. Those are scaffolding for a new project; dropped onto an existing app they'd overwrite the very `CLAUDE.md` you're trying to clean up.

```bash
# from your app's root — create the folder first if you don't have it
mkdir -p .claude/skills
cp -r /path/to/well-actually/.claude/skills/well-actually-* .claude/skills/
```

```powershell
# PowerShell
New-Item -ItemType Directory -Force .claude\skills | Out-Null
Copy-Item -Recurse C:\path\to\well-actually\.claude\skills\well-actually-* .claude\skills\
```

This merges into any skills you already have — the `well-actually-*` names won't collide with typical project skills. Then, from your app:

1. Run **`well-actually-create-rules`** to decompose your existing `CLAUDE.md` into `rules/`. Review the proposed lean `CLAUDE.md` before accepting it — nothing is overwritten without your approval.
2. Run **`well-actually-documentation-full`** to generate `documentation/` from your code.
3. Maintain with **`well-actually-documentation-recent`** as you ship.

The skills create `rules/` and `documentation/` for you if those folders don't exist, and they *propose* changes to `CLAUDE.md` rather than overwriting it. (`well-actually-get-started` runs steps 1–2 together; on an existing app you can also just run them directly.)

## Make it yours

These files are deliberately generic — they're prompts, not prescriptions. Delete the rules you don't need, add the concerns you do, and keep every file short. The moment a rule file turns into a second encyclopedia, you're back to carrying the backpack.
