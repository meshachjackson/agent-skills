# Demo: parallel-agent-waves on a real-shaped plan

A worked example of what the skill does to a plan. The project and numbers are illustrative.

## The plan you approved

"Add CSV export to the reports app."

1. Add an `exporters/csv.py` module.
2. Add an `exporters/json.py` module.
3. Add a `/export` route that picks an exporter.
4. Update the docs page for the new route.
5. Rename `ReportRow` to `Row` across 14 files.
6. Run the full test suite.

## Step 1: dependencies

| # | Reads | Writes | Needs |
|---|---|---|---|
| 1 | `models.py` | `exporters/csv.py` | nothing |
| 2 | `models.py` | `exporters/json.py` | nothing |
| 3 | `exporters/*` | `routes/export.py` | 1, 2 |
| 4 | `routes/export.py` | `docs/export.md` | 3 |
| 5 | 14 files | the same 14 files | nothing, but touches `models.py` |
| 6 | everything | nothing | all |

Step 5 writes `models.py`, which steps 1 and 2 read. It cannot share a wave with them.

## Step 2: the revised plan the skill shows you

**6 steps, 4 waves, 4 agents**

| Wave | Step | Files touched | Model | Why |
|---|---|---|---|---|
| 1 | 1 CSV exporter | `exporters/csv.py` | mid, medium | New code, clear spec |
| 1 | 2 JSON exporter | `exporters/json.py` | mid, medium | Same, no shared file |
| 2 | 5 Rename | 14 files | smallest, low | Mechanical, runs after 1 and 2 read `models.py` |
| 3 | 3 Route | `routes/export.py` | mid, medium | Needs both exporters |
| 3 | 4 Docs | `docs/export.md` | smallest, low | Small, stays in the main session |
| 4 | 6 Tests | none | main session | Verification, not a task |

Step 4 is small, so it does not get its own agent. Only task-sized steps do.

## Step 3: one brief (wave 1, step 1)

```
Task: Add exporters/csv.py with export(rows) -> str. Done = the contract below passes.
Files: you own exporters/csv.py and tests/test_csv.py. Do not touch anything else.

Contract:
  Request: "Add CSV export to the reports app."
  Goals: 1. export() returns valid CSV for any list of Row.
  Exit criteria:
    1. Module exists.
       Check: test -f exporters/csv.py
    2. Round trip works.
       Check: pytest tests/test_csv.py -q exits 0
    3. No other file changed.
       Check: git diff --name-only origin/main...HEAD lists only the two owned files
  Budget: 4
  Rules: no new dependencies; no pushes; do not switch branches.

How to work: invoke goal-loop first. Get an advisor check before you commit to an
approach and again before you say done. End with: done, or blocked and why.
Repo rules: (copied from CLAUDE.md)
```

## Step 4: the final report

```
4 of 4 agent steps passed across 4 waves.

What I did
- Wave 1: CSV and JSON exporters, in parallel (1 iteration each)
- Wave 2: renamed ReportRow to Row in 14 files (1 iteration)
- Wave 3: added the route and the docs page
- Wave 4: full suite passes, run in the main session

Branches (merge when you say go)
- refs/heads/wave1-csv, refs/heads/wave1-json, refs/heads/wave2-rename, refs/heads/wave3-route

Cost: 4 agents started; 3 mid-size, 1 smallest; goal-loop iterations 1, 1, 1, 1
```

## What to notice

- The rename was moved to its own wave, not run "in parallel because it is independent". It shared a file.
- Only the work that earns its own agent got one.
- Each agent was told exactly which files it owns, so no two agents could collide.
