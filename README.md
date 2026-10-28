# Rewise

**Make your AI coding agent learn from its mistakes, per project, without filling its memory with junk.**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Dependencies: none](https://img.shields.io/badge/dependencies-none-brightgreen)
![Format: plain markdown](https://img.shields.io/badge/format-plain%20markdown-lightgrey)

Rewise is an agent skill that gives a project a small, curated memory of past
mistakes. The agent reads it before working, fixes the mistake and its
leftovers when something goes wrong, and records only the lessons that are worth
keeping.

---

## Table of contents

- [The problem](#the-problem)
- [What Rewise does](#what-rewise-does)
- [Install](#install)
- [How it works](#how-it-works)
- [Example](#example)
- [The record gate](#the-record-gate)
- [Before "done" checklist](#before-done-checklist)
- [What gets installed](#what-gets-installed)
- [Supported agents](#supported-agents)
- [Repository layout](#repository-layout)
- [Customizing](#customizing)
- [FAQ](#faq)
- [Limitations](#limitations)
- [Uninstall](#uninstall)
- [Contributing](#contributing)
- [License](#license)

---

## The problem

AI agents forget everything between sessions, so they repeat mistakes. The most
annoying ones are not logic errors. They are forgotten side effects:

- After a refactor, the agent leaves the **old code behind**, and it breaks the
  feature being built.
- A feature changes, but the **docs that describe it are never updated**.

Neither shows up as a failing test, so ordinary error handling never notices.

## What Rewise does

- **Remembers per project.** Lessons live in a `memory/` folder inside your
  repository, so they can be reviewed, committed, and shared with your team.
- **Learns from two signals.** A failing test/build/validation, and you pointing
  out a mistake.
- **Fixes first, records second.** The agent finds the root cause, repairs the
  mistake including leftover code and docs, and only then decides whether it is
  worth remembering.
- **Keeps memory clean.** A [record gate](#the-record-gate) rejects trivial,
  vague, and duplicate entries. Related lessons are merged, not piled up.
- **Catches silent mistakes.** A short [checklist](#before-done-checklist) runs
  before the agent says a task is done.
- **Stays simple.** Plain markdown. No scripts, no database, no dependencies, no
  vendor lock-in.

## Install

Use the open [`skills`](https://github.com/vercel-labs/skills) CLI. It installs
the skill into the agents it detects (Claude Code, Codex, Cursor, and many more).

```bash
npx skills add MuhammadAidil-dev/rewise
```

| Option | Effect |
|---|---|
| `-a claude-code` | Install for a specific agent instead of choosing interactively |
| `-g` | Install globally rather than into the current project |
| `--skill rewise` | Select the skill by name |

Requires Node.js 22.20 or later (a requirement of the `skills` CLI).

**No Node?** Copy `skills/rewise/` into your agent's skills directory, or tell
your agent: *"Follow `skills/rewise/SETUP.md` to install Rewise in this project."*

### First use

There is nothing else to configure. On first use in a project, the skill
bootstraps itself:

1. Creates `memory/` with `mistakes/`, `lessons/`, `decisions/`, and `index.md`.
2. Copies the rules to `memory/PROTOCOL.md`.
3. Adds a short `## Rewise` block to `AGENTS.md` (or `CLAUDE.md`).

Existing files are never overwritten. The `## Rewise` block means the rules keep
applying in later sessions, and for teammates who did not install the skill.

## How it works

```text
        ┌─────────────────────────────────────────────┐
        │ 1. BEFORE WORK                              │
        │    read memory/index.md → read relevant     │
        │    entries only → follow them               │
        └──────────────────────┬──────────────────────┘
                               ▼
        ┌─────────────────────────────────────────────┐
        │ 2. WORK                                     │
        │    don't repeat approaches recorded as bad  │
        └──────────────────────┬──────────────────────┘
                               ▼
              failing test/build, or you point out a mistake?
                       │ yes                    │ no
                       ▼                        │
        ┌───────────────────────────────┐       │
        │ 3. LEARN                      │       │
        │    root cause → fix + cleanup │       │
        │    → confirm at failing layer │       │
        │    → record gate              │       │
        └──────────────┬────────────────┘       │
                       ▼                        ▼
        ┌─────────────────────────────────────────────┐
        │ 4. BEFORE "DONE"                            │
        │    side-effects checklist → tests/build     │
        │    → record gate → update index             │
        └─────────────────────────────────────────────┘
```

## Example

You tell the agent:

> You refactored `parseDate` but left the old `parseDateLegacy`, and it is still
> being called.

The agent:

1. Finds the real cause: it finished the new code path without searching for
   callers of what it replaced.
2. Removes the leftover helper and updates every reference.
3. Reruns the tests to confirm the fix where it failed.
4. Applies the record gate. No entry covers this, so it writes
   `memory/mistakes/stale-code-after-refactor.md`:

```md
# Stale code after refactor

## Context
Replaced a helper during a refactor.

## Problem
The old helper stayed and was still called, causing inconsistent behavior.

## Root Cause
Finished the new code path without searching for callers of what it replaced.

## Correction
Removed the old helper and updated every reference.

## Prevention Rule
After replacing code, search for all remaining references to the old version
and remove them before reporting done.
```

and adds one line to `memory/index.md`:

```md
- mistakes/stale-code-after-refactor.md — replaced code left behind and still called
```

The next time a similar refactor goes wrong, the agent **extends this entry**
instead of creating another. If you only had a typo, nothing would be recorded.

## The record gate

An entry is written only if **all five** checks pass:

| # | Check | Rejects |
|---|---|---|
| 1 | **Recurrence:** could it happen again in a *different* task? | One-off problems |
| 2 | **Actionable:** can the rule be stated as a concrete check or action? | Vague advice such as "be careful" |
| 3 | **Non-trivial:** more than a typo, syntax error, or outage? | Noise a compiler already reports |
| 4 | **Not a duplicate:** searched the index first; extend an existing entry if one fits | Duplicates |
| 5 | **Root cause known:** does it state the cause, not the symptom? | Guesses |

If any check fails, the agent writes nothing and says nothing about memory.

Hard limits keep memory small:

- One line per entry in `index.md`, no prose.
- One topic per entry, around 60 lines at most.
- Prevention Rule is 1-3 sentences.
- Files are named by topic, never by date or task.
- Wrong or outdated entries are fixed or deleted when found.

## Before "done" checklist

Before reporting a task complete, the agent checks:

- [ ] Did this change replace existing code (functions, files, flags, config,
      routes)? Is the old version removed and every reference updated?
- [ ] Did behavior, an API, config, or setup change? Are the docs updated
      (README, API docs, comments, examples, changelog)?
- [ ] Do any tests, types, or fixtures still describe the old behavior?
- [ ] Did I touch anything a memory entry warns about, and did I follow it?

Projects can extend this list. A new item must itself pass the record gate.

## What gets installed

In your project:

```text
your-project/
├── AGENTS.md              + a short "## Rewise" block
└── memory/
    ├── PROTOCOL.md        the rules
    ├── index.md           one line per entry
    ├── mistakes/          what went wrong, why, and the prevention rule
    ├── lessons/           reusable technical knowledge
    └── decisions/         important choices and their reasons
```

In your agent's skills directory (by the `skills` CLI):
`rewise/` with `SKILL.md`, `PROTOCOL.md`, `SETUP.md`, and `templates/`.

## Supported agents

Rewise has two layers, so it is not tied to one tool:

| Layer | Used by | Role |
|---|---|---|
| `SKILL.md` | Agents that support the skill format (installed via the `skills` CLI) | Triggers the bootstrap and the workflow |
| `AGENTS.md` block + `memory/PROTOCOL.md` | Any agent that reads `AGENTS.md` or `CLAUDE.md` | Keeps the rules active in every session |

Once bootstrapped, the plain-markdown layer works even for agents that cannot
load skills.

## Repository layout

```text
rewise/
├── README.md
├── LICENSE
└── skills/
    └── rewise/
        ├── SKILL.md            entry point agents discover
        ├── PROTOCOL.md         the rules (tool-neutral)
        ├── SETUP.md            bootstrap steps an agent follows
        └── templates/
            ├── AGENTS-block.md block added to AGENTS.md / CLAUDE.md
            ├── index.md        empty memory index
            ├── mistake.md      mistake entry template
            ├── lesson.md       lesson entry template
            └── decision.md     decision entry template
```

## Customizing

- **Checklist:** edit section 7 of `memory/PROTOCOL.md` in your project. Remember
  that new items should come from real incidents.
- **Memory language:** entries follow your project's language. Write them in
  whatever your team reads.
- **Categories:** add headings to `memory/index.md` (for example per module).
- **Rules file:** the bootstrap prefers `AGENTS.md`, falls back to `CLAUDE.md`,
  and creates `AGENTS.md` only if neither exists.

## FAQ

**Does memory override what I tell the agent?**
No. Your current explicit instruction always wins, followed by project
requirements and architecture, then memory, then general conventions.

**Will the agent follow a stale entry blindly?**
No. If an entry looks wrong or outdated, the agent checks the real situation,
does the technically correct thing, and fixes or deletes the entry.

**Will it record every error?**
No. Typos, one-off outages, and obvious compiler errors are rejected by the
record gate.

**Is the memory shared across my projects?**
No. Memory is intentionally per project and stored in that repository.

**Should I commit `memory/`?**
Yes. It is plain text, reviewable in pull requests, and shared with your team.

## Limitations

- The `skills` CLI has no post-install hook, so "automatic" means the skill
  bootstraps the project the first time it is triggered, not when `npx` finishes.
- Whether the skill triggers depends on your agent choosing to load it from its
  description. The `AGENTS.md` block is the safety net after bootstrap.
- Rewise improves consistency. It does not replace code review or tests.
- The `skills` CLI requires Node.js 22.20 or later.

## Uninstall

1. Remove the skill: `npx skills remove rewise`, or delete the `rewise/` folder
   from your agent's skills directory.
2. In each project: delete the `memory/` folder and the `## Rewise` block in
   `AGENTS.md` / `CLAUDE.md`.

## Contributing

[Issues](https://github.com/MuhammadAidil-dev/rewise/issues) and pull requests
are welcome. Please keep the kit:

- dependency-free and plain markdown,
- tool-neutral (no stack-specific or project-specific content in templates),
- small: a new rule should be justified by a real, recurring failure.

## License

[MIT](LICENSE)
