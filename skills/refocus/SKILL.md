---
name: refocus
description: Pull a drifting session back to its original goal. Use when the agent is chasing side issues, refactoring things nobody asked for, or has lost the thread, or when the user types /refocus.
version: 0.1.0
disable-model-invocation: true
---

# Refocus

Stop. Do not make the next tool call until you have finished steps 1 to 4.

Long sessions drift. The agent fixes one more thing, adds one more check, and ends up far from the request. This skill resets the session to what the user actually asked for.

## Steps

1. **Find the goal.** Use, in this order:
   - the plan file or task list for this session, if one exists
   - the ticket or card for the work, if there is one
   - the user's first message of the session

   If the user redirected you later in plain words, the latest explicit instruction replaces the first message. If you cannot tell which applies, ask one question and stop.
2. **State it.** One line, in the user's words where you can. Do not reword it in a way that widens it.
3. **Check progress.** Two short lists: done toward the goal, and still needed for the goal.
4. **Name the drift.** List anything you did or were about to do that the goal does not need: side fixes, refactors, extra checks, new tools or dependencies nobody asked for. For each, say what it was and why it is off the goal.
5. **Decide each drift item.**
   - Undo it if it changed files or state and the user did not ask for it. Say so first if the undo is hard to reverse.
   - Otherwise park it: note it as a follow-up in your final message or ticket. Do not do it now.
6. **Resume** with the next step toward the goal. Do not wait for a reply unless step 1 or step 5 needs one.

## Output

Four short parts: Goal, Done / Still needed, Off the goal (what you did about each), Next step. If nothing drifted, skip "Off the goal" and say you checked.
