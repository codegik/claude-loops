---
name: engineer
description: Run the engineer development loop for a task — goal → discover → act → verify → remember. Use when the user asks to "run the loop", "engineer this", "/engineer", or hands you a feature/bug/change to take from intent to a reviewed PR. Drives all five stages in sequence, pausing at gates, running verification in segregated subagents, and persisting outcomes to the repo memory/ store.
---

# Engineer Loop

A five-stage loop that takes one task from intent to a reviewed pull request. Run the
stages **in order**. Stop at each **GATE** and wait for the user unless they have
explicitly told you to run unattended.

The guiding principles, coding standards, and project conventions live in
`CLAUDE.md` and in `memory/` — read them before acting.

---

## Stage 0 — SETUP  (ask once, remember for the session)

The **first** time this skill runs in a session, ask the engineer:

> **Create a new feature branch for these changes, or work on the current branch?**

Record the answer and **keep it for the rest of the session** — do not ask again on
later loops in the same session.

- **New branch** → Stage 3 (ACT) creates `loop/<slug>` before editing; Stage 4 pushes
  that branch and opens a PR.
- **Current branch** → Stage 3 commits on the branch already checked out (no new branch);
  Stage 4 pushes the current branch as-is.

---

## Stage 1 — GOAL  (the start point)

The loop always begins from a single, explicit goal.

1. Restate the task as one clear sentence: *what* changes and *why*.
2. Capture acceptance criteria — the observable conditions that mean "done".
3. Note constraints (scope boundaries, non-goals, deadlines, risk).
4. Recall relevant memory: read `memory/index.md`, then any linked goals,
   decisions, and patterns that touch this area.
5. Write the goal to `memory/goals/<slug>.md` (template in `memory/_TEMPLATE.md`).

**GATE 1 — confirm the goal.** Show the restated goal + acceptance criteria and wait
for the user to confirm or correct before discovering. Skip only if the user said to run
unattended.

---

## Stage 2 — DISCOVER

Understand the system before changing it. **No edits in this stage.**

1. Explore the relevant code, configs, and tests. Use the `Explore` agent for broad
   fan-out searches when you only need conclusions.
2. Identify the files to change, the blast radius, and existing patterns to follow.
3. Surface unknowns and risks. Resolve them by reading code, not guessing.
4. Produce a short implementation plan: ordered steps + files touched.

**GATE 2 — confirm the plan.** Present the plan and wait for approval before acting.

---

## Stage 3 — ACT

Implement the change, honoring the branch decision from Stage 0.

1. **Branch.** Follow the SETUP decision:
   - *New branch* → `git switch -c loop/<slug>` off the default branch before editing.
   - *Current branch* → stay on the checked-out branch; do not create one.
   Either way, avoid committing directly to the default branch unless the engineer
   explicitly chose to work on it.
2. Make the change **in the working tree only**. Match surrounding code style.
   **Do not commit** — no `git commit` happens in this stage.
3. Keep the change within the goal's scope — park out-of-scope findings in
   `memory/runs/<slug>.md` for a future loop.
4. Run the build / linters / fast tests locally as you go.

> Do **not** commit and do **not** open the PR here. Both happen only after VERIFY
> approves (Stage 4). If VERIFY sends the work back, keep editing the same working
> tree and re-verify — still uncommitted.

---

## Stage 4 — VERIFY  (segregated agents)

Verification runs in **separate subagents** so the actor does not grade its own work.
Spawn each via the `Agent` tool and act on what they return. Run **tester** and
**reviewer** in parallel; run **approver** last with both reports as input.

1. **tester** (`.claude/agents/tester.md`) — runs the test suite + relevant manual
   checks, reports pass/fail with evidence.
2. **reviewer** (`.claude/agents/reviewer.md`) — reviews the diff for correctness,
   security, and convention adherence.
3. **approver** (`.claude/agents/approver.md`) — reads both reports and the goal's
   acceptance criteria, then returns **APPROVED** or **CHANGES REQUESTED** with reasons.

> Verification runs against the **uncommitted working tree** — nothing has been
> committed yet.

- If **CHANGES REQUESTED** → loop back to Stage 3 (ACT), address the findings in the
  working tree, re-verify. Still no commit.
- If **APPROVED** → **now** commit the approved change, then push and open the PR:
  - `git add -A && git commit -m "<concise summary of the change>"` — a single commit
    for the whole approved change (this is the **only** commit point in the loop).
  - `git push -u origin <branch>`  (the `loop/<slug>` branch, or the current branch)
  - `gh pr create --fill --base <default-branch> --head <branch>`
  - Put the goal, acceptance criteria, and a summary of the three verify reports
    in the PR body.

**GATE 4 — the PR is the deliverable.** Report the PR URL to the user. Do not merge.

---

## Stage 5 — REMEMBER  (memory lives outside the chat)

Persist what was learned so the next loop starts smarter. Write to the repo `memory/`
store (versioned markdown — it travels with the project and shows up in PRs).

1. **Run record** → `memory/runs/<slug>.md`: goal, what shipped, PR link, verify
   outcome, anything parked for later.
2. **Decisions** → `memory/decisions/<slug>.md`: any non-obvious choice and its *why*.
3. **Patterns** → `memory/patterns/<name>.md`: reusable conventions discovered or
   reinforced.
4. Add/refresh one-line pointers in `memory/index.md`.

Only record what is non-obvious and not already captured by the code or git history.

**GATE 5 — close the loop.** Confirm what was remembered and where. The loop ends here.

---

## Notes

- **GitHub connector:** the `gh` CLI (authenticated) is the connector for branches and
  PRs. No scheduled/automated runs — the loop is user-driven for now.
- **Segregation matters:** never let the acting context approve itself. Verification is
  always a fresh subagent.
- **Memory is the spine:** GOAL reads it, REMEMBER writes it. Keep it small and true.
