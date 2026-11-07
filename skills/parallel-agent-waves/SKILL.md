---
name: parallel-agent-waves
description: Turn a proposed plan into parallel waves, then run each step as its own sub-agent or session, each with a right-sized model, an advisor check and a goal-loop contract. Use when the user says "run this in waves", "parallelize the plan", or approves a plan with two or more independent steps.
version: 0.1.0
---

# Parallel agent waves

Rewrite the plan so independent steps run at the same time. Each step runs as its own agent with the right model, a check before it commits to an approach, and a goal-loop contract, so it finishes without wandering and without wasting tokens.

Requires the `goal-loop` skill and `loop-verifier` agent from this repo.

## Mapping to your harness

This skill is written in terms of three capabilities. Use whatever your Claude Code setup offers:

| Capability | Typical way to do it |
|---|---|
| Start a sub-agent with a chosen model and effort | The `Agent` tool with a `model` override, or a separate session if your harness has one |
| Advisor check (second opinion before committing and before "done") | An `advisor` tool or a reviewer sub-agent, if you have one; otherwise skip and rely on the verifier |
| Goal-loop contract | The `goal-loop` skill in this repo |

If your harness cannot run agents at the same time, say so and run the plan normally. Do not fake parallelism.

## Steps

### 1. Revise the plan into waves

1. List every step with what it reads, what it writes (files, tables, branches) and what it needs from other steps.
2. Group steps into waves:
   - A wave holds steps that need nothing from each other and write no file in common.
   - A step goes in the earliest wave where everything it needs is already done.
   - Steps that must land in one change are merged into one step.
   - Verification-only steps (such as running the test suite) go in the last wave and run in the main session.
3. Mark which steps are real tasks (a branch, a PR, a result to hand back) and which are small. Small steps stay in the main session.
4. Cap a wave at 4 agents. Split a bigger wave in two.
5. Show the revised plan as a table: wave, step, files touched, model, why. Lead with one line: "N steps, W waves, S agents". Then start Wave 1 unless the user asked to approve first.

If no two steps are independent, say so in one line and run the plan normally.

### 2. Gather tools and rules

Sub-agents do not inherit what you read. Before writing briefs:

1. Read the repo's `CLAUDE.md` (or equivalent) and copy its rules into every brief.
2. Name the skills, MCP tools and permissions each step needs, and the ones it must not use.

### 3. Choose model and effort per step

Cheapest that still does the job.

| Step type | Model | Effort |
|---|---|---|
| Read-only search, inventory, doc lookup | Smallest | low |
| Mechanical edits, renames, fixture updates, tests from a spec | Mid | low or medium |
| Normal feature or bug fix with clear criteria | Mid | medium |
| Design-heavy, ambiguous, cross-cutting, data migrations | Your own model | high |

### 4. Write a goal-loop contract per step

Follow the `goal-loop` format: Request, Goals (3 max), numbered Exit criteria each with a `Check:` line, Tests, Budget (default 4), Rules.

- Every criterion names a count, file, command or field. No "good" or "clear".
- Rules include: only touch the files this step owns, commit nothing outside them, never switch branches in a shared checkout.
- Do not review the contract yourself. The agent runs goal-loop and `loop-verifier` does that.

### 5. Brief and start the wave

Start one agent per step, all in one message so they run together. Use an isolated git worktree for any step that edits code. Each brief has, in order:

1. **Task.** One paragraph and what "done" looks like.
2. **Files.** What the step owns and what it must not touch.
3. **Contract.** The goal-loop contract from step 4.
4. **How to work.** Invoke `goal-loop` first and follow it; do not grade your own work. Get an advisor check once before committing to an approach and once before declaring done, and make the work durable (saved, committed) before the final check. End with a clear result: done, or blocked and why.
5. **Repo rules.** Copied from `CLAUDE.md`.

### 6. Between waves

1. Check each result against its contract. A report is not proof: look at the diff or output.
2. If an agent failed or is blocked, decide: send a follow-up, redo it yourself, or ask the user.
3. Agents in their own worktree commit to their own branch. Before the next wave needs their work, name each branch by full ref (`refs/heads/...`) and ask whether to merge. Never merge or push on your own.
4. Re-check the next wave against what actually happened and revise it before starting.

### 7. Final report

Headline first: "N of M steps passed across W waves." Then, skipping empty ones: What I did (one line per wave); What needs attention; Blocked on the user / Blocked on you / Ready; Branches and their state. End with `Cost:` agents started, models used, and goal-loop iterations per step.

## Hard rules

- Two agents never write the same file. If two steps touch one file, they go in different waves.
- No agent pushes, deploys, adds dependencies or writes protected data unless the repo rules and the user's prompt allow it.
- Never put secrets or confidential data in a brief.
- At most 4 live agents at a time.
- Do not start a wave until its inputs exist.
- This skill grants no new permissions. Repo rules win.
