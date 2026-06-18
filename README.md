# well-actually

A starter layout for keeping Claude Code sharp as your codebase grows.

The premise, in one line: **the context window is a budget, not a backpack.** Everything you load costs you twice — once in space, once in attention. So instead of cramming everything into `CLAUDE.md`, this template splits context into pieces that load only when they're relevant — and for the rules half, it uses Claude Code's own path-scoped loading so "only when relevant" is a *mechanism*, not a hope.

Full write-up: [Context Is a Budget, Not a Backpack](https://banyanbusinessoutcomes.com/attention/context-is-a-budget-not-a-backpack/)

## The structure

```
.
├── CLAUDE.md                 # lean map: orientation + pointers to documentation/, nothing more
├── .claude/
│   ├── rules/                # how we do things — each file auto-loads ONLY when its paths: match
│   │   ├── database.md        #   paths: migrations, *.sql, src/db/**
│   │   ├── api-design.md      #   paths: src/api/**, routes, handlers
│   │   ├── testing.md         #   paths: *.test.*, *.spec.*, tests/**
│   │   └── security.md        #   paths: auth, api, webhooks, middleware, env
│   └── skills/
│       ├── well-actually-get-started/         # seeds rules/ + documentation/ in one pass
│       ├── well-actually-create-rules/        # splits a fat CLAUDE.md into .claude/rules/
│       ├── well-actually-documentation-full/  # documents the whole repo from scratch
│       └── well-actually-documentation-recent/# syncs docs with commits since the last sync
├── documentation/            # how the system works — read on demand, pointed at by CLAUDE.md
│   ├── auth-flow.md
│   ├── ingestion-pipeline.md
│   ├── billing-webhooks.md
│   └── .last-synced          # commit SHA the docs were last synced to (created on first full pass)
└── example/
    └── walkthrough.md        # one small feature, start to finish — see the mechanism in action
```

### How loading actually works

This is the part most "split your CLAUDE.md" advice gets wrong, so it's worth being precise (confirmed against the [official memory docs](https://code.claude.com/docs/en/memory)):

- **`CLAUDE.md`** loads *in full, every session*. So it stays a table of contents, not an encyclopedia — orientation plus pointers to `documentation/`, and any standard so universal it should always be in context.
- **`.claude/rules/*.md`** is the load-bearing trick. Each rule file carries a `paths:` glob in its YAML frontmatter, and Claude Code loads it **automatically and only when Claude touches a file matching that glob**. Open a migration → `database.md` arrives. Fix a typo → nothing arrives. You don't reference these from `CLAUDE.md`; the glob does the work.
  - ⚠️ A rule file *without* a `paths:` block loads on every session — that's the backpack again. Every file here has one on purpose.
  - This beats the common alternatives: a markdown *mention* of a file loads nothing automatically (the model has to choose to read it), and an `@import` loads eagerly at startup (costs context every session). `paths:` is the only mechanism that's both automatic *and* relevance-gated.
- **`documentation/`** is *read on demand* — there's no auto-load, which is correct for reference material you reach for occasionally. `CLAUDE.md` lists these so the right one is easy to find. Good docs are cached understanding; the `.last-synced` marker is how the maintenance skill keeps that cache honest.

See [`example/walkthrough.md`](example/walkthrough.md) for a concrete before/after: a path-scoped rule loading itself mid-task, and `documentation/` re-syncing from a diff afterward.

### The skills

The skills are what keep the layout from being a chore:

- **`well-actually-get-started`** seeds both trees at once — splits your `CLAUDE.md` into `.claude/rules/` and reads your code into `documentation/`, fanning out subagents so it's fast.
- **`well-actually-create-rules`** does just the rules half: reads an existing `CLAUDE.md` and decomposes it into concern-scoped `.claude/rules/` files, each with the right `paths:` glob.
- **`well-actually-documentation-full`** reads the whole repo and writes `documentation/` from scratch — one doc per subsystem, in parallel. Use it to seed, or to reset docs that have rotted too far. It also writes the first `.last-synced` marker.
- **`well-actually-documentation-recent`** is the maintenance loop: it reads every commit *since the last sync* (tracked by `.last-synced`, not a calendar window) and updates only the affected docs, then advances the marker.

## Quickstart

1. Clone this repo, or copy the layout into an existing project.
2. Open the folder in Claude Code.
3. Run **`well-actually-get-started`**. It reads your existing `CLAUDE.md` and code and seeds `.claude/rules/` and `documentation/` for you — replacing the placeholders with the real thing. (Starting from scratch with no `CLAUDE.md`? Skip this and fill the placeholder files in by hand; each ships with a short prompt explaining what belongs there, and the rule files show the `paths:` format to copy.)
4. Review the proposed lean `CLAUDE.md` and approve it.
5. From then on, after each feature lands, run **`well-actually-documentation-recent`** and watch the affected docs update themselves. If the docs ever drift too far, **`well-actually-documentation-full`** rebuilds them from the whole repo.

## Install into an existing project

The Quickstart assumes you're starting fresh. To add this to an app you **already have**, you only need the skills — the placeholder `CLAUDE.md`, `.claude/rules/`, `documentation/`, `src/`, and `example/` are scaffolding for a new project, and the skills regenerate the real versions from your code. Copy just the skills:

```bash
# from your app's root
mkdir -p .claude/skills
cp -r /path/to/well-actually/.claude/skills/well-actually-* .claude/skills/
```

```powershell
# PowerShell
New-Item -ItemType Directory -Force .claude\skills | Out-Null
Copy-Item -Recurse C:\path\to\well-actually\.claude\skills\well-actually-* .claude\skills\
```

The `well-actually-*` names won't collide with typical project skills, so this merges cleanly. Then, from your app:

1. Run **`well-actually-create-rules`** to decompose your existing `CLAUDE.md` into `.claude/rules/`. It proposes a slimmed `CLAUDE.md` (written to `CLAUDE.md.proposed` for you to diff) and never overwrites the original without approval.
2. Run **`well-actually-documentation-full`** to generate `documentation/` from your code and write the first `.last-synced` marker.
3. Maintain with **`well-actually-documentation-recent`** as you ship.

The skills create `.claude/rules/` and `documentation/` if those folders don't exist, so there's nothing to set up by hand. (`well-actually-get-started` runs steps 1–2 together; on an existing app you can also just run them directly.)

## Make it yours

These files are deliberately generic — they're prompts, not prescriptions. Delete the rules you don't need, add the concerns you do, and **tune each rule's `paths:` glob to your actual layout** — that's the one thing worth getting right, since it decides when the rule fires. Keep every file short. The moment a rule file turns into a second encyclopedia, you're back to carrying the backpack.
