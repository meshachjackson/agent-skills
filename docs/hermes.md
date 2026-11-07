# Using these skills with Hermes

Result of one test pass on 2026-11-07. The skills are plain Markdown with YAML frontmatter, and Hermes loads them as is. Two things need care: the `loop-verifier` agent file, and sub-agent context size.

## Test setup

| Item | Value |
|---|---|
| Harness | Hermes Desktop App |
| Model | Qwen3.6-35B-A3B-UD-Q4_K_M, custom provider |
| Main context window | 262,144 tokens |
| Sub-agent context | Smaller than the main window. The server rejected delegations of about 13,800 tokens as "too large" |
| Hermes toolset | terminal, delegate_task, write_file, patch, kanban_*, skill_*, web_*, execute_code, browser_*, vision_*, text_to_speech |

This is one model on one harness. Treat it as a first data point.

## Results

| Skill | Verdict | What was tested |
|---|---|---|
| refocus | Fully compatible | Applied the 5-step workflow to the test conversation. No changes needed |
| merge-to-main | Fully compatible | Ran end to end on a real repo and PR. State reached `MERGED`, local `main` SHA equalled remote SHA |
| goal-loop | Compatible in principle | Contract, work and verify all passed (3 of 3 checks) when the verifier ran its checks locally. Sub-agent delegation failed on context size, not on skill format |
| parallel-agent-waves | Compatible in principle | Not tested alone. Waves map to `delegate_task` fan-out, and it has the same sub-agent limit as goal-loop |
| loop-verifier agent | Needs tool-name mapping | See below |

`disable-model-invocation` in `merge-to-main` frontmatter is ignored by Hermes. The skill still works, but the model may offer to run it without being asked.

## Install

```bash
mkdir -p ~/.hermes/skills/agent-skills
cp -R skills/* ~/.hermes/skills/agent-skills/
```

Hermes finds each `SKILL.md` in the subfolders and loads skills by keyword match. No config change is needed.

## The verifier

Hermes has no `~/.claude/agents` folder, so skip `agents/loop-verifier.md`. Use `delegate_task` with the verifier prompt inline instead:

```
delegate_task({context: "You are the loop-verifier. Read <contract path>,
run each Check: command independently, report PASS/FAIL per criterion.
Never edit anything."})
```

If your sub-agent context is too small for that (as in this test), run the `Check:` commands directly in the main session. That passed 3 of 3 in the test. The cost is independence: the same session that did the work now grades it. Keep the checks mechanical (a command and an exit code) so there is little to rationalize.

### Tool name mapping

`loop-verifier` declares Claude Code tool names. Map them like this:

| Claude Code | Hermes |
|---|---|
| Read | `read_file`, or `terminal` with `cat` |
| Grep | `search_files` with `target='content'` |
| Glob | `search_files` with `target='files'` |
| Bash | `terminal` |

## Known limits

- **Sub-agent context.** If your harness gives sub-agents a smaller window than the main session, keep delegated prompts small. Pass a contract path, not the contract text.
- **parallel-agent-waves** depends on sub-agent spawning. If delegation is rejected, it falls back to running the plan in the main session.
- **Not tested:** other models, other Hermes versions, and a full multi-wave run.
