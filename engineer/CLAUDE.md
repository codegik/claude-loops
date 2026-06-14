# Engineer Loop — Project Guide

This repository runs an **engineer development loop**: every task moves through five
stages — **goal → discover → act → verify → remember** — ending in a reviewed pull
request and a memory record.

Start a task with the `engineer` skill (`/engineer "<goal>"`). The full stage
protocol lives in `.claude/skills/engineer/SKILL.md`. Verification runs in the
segregated subagents under `.claude/agents/`. Persistent memory lives in `memory/`.

---

## The loop at a glance

| Stage | Purpose | Output |
|-------|---------|--------|
| **goal** | Pin down what & why + acceptance criteria | `memory/goals/<slug>.md` |
| **discover** | Understand the system, plan the change (no edits) | a plan |
| **act** | Implement on a `loop/<slug>` branch | commits |
| **verify** | tester + reviewer + approver (segregated agents) | APPROVED → PR |
| **remember** | Persist outcomes & learnings outside the chat | `memory/` updates |

## Rules of the loop

- One goal per loop. Out-of-scope findings get parked in `memory/runs/`, not done now.
- At skill start, ask once whether to create a new `loop/<slug>` feature branch or work on
  the current branch; honor that decision for the rest of the session. Avoid committing
  directly to the default branch unless that's the branch explicitly chosen.
- The actor never approves its own work — verification is always a fresh subagent.
- **No commits during the loop.** ACT and VERIFY work on the uncommitted working tree;
  the single commit happens only after the **approver** returns APPROVED, right before
  the push.
- The PR opens only after the **approver** returns APPROVED. Do not merge automatically.
- GitHub is driven by the authenticated `gh` CLI. No scheduled/automated runs for now.

---

<!-- ============================================================ -->
<!-- BELOW: the guidance YOU design. Fill these in for your stack. -->
<!-- ============================================================ -->

## Project context
<!-- TODO: what is this codebase / what does the engineer loop operate on? -->

## Tech stack & tooling
<!-- TODO: languages, frameworks, package manager. -->

## Build, test, lint commands
<!-- TODO: the exact commands the `tester` agent should run. -->

## Coding standards & conventions
<!-- TODO: style, structure, naming, testing expectations the `reviewer` enforces. -->

## Definition of done
<!-- TODO: baseline acceptance bar every change must clear. -->
