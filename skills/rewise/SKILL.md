---
name: rewise
description: Project memory that makes the agent learn from its mistakes. Use at the start of any coding task in a project, when a test/build fails, or when the user points out a mistake (stale code left after a refactor, docs not updated). Installs itself into the project on first use.
---

# Rewise

Rewise gives a project a small, curated memory of past mistakes so the agent
avoids repeating them. The rules are tool-neutral and live in `PROTOCOL.md`
next to this file; this file only says when to use them.

## Step 0: bootstrap the project (first use only)

Check the project root for `memory/PROTOCOL.md`.

- **Missing:** follow `SETUP.md` (in this skill's directory) now, without asking.
  It creates `memory/`, copies the protocol, and adds a short `## Rewise` block
  to the project's `AGENTS.md` (or `CLAUDE.md`) so every agent reads the rules in
  future sessions even without this skill installed. It never overwrites
  existing files. Tell the user in one line what was created, then continue
  with their task.
- **Present:** skip to the next section.

## Use in a project that has Rewise

Follow `memory/PROTOCOL.md` (the project copy). In short:

1. Before work: read `memory/index.md`, then only the relevant entries.
2. On a failing test/build, or when the user points out a mistake: find the
   root cause, fix it including leftovers, confirm at the failing layer, then
   apply the record gate.
3. Before saying done: run the side-effects checklist (stale code, stale docs,
   stale tests, memory warnings).

## Non-negotiable

- Never create a memory entry unless it passes the record gate in
  `PROTOCOL.md` (recurs, actionable, non-trivial, not a duplicate, root cause
  known). If it does not pass, write and say nothing.
- Extend an existing entry instead of creating a duplicate.
- Memory never overrides the user's current explicit instruction.
