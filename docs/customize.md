# Customize for your setup

These skills are plain Markdown. Edit them like any other file in your config.

## Other agent harnesses

The skills use Claude Code names where they must. To port:

1. In `parallel-agent-waves`, edit the "Mapping to your harness" table. Replace the `Agent` tool and the `advisor` tool with whatever your harness calls them.
2. In `agents/loop-verifier.md`, keep the tool list read-only (`Read, Grep, Glob, Bash`). The verifier should never be able to edit.
3. If your harness has no sub-agents, goal-loop still works: run the verifier prompt in a fresh session with only the contract and output paths.

## Other model providers

The model table in `parallel-agent-waves` uses tiers (smallest, mid, your own), not model names, on purpose.

- Replace each tier with a real model ID your harness can reach.
- Keep the rule: the cheapest model that still does the job.
- Use a different provider for the verifier if you want a grader with different blind spots than the worker.

## Tuning

| Setting | Where | Default | Change it when |
|---|---|---|---|
| Iteration budget | goal-loop, "Budget" | 4 | Tasks are bigger, or you want a tighter leash |
| Contract review rounds | goal-loop, section 2 | 2 | You want more or less review |
| Wave size cap | parallel-agent-waves, step 1 | 4 agents | Your rate limits or machine allow more |
| Model per step type | parallel-agent-waves, step 3 | tiers | Your provider's names or prices differ |
| Repo rules | each brief | copied from `CLAUDE.md` | Your project uses another rules file |

## Add your own rules

Put hard limits in the contract's `Rules` section: no pushes, no deploys, no new dependencies, no paid calls. Both skills treat repo rules as winning over the skill itself.
