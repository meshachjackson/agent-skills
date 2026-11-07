# Contract: add --json flag

## Request
"Add a --json flag to textstats that prints the report as JSON instead of the text lines."

## Goals
1. `python3 -m textstats.cli --json` prints the report as valid JSON with keys words, lines, chars.
2. Default output (no flag) is unchanged.

## Exit criteria
1. Flag is handled in the CLI.
   Check: `grep -n -e '--json' textstats/cli.py` returns at least 1 line.
2. JSON output is valid and has the three keys.
   Check: `set -o pipefail; printf 'a b\nc' | python3 -m textstats.cli --json | python3 -c 'import sys,json; d=json.load(sys.stdin); assert set(d)=={"words","lines","chars"} and d["words"]==3'` exits 0.
3. Default output unchanged.
   Check: `printf 'a b\nc' | python3 -m textstats.cli` prints exactly the three lines `words: 3`, `lines: 2`, `chars: 5`.
4. A new function has a test.
   Check: `grep -c 'def test_' tests/test_core.py tests/test_cli.py 2>/dev/null` total is at least 2, and `python3 -m unittest discover -s tests -t .` exits 0.
5. Rules respected.
   Check: with fixed base commit `ecadf9a`, `(git diff --name-only ecadf9a; git ls-files -o --exclude-standard) | grep -v -E '^(textstats/|tests/|docs/plans/2026-11-07-json-flag\.md$)'` prints nothing (changed and untracked files are only under `textstats/` or `tests/`, plus this contract file itself), and `grep -rhE '^(import|from) ' textstats | grep -v -E '^(import|from) (sys|json|argparse|textstats)'` prints nothing.

## Tests
- `python3 -m unittest discover -s tests -t .` exits 0.

## Budget
4 iterations.

## Rules
- Standard library only. No new dependencies.
- No pushes, no network.
- Keep functions small; add a test for every new function.
- Do not edit this contract after the first verify.

## Amendments
- Amendment 1 (user-approved after the first full verify): criterion 5 now allows this contract file in the diff. The first full verify failed only because the author's earlier fix edited the contract after naming the base commit. No other criterion changed.
