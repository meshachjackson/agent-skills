# Contract: add --json flag

## Request
"Add a `--json` flag to the CLI that prints the report as JSON."

## Goals
1. `mycli report --json` prints valid JSON.
2. Existing text output is unchanged.

## Exit criteria
1. Flag is registered.
   Check: `grep -n -- '--json' src/cli.py` returns at least 1 line.
2. Output is valid JSON.
   Check: `mycli report --json | python3 -m json.tool > /dev/null` exits 0 (run with `set -o pipefail`).
3. Text output unchanged.
   Check: `git diff origin/main...HEAD -- tests/golden/report.txt` is empty.

## Tests
- `pytest -q` exits 0.

## Budget
4 iterations.

## Rules
- No new dependencies.
- No pushes or deploys.
