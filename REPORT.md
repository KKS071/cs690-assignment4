# Assignment 4 Report

Name: Kundan Singh
Commit graded: 3c8e0dd

Answer from your own traces. Give step numbers as `attempt 1, step 4`. Keep to the word
limits; a short answer with the right evidence earns full points.

## 1. The runs

| Run | Tools | Judge | Stopped (per attempt) | Model calls | Tool calls | Input tokens | Output tokens | Cost | Judge verdict |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| runA | local | after | 1: done | 7 | 9 | 13722 | 693 | $0.0036 | 1: PASS |
| runB | mcp | reflexion | 1: done | 5 | 7 | 7923 | 434 | $0.0021 | 1: PASS |

## 2. A rejection is feedback (Run A, at most 100 words)

The edit I rejected was proposed at attempt 1, step 3. It only uppercased the code (rule 2).
My reason was: "Before editing, check every rule in spec.md
against this function, not only the failing test." At attempt 1, step 4 the agent read
`discounts.py` again, and its thought said it would reassess "all specification rules,
especially the currently unused expiration date". At attempt 1, step 5 it proposed an edit
with both the uppercase lookup and a `today > last_day` check, which I approved. It
followed my reason and found rule 4 without being told which rule was missing.

## 3. The visible tests and the judge (Run A, at most 120 words)

The last `run_tests`, at attempt 1, step 6, reported "PASSED: 4 of 4 tests passed." The
agent then stopped at attempt 1, step 7 with status `done`, because it replied without a
tool call. The judge passed: "PASSED: 6 of 6 tests passed." No spec rule was broken.

The rejection made the difference. In my practice run try1 I rejected the first edit with
"Add a comment that names the spec rule you are following." The agent fixed only rule 2,
the visible tests passed, and the judge failed rule 4
(`test_rule_4_code_is_rejected_after_its_last_day`). The visible tests never check
expiry, so the agent had no way to see that gap.

## 4. The lessons (Run B, at most 150 words)

runB wrote no lessons, and `NOTES.md` is still empty, because attempt 1 passed the judge
("PASSED: 6 of 6 tests passed") and Reflexion only writes a lesson after a failure. There
was no attempt 2.

Attempt 1, step 3 shows why no lesson was needed. At attempt 1, step 2 the agent read
`discounts.py`, `spec.md` and the test file before editing, so its first edit at step 3
already had both the uppercase lookup (rule 2) and the `today > last_day` check (rule 4).
I approved it, the visible tests passed at step 4, and it stopped `done` at step 5. The
lesson path was only exercised by `tests/test_reflexion.py`, with a scripted model.

## 5. Memory grows (Run A, at most 60 words)

The first tool event, attempt 1, step 1, has `input_tokens` 786; the last, attempt 1,
step 6, has 2709. It grows because every model call resends the whole history: the task,
every earlier reply and every tool result, including the files read at step 2. Nothing is
dropped, so each step's input is larger.

## 6. MCP (at most 120 words)

`run_tests`, as tools/list showed it:

"Run every unit test in the project's tests/ folder and report the result.\n\n    Takes no inputs. Returns \"PASSED: N of N tests passed.\" when everything passes.\n    Otherwise the first line is \"FAILED: K of N tests did not pass.\", followed by one line\n    per failing test (at most 10): FAIL or ERROR, the test name, and the last line of its\n    error. Passing tests and tracebacks are not shown. Run it before your first edit to see\n    what fails, and after each edit to check it. The tests stop after 20 seconds.\n    "

Risk: a server's code runs with my permissions and can do anything, such as reading my
`.env`. Nothing in my agent stops that; `gate_for` only limits which tools the model may call.

## 7. The reviewer and the judge (at most 100 words)

The reviewer said runB's `discounts.py` follows all five rules, which matches the judge's
PASS (6 of 6). It missed nothing the judge caught, and found things the judge does not check: whitespace codes like " SAVE10" are not trimmed, a non-string code raises
`AttributeError`, and the visible tests never test rule 4. In Claude Code the read-only rule
was a lock: the header lists only `Read, Grep, Glob`, so the subagent had no Edit or Bash
tool. The main session had no such lock. When I answered "y" by mistake, it edited runB's
test file, which I restored with git.

## 8. Your agent and a vendor's agent (at most 120 words)

The fixer added one line, `code = code.upper()`. `judge vendor` reported visible tests
"PASSED: 4 of 4" and judge "FAILED: 1 of 6", on rule 4
(`test_rule_4_code_is_rejected_after_its_last_day`). Only `discounts.py` changed, nothing in
`tests/`. runA had the same visible result but passed the judge, because my rejection told
it to check the whole spec. The fixer even read `spec.md` and said rule 4 was unhandled, but
its instructions said "smallest change", so it stopped. Its first session ran in auto mode,
so I did not approve that edit.

My agent's `tests/` rule is code in `gate_for`. The fixer's "Do not change anything in
tests/" is only a request, since it has Edit and Bash. Setup was easier: one file, no loop.
