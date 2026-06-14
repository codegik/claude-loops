# Engineer Loop

A Claude Code configuration that runs one task through five stages —
**goal → discover → act → verify → remember** — ending in a reviewed pull request.

## Run it

```
/engineer "<your goal>"
```

The skill drives all five stages, pausing at gates for your confirmation.

## How it's wired

```
.claude/
  skills/engineer/SKILL.md   # the orchestrator (5-stage protocol)
  agents/                         # segregated verify stage
    tester.md                     #   runs tests, reports pass/fail
    reviewer.md                   #   reviews the diff
    approver.md                   #   final APPROVED / CHANGES REQUESTED gate
  settings.json                   # allowlists the gh/git calls the loop uses
memory/                           # versioned, lives outside the chat
  goals/  decisions/  patterns/  runs/
  index.md                        # one-line pointers, read at GOAL, updated at REMEMBER
  _TEMPLATE.md                    # entry templates
CLAUDE.md                         # project guide — fill in the stack-specific sections
```

## Principles

- One goal per loop; out-of-scope work is parked in `memory/runs/`.
- All changes on a `loop/<slug>` branch — never the default branch.
- The actor never approves its own work; verification is a fresh subagent.
- The PR opens only after `approver` returns APPROVED. No auto-merge, no scheduled runs.
- GitHub via the authenticated `gh` CLI.

## To do before first run

Fill in the marked sections of `CLAUDE.md` (stack, build/test/lint commands, conventions,
definition of done) — that's the guidance the verify agents enforce.
