# Rewise — Design

Status: draft for approval. Date: 2026-10-28.

## 1. Intent and why

AI coding agents repeat mistakes because they forget everything between sessions.
Two real examples from the author:

- During a refactor the agent left the old code behind; the leftover code broke
  the feature being built.
- A feature changed, but the agent did not update the docs that described it.

Neither is a logic error. Both are "forgot the side effects" errors, and neither
shows up as a failing test. The agent must (a) understand the mistake when it is
pointed out, (b) fix it right then, and (c) not repeat it in future sessions.

This project is an **open-source, agent-agnostic "Rewise" kit**: a
protocol plus templates that any project can install, so its agent learns from
mistakes and keeps what it learns per project. It is a generalized version of
the system already running in a private project (`intrans-app`), plus two
additions: a trigger on user correction, and a pre-finish side-effects checklist.

## 2. Goals

1. Works with any AI coding agent (Claude Code, Codex, Cursor, others).
2. Per-project memory, stored as plain markdown inside the project, committable
   and reviewable by the team.
3. The agent reads relevant memory **before** working and records a lesson
   **after** a meaningful mistake.
4. Two record triggers: failed test/build/validation, and the user pointing out
   a mistake.
5. A short "before you say done" checklist that targets silent side-effect
   mistakes (stale code left behind, stale docs, stale references).
6. Installable by any agent from written instructions alone.

## 3. Non-goals

- No cross-project or global memory (decided: memory stays inside its project).
- No scripts, CLI, database, vector search, or dependencies.
- Not a replacement for reasoning. Memory never overrides the user's current
  explicit instruction.
- No automatic hooks tied to one tool.

## 4. Anti-goals (what counts as failure even if it technically works)

- **Memory full of junk.** Trivial, one-off, or obvious entries.
- **Duplicates.** Two entries that teach the same thing.
- Rules too vague to act on ("be careful with refactors").
- The agent following stale or wrong memory blindly.
- Project-specific details leaking into the shared templates.

The anti-junk/duplicate requirement is the author's top concern and drives the
mechanisms in section 6.

## 5. Constraints and decisions

Decided by the author:

- Reusable in any project; memory is project-local.
- Published as open source; audience is global.
- Agent-agnostic.
- Triggers: test/build failure **and** user-pointed mistakes.

Calls made by the designer (author may override):

- **Name:** `rewise` (chosen by the author).
- **Language:** kit files (SKILL.md, PROTOCOL.md, templates, README) in English.
  Memory entries created in a user's project follow that project's language.
- **Entry point:** `AGENTS.md` (cross-tool convention). If a project only has
  `CLAUDE.md`, append there instead. A short block points to `PROTOCOL.md`.
- **Format:** plain markdown only. Easiest to read, diff, and maintain; no
  dependency to audit.
- **Core vs wrapper:** the protocol lives in tool-neutral `PROTOCOL.md`;
  `SKILL.md` is a thin Claude Code wrapper pointing to it.

## 6. Design

### 6.1 Repository layout (the kit)

```
rewise/
├── README.md              what it is, install, how it works, example
├── LICENSE                MIT (designer's call)
├── SKILL.md               thin Claude Code wrapper -> PROTOCOL.md
├── PROTOCOL.md            tool-neutral lifecycle rules (the core)
├── SETUP.md               step-by-step install any agent can follow
└── templates/
    ├── AGENTS-block.md    block appended to project AGENTS.md/CLAUDE.md
    ├── index.md           empty memory index
    ├── mistake.md         mistake entry template
    ├── lesson.md          reusable-knowledge entry template
    └── decision.md        decision entry template
```

### 6.2 What gets installed in a user's project

```
<project>/
├── AGENTS.md              + short block (always-on trigger)
└── memory/
    ├── index.md           one line per entry
    ├── mistakes/
    ├── lessons/
    └── decisions/
```

`PROTOCOL.md` is copied into the project (e.g. `memory/PROTOCOL.md`) so the
rules travel with the repo and need no external lookup.

### 6.3 Lifecycle

1. **Before work:** read `memory/index.md`, read only relevant entries, follow
   them or state a technical reason for not following.
2. **During work:** do not reintroduce an approach already recorded as wrong.
3. **On failure signal** (failing test/build/validation, or the user points out
   a mistake):
   1. Understand the root cause first. Do not record before understanding.
   2. Fix it now, including leftovers (remove stale code, update docs).
   3. Confirm the fix at the layer that actually failed.
   4. Run the record gate (6.4).
4. **Before saying done:** run the side-effects checklist (6.5), then the record
   gate for anything new, then update the index.

### 6.4 Record gate (anti-junk and anti-duplicate)

An entry is written only if **all** pass:

1. **Recurrence test:** could this realistically happen again in a *different*
   task? If no, do not record.
2. **Actionable rule:** can the Prevention Rule be stated as a concrete check
   or action? If it can only be phrased as advice, do not record.
3. **Not trivial:** not a typo, syntax error, one-off outage, or something the
   compiler/linter already reports clearly.
4. **Not a duplicate:** search `index.md` and the relevant folder first. If a
   same-topic entry exists, **edit/extend it** instead of creating a new one.
   Merge related entries.
5. **Root cause known:** the entry states the real cause, not the symptom.

Hard limits that keep memory small:

- One line per entry in `index.md` (title + short hook, no prose).
- One topic per entry; target under ~60 lines.
- Prevention Rule is 1-3 sentences.
- Entries are named by topic (`stale-code-after-refactor.md`), never by date.
- If the gate says "do not record", say nothing and write nothing.

Maintenance: when reading an entry that is wrong, outdated, or contradicted by
current code, fix or delete it in the same task. Delete beats keep-just-in-case.

### 6.5 Side-effects checklist (before "done")

Run mentally, act on each "yes":

- Did this change replace or supersede existing code (functions, files, flags,
  config, routes)? Is the old version removed and are all references updated?
- Did behavior, API, config, or setup change? Are the docs that describe it
  updated (README, API docs, comments, examples, changelog if the project has one)?
- Are there tests, types, or fixtures that still describe the old behavior?
- Did I touch anything an existing memory entry warns about?

The checklist lives in `PROTOCOL.md`; projects may extend it. Items found
missing in a real incident are the main source of new checklist rules via the
normal record gate.

### 6.6 Entry templates

- **Mistake:** Context, Problem, Root Cause, Correction, Prevention Rule,
  optional Example.
- **Lesson:** Context, Knowledge, When it applies, optional Example.
- **Decision:** Context, Decision, Reason, Consequences.

### 6.7 Priority

Explicit current user instruction > current project requirements > project
architecture/decisions > relevant memory > general conventions. Memory is never
followed blindly; conflicts are investigated and the stale entry updated.

### 6.8 Install (`SETUP.md`)

An agent, given only `SETUP.md`, should:

1. Create `memory/` with `index.md`, `mistakes/`, `lessons/`, `decisions/`.
2. Copy `PROTOCOL.md` into `memory/`.
3. Append `templates/AGENTS-block.md` to `AGENTS.md` (or `CLAUDE.md` if that is
   all that exists; create `AGENTS.md` if neither exists, and tell the user).
4. Skip anything that already exists; never overwrite user content.
5. Report what was created.

## 7. Left to the builder

- Exact wording of README, SETUP, and templates.
- Exact `SKILL.md` frontmatter and trigger description for Claude Code.
- A small worked example in README (generic, no project-specific stack).
- Whether `AGENTS-block.md` stays under ~10 lines (it should).

## 8. Verification

No code to unit-test. Verify by dry run in a throwaway empty project:

1. Install using only `SETUP.md`; confirm structure and AGENTS block appear and
   nothing existing is overwritten.
2. Simulate the two author scenarios (leftover old code after refactor; docs not
   updated). Confirm the agent fixes it, passes the record gate, and writes one
   entry plus one index line.
3. Simulate a trivial typo error. Confirm **nothing** is recorded.
4. Repeat scenario 1 in a second task. Confirm the agent extends the existing
   entry instead of creating a duplicate.
5. Read the kit for leaked project-specific content.
