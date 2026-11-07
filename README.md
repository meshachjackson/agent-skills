# agent-skills

Two Claude Code skills for work where "done" is easy to fake.

- **goal-loop**: write a contract (goals, exit criteria, a runnable check for each), do the work, and let an independent `loop-verifier` agent grade it. Repeat until PASS or the budget runs out. You never grade your own work.
- **parallel-agent-waves**: turn a plan into waves of independent steps, run each step as its own agent with a right-sized model, and give each one a goal-loop contract.

## Install

Copy into your Claude Code config:

```bash
mkdir -p ~/.claude/skills ~/.claude/agents
cp -R skills/goal-loop skills/parallel-agent-waves ~/.claude/skills/
cp agents/loop-verifier.md ~/.claude/agents/
```

`parallel-agent-waves` needs `goal-loop` and `loop-verifier`. Install all three.

## 30-second demo

1. Tell Claude: "goal loop: add a `--json` flag to the CLI."
2. It writes a contract like [examples/sample-contract.md](examples/sample-contract.md) before any code.
3. `loop-verifier` reviews the contract (max 2 rounds), then runs each `Check:` after the work.
4. You get `PASS in 2 iterations` or a fix list of exactly what still fails.

## Why it works

- The verifier sees the contract and the output, never the worker's notes.
- A criterion with no runnable check counts as failed.
- The contract cannot be edited to make a failing check pass.
- A list of common check-writing mistakes (shell quoting, self-matching grep, hidden pipe exit codes, moving diff bases) is built in.

## Status

Early. Tested in Claude Code. Issues and PRs welcome.

## License

MIT, see [LICENSE](LICENSE).
