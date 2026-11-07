# agent-skills

Skills that make AI coding agents finish what they start, and prove it.

Agents are good at saying "done". They are less good at being done. This repo has four small skills for [Claude Code](https://code.claude.com/docs/en/skills) that close that gap. Each is one folder with one `SKILL.md`. Drop it in and go.

| Skill | What it does | Use it when |
|---|---|---|
| [goal-loop](skills/goal-loop/SKILL.md) | Writes a contract, does the work, and has a separate agent check it | "Done" is easy to fake |
| [parallel-agent-waves](skills/parallel-agent-waves/SKILL.md) | Splits a plan into steps that can run at the same time, one agent each | A plan has 2 or more independent steps |
| [merge-to-main](skills/merge-to-main/SKILL.md) | Merges a PR, confirms it merged, then cleans up | You say "merge" and want local and remote back on `main` |
| [refocus](skills/refocus/SKILL.md) | Pulls a drifting session back to the original request | The agent is chasing side issues |

## Install

```bash
git clone https://github.com/meshachjackson/agent-skills
cd agent-skills
mkdir -p ~/.claude/skills ~/.claude/agents
cp -R skills/* ~/.claude/skills/
cp agents/loop-verifier.md ~/.claude/agents/
```

`parallel-agent-waves` needs `goal-loop` and the `loop-verifier` agent. Install all three together.

## goal-loop

**The problem.** You ask an agent to "add a flag and make sure it works". It adds the flag, runs nothing, and says it works. You find out later.

**What goal-loop does.**

1. Before any work, the agent writes a short contract. It has your request word for word, up to three goals, and numbered exit criteria. Each criterion has a `Check:` line with a command or file to inspect.
2. A separate `loop-verifier` agent reviews the contract first. Vague criteria like "works well" fail here.
3. The agent does the work.
4. The verifier checks each criterion against real files and real commands. It returns PASS or FAIL with a fix list. It never fixes anything.
5. On FAIL, the agent fixes only what failed, and the verifier rechecks only that. This repeats until PASS or the budget runs out (4 rounds by default).

See a full contract in [examples/sample-contract.md](examples/sample-contract.md).

**Where the idea comes from.** Anthropic does not publish a single named "loop architecture". This skill builds on two patterns it does describe:

- **Evaluator-optimizer.** One model generates, another evaluates and gives feedback, in a loop. Anthropic says it works best when the criteria are clear. See [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents).
- **Generator and evaluator with an agreed contract.** In [Harness design for long-running application development](https://anthropic.com/engineering/harness-design-long-running-apps), the generator and evaluator agree on testable success criteria before each chunk of work. goal-loop makes that step a hard rule.

goal-loop is an independent implementation. It is not an Anthropic product.

**How it differs from other loop skills.** There are good goal and verify kits out there. From their public READMEs, here is where goal-loop makes different choices:

| Choice | goal-loop | Why |
|---|---|---|
| Who grades | A separate agent that sees only the contract and the output, never the worker's notes | A grader who reads the worker's reasoning tends to agree with it |
| Contract review | The contract is reviewed before any work, capped at 2 rounds | Bad criteria are cheaper to fix before the work than after |
| Every criterion has a runnable `Check:` | Required. A criterion you cannot check counts as failed | "Looks good" is not evidence |
| Contract is locked | After the first verify, changing it needs your OK | Stops the agent from moving the goalposts |
| Rechecks | Later rounds recheck only what failed, then one full pass before PASS | Cheaper, and still catches regressions |
| Check-writing pitfalls | A built-in list: shell quoting, self-matching grep, pipes that hide failures, moving diff bases | These caused most real contract failures |

We compared against public READMEs and blog posts, not full source. If we got a project wrong, open an issue.

## parallel-agent-waves

**The problem.** A plan has six steps. Four of them do not depend on each other, but the agent does all six in a row. Or you start four agents at once and two of them edit the same file.

**What it does.** It reads your plan and sorts the steps into waves. Steps in one wave need nothing from each other and never write the same file. Each step that is big enough gets its own agent, with the right model size, a goal-loop contract, and a second-opinion check. Small steps stay in the main session. Then it starts wave 1.

See [examples/parallel-waves-demo.md](examples/parallel-waves-demo.md) for a worked example, from plan to final report.

**Why it works this way.**

- **Waves, not one big fan-out.** Two agents writing the same file is the most common way parallel agents break each other. Sorting by what each step reads and writes prevents it.
- **A contract per agent.** Each agent runs goal-loop, so "done" means a check passed, not "I think so".
- **Branches are never merged for you.** Between waves, the skill names each branch and asks. You stay in control of what lands.
- **No fake parallelism.** If no two steps are independent, it says so and runs the plan normally.

**How it can save tokens.** These are mechanisms, not benchmarks. We have not measured totals yet.

- **Cheaper models where they are enough.** Search, renames and test fixtures go to a small model. Only design-heavy steps use your strongest one.
- **Smaller context per agent.** Each agent gets a brief with its task, its files and its rules, not the whole conversation.
- **Short feedback loops.** The verifier reads a contract and an output, not a transcript. Rechecks cover only what failed.
- **Budget caps.** Every contract has an iteration limit, so a stuck step stops instead of burning tokens.

Running agents side by side can use more tokens in total than running them one at a time. What you get is the same work in less wall-clock time, with fewer wasted tokens per step.

**Make it fit your setup.** The skill is written in terms of three capabilities, not specific tools:

| Capability | Default | Swap in |
|---|---|---|
| Start a sub-agent with a chosen model | Claude Code's `Agent` tool | Your harness's sub-agent or session API |
| Pick a model per step | Smallest, mid, and your own model | Any tiers you have, including other providers if your harness can route to them |
| Second-opinion check | An `advisor` tool or reviewer sub-agent | Any reviewer, or skip it and rely on the verifier |

To customize, edit the "Mapping to your harness" table and the model table in `skills/parallel-agent-waves/SKILL.md`. Details in [docs/customize.md](docs/customize.md).

## merge-to-main

Merging is where agents get careless. They chain `merge && delete branch && remove worktree` in one command, the merge fails quietly, and the only copy of the work is gone.

merge-to-main runs the merge by itself, confirms the PR state is `MERGED`, and only then cleans up. It finishes by showing that local `main` equals `origin/main`, and that the branch and worktree are gone. It will not touch another session's branch on a shared checkout.

## refocus

Long sessions drift. refocus makes the agent restate the original goal, list what is done and what is left, name everything it did that the goal did not need, and undo or park each item. Then it resumes. Run it with `/refocus`.

## Status

Early. Written and used in Claude Code. The skills are plain Markdown, so they port, but only Claude Code is tested. Issues and pull requests are welcome.

## License

MIT. See [LICENSE](LICENSE).
