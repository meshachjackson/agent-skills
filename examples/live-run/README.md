# Live goal-loop run

A real run, not a mock-up. Task: add a `--json` flag to a tiny Python CLI (`textstats`, standard library only). Worker and contract author: the main Claude Code session. Verifier: the `loop-verifier` agent. Date: 2026-11-07.

The final contract is in [contract.md](contract.md).

## What happened

| Step | Result |
|---|---|
| Contract review, round 1 | **FAIL**. Two real faults in criterion 5: a `grep -v` filter that would drop every line (so the check could never fail), and a `git diff HEAD` against a moving ref that would be empty after committing |
| Contract fixed, review round 2 (recheck of that item only) | **PASS** |
| Work pass | `--json` flag added with 2 new tests, committed |
| Full verify | **FAIL** on criterion 5, **PASS** on criteria 1 to 4 |

## Why the final verify failed

My fix to criterion 5 named a fixed base commit, then I committed the contract edit *after* that base. The diff against the base therefore lists the contract file, and the check prints something when it should print nothing. The mistake was mine, not the worker's, and the verifier reported it exactly.

The skill's rule is that the contract is locked after the first verify, so the run stopped there and asked the user how to proceed. It did not quietly rewrite the check to pass.

## What this shows

- The review step earns its place: it caught two checks that looked fine but could not fail.
- A recheck round costs about 24k tokens and 19 seconds. A full verify cost about 25k tokens and 25 seconds.
- The lock works, and it also forces a human decision when the contract itself is wrong. That is the intended behavior, and it has a cost.
- This is one run on a small task. It is an example, not a benchmark.
