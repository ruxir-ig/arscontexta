# T3 Stack Platform — arscontexta Setup

This guide sets up arscontexta memory infrastructure for a **T3 Stack** project
(`create-t3-app` — Next.js + tRPC + Tailwind + Prisma + TypeScript).
Covers **Codex**, **OpenCode**, and **Cursor** since those are the primary agents used.
Claude Code hooks are not the focus here.

---

## What this gives you

- A `CONTEXT.md` file at repo root auto-loaded by each agent as project memory
- Atomic decision notes in `decisions/` (architecture, schema, router patterns, bug causes)
- Module maps (MOCs) in `maps/` for quick orientation across tRPC routers, Prisma models, features
- A `scratch/` inbox for unprocessed spikes and todos
- Consistent vocabulary across sessions so any agent picks up where the last left off

---

## File layout

```
your-t3-project/
├── CONTEXT.md          ← master context file (commit this)
├── decisions/          ← atomic decision notes
│   ├── architecture/
│   ├── schema/
│   ├── routers/
│   └── bugs/
├── maps/               ← module maps (MOCs)
│   ├── architecture.md
│   ├── trpc-routers.md
│   ├── prisma-schema.md
│   ├── features.md
│   └── env-config.md
└── scratch/            ← unprocessed inbox (gitignore optional)
```

---

## Step 1 — Create `CONTEXT.md`

Place this at your repo root. Fill in the bracketed sections:

```markdown
# Project Context

## Stack
- Framework: Next.js (App Router / Pages Router — pick one)
- API: tRPC v11
- ORM: Prisma
- Styling: Tailwind CSS v4
- Auth: NextAuth.js / Clerk / (your choice)
- DB: PostgreSQL / PlanetScale / SQLite
- Deploy: Vercel / Railway / (your choice)

## Architecture
<!-- One paragraph: what does this app do, who uses it, what is the core data model -->

## Active tRPC Routers
<!-- list routers and their purpose, e.g. userRouter, postRouter -->

## Prisma Models (key ones)
<!-- list models + brief description -->

## Env Variables
<!-- non-secret keys only, e.g. DATABASE_URL, NEXTAUTH_URL -->

## Current Focus
<!-- what feature / bug is being worked on RIGHT NOW -->

## Open Questions
<!-- things that haven't been decided yet -->

## Decision Log
<!-- link to decisions/ notes, newest first -->
```

---

## Step 2 — Create starter module maps

Create `maps/trpc-routers.md`:
```markdown
# tRPC Routers Map

## appRouter composition
<!-- list sub-routers here -->

## [routerName]
- file: `src/server/api/routers/[name].ts`
- procedures: <!-- list procedures -->
- depends on: <!-- models, services -->
```

Create `maps/prisma-schema.md`:
```markdown
# Prisma Schema Map

## Models
<!-- one section per model -->

## [ModelName]
- table: `[table_name]`
- key fields: <!-- field: type -->
- relations: <!-- other models -->
- last changed: <!-- date + what changed -->
```

---

## Step 3 — Agent-specific setup

### Codex (OpenAI)

Codex reads context from files you explicitly reference in your prompt or system message.
Add this to your Codex task prompt template:

```
Before starting, read CONTEXT.md at the repo root for project architecture,
active routers, and current focus. After completing work, append a one-line
decision note to the relevant section of CONTEXT.md.
```

For persistent memory across tasks, keep `CONTEXT.md` committed and updated.
Codex does not have session hooks, so the update instruction in the prompt is the mechanism.

### OpenCode

OpenCode supports a project-level system prompt via `.opencode/system.md` (or your config).
Create `.opencode/system.md`:

```markdown
# Project Memory

At the start of every session:
1. Read `CONTEXT.md` — understand the stack, active routers, current focus.
2. Read relevant `maps/*.md` for the area you are working in.
3. After completing a significant change, update `CONTEXT.md` > Current Focus
   and append a decision note in `decisions/[area]/YYYY-MM-DD-[topic].md`.

## Decision note format
```
---
date: YYYY-MM-DD
area: [architecture|schema|router|bug]
tags: [nextjs|trpc|prisma|auth|env]
---
# [Decision title as a claim — what was decided]

## Context
<!-- why this decision was needed -->

## Decision
<!-- what was decided and how -->

## Consequences
<!-- what this changes, what debt it creates -->
```
```

### Cursor

Cursor loads `.cursorrules` (legacy) or `.cursor/rules/*.mdc` (current) as persistent context.

Create `.cursor/rules/arscontexta.mdc`:

````markdown
---
description: Project memory and context management
alwaysApply: true
---

# Project Memory — arscontexta T3 Stack

## On session start
- Read `CONTEXT.md` for stack overview, active routers, current focus.
- Read `maps/trpc-routers.md` before touching any tRPC code.
- Read `maps/prisma-schema.md` before touching any Prisma model.

## On completing work
- Update `CONTEXT.md` > Current Focus if the task focus has shifted.
- Write a decision note to `decisions/[area]/YYYY-MM-DD-[topic].md` when you:
  - Add or change a Prisma model
  - Add or refactor a tRPC router/procedure
  - Make an architectural choice
  - Resolve a non-trivial bug

## Decision note format
```
---
date: YYYY-MM-DD
area: [architecture|schema|router|bug]
tags: []
---
# [Title as a claim]

## Context
## Decision  
## Consequences
```

## T3 Stack conventions to remember
- tRPC procedures: `publicProcedure` vs `protectedProcedure` (check `src/server/api/trpc.ts`)
- Prisma client: imported from `~/server/db`
- env vars validated in `src/env.js` — always add new vars there
- Tailwind config: `tailwind.config.ts`
- App Router: layouts in `app/`, server components by default
````

---

## Step 4 — Gitignore

Add to `.gitignore` if you want scratch to stay local:
```
# arscontexta scratch inbox
scratch/
```

Commit everything else (`CONTEXT.md`, `decisions/`, `maps/`, agent rule files).

---

## Workflow loop

```
Start session
  └─ agent reads CONTEXT.md + relevant maps/
        └─ does the work
              └─ updates CONTEXT.md > Current Focus
                    └─ writes decision note if significant change
                          └─ commits
```

That's it. No CLI, no plugin install — just markdown files and agent rules.
