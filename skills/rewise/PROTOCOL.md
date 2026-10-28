# Rewise Protocol

Rules for an AI coding agent that learns from its mistakes in this project.
Memory lives in `memory/` and is plain markdown. These rules are tool-neutral.

Rewise is not a replacement for reasoning. The goal is not to remember
everything; it is to remember the few experiences that stop the same mistake
from happening again.

## 1. Layout

```text
memory/
├── PROTOCOL.md   this file
├── index.md      one line per entry
├── mistakes/     what went wrong, why, and the rule that prevents it
├── lessons/      reusable technical knowledge
└── decisions/    important choices and the reason behind them
```

Always-on project rules (the short block in `AGENTS.md` / `CLAUDE.md`) must not
be duplicated here. Long root-cause detail lives in `memory/`; the always-on
file only points to it.

## 2. Before work

1. Read `memory/index.md`.
2. Pick the entries relevant to the current task.
3. Read **only those** entries before changing anything.
4. Follow them, or state a concrete technical reason for not following one.

Do not read every file in `memory/`.

## 3. During work

- Do not reintroduce an approach already recorded as wrong.
- If existing code conflicts with a memory rule, investigate before changing it.
- Prefer consistency with recorded decisions unless there is a strong technical
  reason not to.

## 4. Failure signals

A failure signal is either:

- a failing test, build, lint, typecheck, or validation, or
- the user pointing out a mistake (for example: "you left the old code behind",
  "the docs are out of date").

When one occurs:

1. **Understand the root cause first.** Do not record anything before you
   understand what actually went wrong and why.
2. **Fix it now**, including leftovers: remove stale code, update stale docs,
   fix stale references.
3. **Confirm the fix at the layer that failed** (rerun the test, rebuild, or
   re-check the doc), not at a neighbouring layer.
4. Run the record gate (section 5).

## 5. Record gate

Write a memory entry only if **all** of these hold:

1. **Recurrence:** this could realistically happen again in a *different* task.
2. **Actionable:** the Prevention Rule can be stated as a concrete check or
   action. If it can only be phrased as advice ("be careful"), do not record.
3. **Non-trivial:** not a typo, syntax error, one-off outage, or something a
   compiler/linter already reports clearly.
4. **Not a duplicate:** search `memory/index.md` and the relevant folder first.
   If an entry on the same topic exists, **extend or correct that entry**
   instead of creating a new one. Merge related entries.
5. **Root cause known:** the entry states the real cause, not the symptom.

If any check fails, write nothing and say nothing about memory.

### Size limits

- `index.md`: one line per entry (title plus a short hook). No prose.
- One topic per entry; aim for under ~60 lines.
- Prevention Rule: 1-3 sentences.
- Name files by topic (`stale-code-after-refactor.md`), never by date or task.

### Maintenance

If you read an entry that is wrong, outdated, or contradicted by current code,
fix or delete it in the same task. Deleting beats keeping "just in case".

## 6. Writing an entry

1. Pick the folder: `mistakes/`, `lessons/`, or `decisions/`.
2. Use the structure below for the entry type.
3. Add one line to `memory/index.md` in the right category.

Mistake:

```md
# <Short title>

## Context
## Problem
## Root Cause
## Correction
## Prevention Rule
## Example   (optional)
```

Lesson: Context, Knowledge, When it applies, Example (optional).
Decision: Context, Decision, Reason, Consequences.

## 7. Before saying a task is done

Run this checklist and act on every "yes":

- [ ] Did this change replace or supersede existing code (functions, files,
      flags, config, routes)? Is the old version removed and every reference
      updated?
- [ ] Did behavior, an API, config, or setup change? Are the docs that
      describe it updated (README, API docs, comments, examples, changelog)?
- [ ] Do any tests, types, or fixtures still describe the old behavior?
- [ ] Did I touch anything a memory entry warns about, and did I follow it?

Then:

1. Run the relevant tests, typecheck, lint, or build.
2. Verify the result matches what the user asked for.
3. Run the record gate for anything new. Update `memory/index.md` if an entry
   was added, merged, or deleted.
4. Only then report the task as done.

Projects may extend this checklist. A new checklist item must itself pass the
record gate (a real incident, likely to recur).

## 8. Priority

```text
Explicit current user instruction
        ↓
Current project requirements
        ↓
Project architecture / decisions
        ↓
Relevant memory
        ↓
General conventions
```

Memory never overrides the user's current explicit instruction. Do not follow an
entry blindly: if it looks wrong or outdated, check the real situation, do the
technically correct thing, and update the entry.
