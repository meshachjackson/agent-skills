---
name: merge-to-main
description: Take the current branch's pull request all the way to merged, with local and remote both back on main and in sync. Confirms the merge happened before any cleanup. Use when the user says "merge", or types /merge-to-main.
version: 0.1.0
disable-model-invocation: true
---

# Merge to main

The job is not done at "PR merged". It is done when the end-state checks in step 8 pass.

The goal, set before you start and checked at the end:

1. The PR is pushed and merged.
2. Local matches remote: no unpushed commits, no uncommitted work left behind.
3. Local and remote are both back on `main`, with local `main` equal to `origin/main`.

The rule that makes this safe: **merge on its own, confirm the state is MERGED, and only then clean up.** Never chain cleanup to the merge command. If the merge did not happen, the worktree and branch are your only copy of the work.

## Steps

1. **Orient.** In the repo and worktree you changed, run `git status`, `git worktree list`, and `gh pr list --head <branch>`. Name the PR with its title. No PR yet? Push the branch and open one. Never touch another session's branch or uncommitted changes.
2. **Commit** finished work. Leave nothing uncommitted without saying so and why.
3. **Push** the branch. Confirm local `HEAD` equals `origin/<branch>` (`git rev-parse HEAD origin/<branch>`).
4. **Ready to merge?** Run `gh pr view <n> --json mergeable,mergeStateStatus,statusCheckRollup`.
   - Behind or conflicted: rebase or merge `origin/main` in the existing worktree, resolve, re-run tests, push, re-check.
   - Checks failing: fix or report. Do not merge red unless the user says so.
   - Required review missing: stop and say who it is waiting on.
5. **Merge, alone.** Run `gh pr merge <n>` by itself, using the repo's usual method (`--squash` unless the repo says otherwise).
6. **Confirm MERGED.** `gh pr view <n> --json state,mergeCommit`. Continue only when the state is `MERGED`. If not, stop, report why, and leave the worktree and branch untouched.
7. **Return to main.**
   - From the main checkout: `git fetch --prune origin`, then `git switch main` and `git pull --ff-only`. If another session's branch or uncommitted changes are on the main checkout, skip the switch and pull, say so, and use `git fetch` only.
   - Remove your worktree (`git worktree remove <path>`), then delete the local branch (`git branch -d <branch>`; `-D` only if the PR shows MERGED and squash left it looking unmerged).
   - Delete the remote branch if GitHub did not (`git push origin --delete <branch>`).
8. **Verify the end state** and show the output:
   - `git status -sb` on the main checkout: on `main`, not ahead or behind, clean (or only other sessions' changes, named).
   - `git rev-parse main origin/main`: same SHA.
   - `git branch -a` and `git worktree list`: your branch and worktree are gone.
   - `gh pr view <n> --json state`: `MERGED`.

   If any check fails, fix it or name it as blocked. Do not report done.

## Final message

Plain language, scannable:

- One-line headline: PR merged, local and remote on `main`.
- What was just built, in a sentence or two (PR with its title).
- What can be tested now, and where.
- Work that was waiting on this PR and can go now.
- Remaining steps sorted as: blocked on the user, blocked on you (with the reason), ready. Do the ready ones you are already allowed to take instead of listing them.

## Hard rules

- Never force-push, and never merge a red PR without the user's say-so.
- Never switch, pull, stash or commit on a shared checkout holding another session's work.
- Repo rules win. This skill grants no new permissions.
