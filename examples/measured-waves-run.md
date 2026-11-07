# Measured waves run

A real comparison of the same four steps done two ways. Date: 2026-11-07.

**Task.** Add four small, independent modules to a Python project (`vowels`, `longest line`, `top words`, `reading time`), each with tests. Standard library only.

**Arms.**
- **Waves.** Four agents at once, one per step, each in its own git worktree, same model tier for all.
- **Baseline.** One agent does all four steps in a row, same model tier, same checks.

## Results (n = 1)

| | Baseline (1 agent) | Waves (4 agents) |
|---|---|---|
| Tokens reported by the harness | 111,434 | 440,170 (110,049 / 109,859 / 110,109 / 110,153) |
| Agent time | 29.7 s | 14.9 / 17.5 / 19.7 / 24.1 s (sum 76.2 s) |
| Wall-clock (slowest agent) | 29.7 s | about 24.1 s |
| Tests in the end | 8 | 13 after merging all four branches |
| File overlap between agents | n/a | none (each branch touched only its own 2 files) |
| Merge | n/a | clean, no conflicts |

## What it means

- **Time.** Waves finished about 19% sooner. That is about 5.6 seconds on a job this small.
- **Tokens.** Waves used about 3.9 times as many. Each agent used about 110k tokens whether it built one module or four, so most of each agent's count is a fixed per-agent cost, not the work.
- **The README's token claims.** Cheaper models and capped budgets still save tokens in principle, but "smaller context per agent" did not show up here. On small tasks, every extra agent adds a large fixed cost.
- **When waves should pay off.** Steps that are long, so the fixed cost is small next to the work, and steps that take real wall-clock time. These four steps took seconds each, which is the worst case for waves.

## Caveats

- One run per arm. No repeats, no variance.
- Token numbers are the harness's per-agent count. We did not check whether it includes cached input.
- Same model tier in both arms, so this does not test the "cheaper model per step" saving.
- The waves arm ran its checks inline; it did not run `goal-loop` or a verifier per agent. That would add cost.
- The task was written to be easy and fully independent.

Treat it as a first data point, and rerun it on larger tasks before drawing a conclusion.
