---
name: goal-loop
description: Run a task as a goal loop. Write a contract (goals, exit criteria, checks, budget), work, verify with an independent agent, repeat on FAIL until PASS or budget. Use for "goal loop", "verify until done", or exit criteria plus a verifier.
version: 1.1.0
---

# goal-loop

You do the work; the `loop-verifier` agent grades it. Never grade your own work.

Use for goal loops, exit criteria, verifiers, or "keep going until it passes", when "done" is easy to fake. Skip small one-step tasks.

## 1. Contract (before any work)

Write it to the repo's plan location (e.g. `docs/plans/<yyyy-mm-dd>-<slug>.md`), else the scratchpad. Parts:

- **Request.** The user's words, verbatim. Criteria are judged against this, not your reading of it.
- **Goals.** Three at most, one or two sentences.
- **Exit criteria.** Numbered. Each names a count, file, command or field (no "good", "clear", "thorough") and has a `Check:` line: the command, or the file and what to look for. If you cannot write one, rewrite or drop the criterion.
- **Tests.** Commands that must pass. For analysis tasks, every `file:line` claim must be real.
- **Budget.** Max iterations, default 4 (one work pass plus one verify).
- **Rules.** What is off limits (writes, pushes, deploys, new dependencies, paid or rate-limited calls), from the repo's `CLAUDE.md` and the user's prompt. Copy the repo's `CLAUDE.md` rules in; do not just point at the file.

**Check-writing pitfalls.** These caused most contract review failures:

- Shell quoting: `\$` inside double quotes. Use single quotes, or test the command once.
- A grep pattern that matches its own source (the contract, the check line, the script). Anchor it or exclude the file.
- Pipe exit codes: `cmd | grep x` hides the failure of `cmd`. Use `set -o pipefail` or check each step.
- Diffing against a moving `origin/main`. Name a fixed base, such as `origin/main...HEAD` after a fetch, or a commit SHA.
- Missing repo rules: read the repo's `CLAUDE.md` and copy its constraints into Rules.

After the first verify, changing the contract needs the user's OK. If a criterion is wrong or impossible, stop and ask.

## 2. Review the contract

Spawn `loop-verifier` with `mode: review contract` and the contract path only. It uses a fixed checklist. Round 2 passes `recheck only: <items flagged>`; the verifier then re-checks only those and adds nothing new.

Cap: at most 2 review rounds. A review round is not an iteration. If round 2 still fails, stop reviewing, proceed to work, and write under Rules: `Contract review capped at 2 rounds; the final work verify is the gate. Open review items: <list>.` Ask the user only if a criterion is impossible or the Request is unclear. If the reply lacks `Scope: contract`, the agent definition is stale: say so and stop.

Then show the user the contract as a short list and start, unless they asked to approve it first or it needs a risky action.

## 3. Work

One pass against the contract; save output where it says.

## 4. Verify

Spawn `loop-verifier` with only the contract path, the output path, and from iteration 2 on `recheck only: <numbers of the criteria that failed>`.
Rechecks can miss regressions, so PASS is final only after one pass over all criteria. If `loop-verifier` is unavailable, say so and stop.

## 5. Decide

- **PASS on a full pass:** report.
- **PASS on a recheck round:** run one full pass. It is not a new iteration unless it fails.
- **FAIL, budget left:** fix only the fix-list items, then verify with a recheck list.
- **FAIL, budget spent:** stop and report what still fails.
- **Same criterion fails twice for the same reason:** stop and ask the user.

## 6. Report

Plain language, headline first: PASS in N iterations, or FAIL with what is left. Then, skipping empty ones: What I did; Verifier result; Blocked on me; Blocked on you (with the reason); Ready. Name the contract path. End with `Cost:` iterations used, and verifier calls split into review, recheck and full.

## Hard rules

- The contract exists before the first work pass.
- The verifier sees the contract and output, not your notes.
- An uncheckable criterion counts as failed.
- Never edit the contract to make a failing check pass.
- Repo rules win. This skill grants no permission to push, deploy, write protected tables or add dependencies.
- Stay inside the budget; more iterations need the user's OK.
