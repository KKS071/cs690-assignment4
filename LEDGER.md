# Provenance Ledger

Write one entry per reviewable change or experiment, when the work is done. Use exactly
the schema in HANDOUT.md. Put the prompts you typed into AI tools in `prompts/` and name
the file in the entry's `prompts` field.

For a run, most of the entry comes straight from its trace: what the agent did and in
what order, the commands that checked it, and the result. You add what only you know:
how you reviewed it, what you rejected, and what is still uncertain.

## Worked example (not graded; leave it here and add your entries under "My entries")

This shows the level of detail expected. The commit SHAs, dates, steps and numbers are
made up, and so is the practice run.

```
## Entry 2
artifact:  agent/loop.py, run_agent, at commit 9c3a71e
tool:      Claude Code, model Claude Sonnet 5.5, 2026-11-01
prompts:   asked for the loop body from the comment above run_agent;
           prompts/loop-01.md
review:    read every line against the comment. It returned as soon as is_stuck was
           true, which skipped the tool results for the other calls in the same turn;
           moved the stuck check after the for loop as the comment says. Rejected its
           suggestion to import a framework's message classes; kept the dicts from
           core.py
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3; the comment above run_agent, steps 4 and 5
risk:      no test of mine has three tool calls in one turn

## Entry 4
artifact:  runs/try1, a practice run, agent code at commit 4b9e2d1
tool:      my agent (python -m agent run try1), anthropic claude-haiku-5-5, 2026-11-02
prompts:   the default task and BASE_SYSTEM in agent/loop.py; prompts/try1-task.md
review:    read every event in trace.jsonl. At attempt 1, steps 4 and 6, the model
           copied old_text with the read_file line numbers, and edit_file refused both
           times. At step 7 it made an edit that changed nothing the tests check, and
           at step 8 the stuck rule stopped it. I approved the step 7 edit without
           reading it closely
checks:    python -m agent check-trace try1: complete
evidence:  HANDOUT Part 1 (tool descriptions) and Part 2 (is_stuck)
risk:      one practice run; a better description may still not stop the copying
dataset:   fixture/ at commit 4b9e2d1, task "Make the tests pass."
result:    stopped stuck after 8 model calls, 14,880 input tokens, about $0.002;
           runs/try1/trace.jsonl
changed:   wrote "copy it without the line numbers" into the read_file and edit_file
           descriptions, and read every diff before approving
```

## My entries

## Entry 1
artifact:  Part 0 setup: .env, .gitignore, prices
tool:      none (set up by hand), 2026-10-28
prompts:   none
review:    provider OpenAI, model gpt-5.6-luna (the default). Prices: input $0.20 per
           million tokens, output $1.20 per million tokens, from the GPT-5.6 Luna model
           page, https://developers.openai.com/api/docs/pricing?latest-pricing-expanded=true. My virtual environment is named venv/, not .venv/, so I added venv/ to .gitignore
checks:    python -m agent check-setup: All checks passed (Python 3.12.3, provider
           openai, model gpt-5.6-luna, tool-calling test 143 tokens in, 13 out);
           git status does not list .env or venv/
evidence:  HANDOUT Part 0; README Setup steps 1 to 5
risk:      I used the standard input price, not the cached-input price ($0.02), so
           summary may overstate the cost slightly if any input was cached

## Entry 2
artifact:  agent/tools.py at commit 1bd8d70
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for help with Part 1 written from the comments in the starter file;
           prompts/code-parts1-5.md
review:    read the five functions against the comment above each. Checked by hand
           that safe_path refuses ../judge/acceptance_test.py and an absolute path,
           that read_file returns a numbered window with the header line, and that an
           edit whose old_text is not in the file is refused and changes nothing (the
           count is checked before writing). Kept the sentence in the read_file and
           edit_file descriptions about leaving out the line numbers, since copying
           numbered lines into old_text is the likeliest way an edit fails. Confirmed
           format_report prints no timings or tracebacks, because is_stuck compares two
           outputs as exact text
checks:    pytest tests/test_tools.py: 16 passed
evidence:  HANDOUT Part 1; the comments above each tool
risk:      a file that is not UTF-8 makes read_file raise UnicodeDecodeError, which the
           toolbox reports only as "Error executing tool read_file" with no advice;
           the fixture has no such file, so this is untested

## Entry 3
artifact:  agent/policy.py at commit 081f85c
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for help with Part 2 written from the comments in the starter file;
           prompts/code-parts1-5.md
review:    checked that the tests/ rule replaces "\" with "/" before normpath, so
           tests\x.py and src/../tests/x.py are denied, and that a missing or non-string
           path returns "ask" instead of crashing. Checked budget_stop tests steps
           before tokens, and is_stuck needs all three conditions so running the tests
           twice with no edit in between does not stop the run
checks:    pytest tests/test_policy.py: 7 passed; my own gate_for check on five paths
evidence:  HANDOUT Part 2; the comments above gate_for, budget_stop, is_stuck
risk:      the tests/ check is case-sensitive: Tests/x.py comes back "ask", and on a
           case-insensitive file system (Windows, macOS) that is the same folder. The
           person approving the diff is the only thing that would catch it

## Entry 4
artifact:  agent/loop.py at commit d0f9551
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for help with Part 3 written from the comment above run_agent, with no agent
           framework or SDK loop helper; prompts/code-parts1-5.md
review:    read run_agent against steps 1 to 5 of the comment. Checked: the budget is
           checked before each model call; turn.message is appended before any tool
           runs; an edit with no preview runs without asking; the person's reason is
           kept only when an edit is rejected, so the trace reason is "" otherwise; the
           stuck check happens after every call in the turn, so each call still gets its
           result in the history. Every exit goes through one finish() helper, so each
           stop writes exactly one stop event. The loop uses only provider.next_turn,
           one plain API call per step
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3 and "The rules for AI tools" rule 1; the comment above
           run_agent
risk:      an exception from the provider that is not AIError (for example a raw
           network error) would crash the loop without writing a stop event; no test
           of mine has a rejection and a stuck stop in the same run

## Entry 5
artifact:  agent/reflexion.py at commit 477a384
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for help with Part 4 written from the comments in the starter file;
           prompts/code-parts1-5.md
review:    checked that NOTES.md is read fresh at the start of every attempt, that an
           "error" status stops before the judge runs, that a newline is added first if
           NOTES.md does not end with one, and that the lesson is collapsed to one line
           of at most 600 characters. Read judge/acceptance_test.py next to the visible
           tests: the visible tests cover rule 2 (case) but not rule 4 (expiry), so a
           fix that only uppercases the code passes the visible tests and fails the judge
checks:    pytest tests/test_reflexion.py: 5 passed
evidence:  HANDOUT Part 4; the comments above judge, write_lesson, reflexion
risk:      the lesson's tokens are not counted against the step or token budget, and
           whether a lesson is any good is only checked by me reading it

## Entry 6
artifact:  server.py at commit 810ebd3
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for help with Part 5 written from the server.py docstring;
           prompts/code-parts1-5.md
review:    checked that mcp.run is only reached through main() under
           if __name__ == "__main__", so the tests can import the file without starting
           the server. Read the tools/list reply from check-mcp and confirmed each
           description matches the docstring in agent/tools.py
checks:    pytest: 40 passed; python -m agent check-mcp: all checks passed
evidence:  HANDOUT Part 5; the server.py docstring
risk:      a relative --workspace is resolved from the repository root, not the folder
           the host starts in; only tested on WSL Linux so far, not yet from Claude Code

## Entry 7
artifact:  runs/runA, recorded run A, agent code at commit 810ebd3
tool:      my agent (python -m agent run runA), openai gpt-5.6-luna, 2026-10-30
prompts:   the default task, BASE_SYSTEM; prompts/runA-task.md
review:    read every diff before answering. Step 3: the first edit only uppercased the
           code (rule 2) and left last_day unused, so I rejected it with "Before
           editing, check every rule in spec.md against this function, not only the
           failing test." Step 4: the agent re-read discounts.py and its thought named
           "the currently unused expiration date". Step 5: checked the new diff against
           all five spec rules (empty code, case, unknown code, today > last_day so the
           last day itself still works, rounding down with //) and approved it
checks:    python -m agent check-trace runA: complete, 11 events, 9 tool calls;
           python -m agent replay runA: matches its trace, 11 events
evidence:  HANDOUT Part 6, Run A; RUBRIC B1, B2, B3
risk:      the expiry check was only tested by the judge, not by a visible test; the
           expired-code error message ("expired code: ...") is not checked by anything
dataset:   fixture/ unchanged (check-setup hash 8aaab9373de615a5), task "Make the
           tests pass."
result:    stopped done after 7 model calls, 9 tool calls, 13722 input and 693 output
           tokens, $0.0036; visible tests 4 of 4 passed at step 6; judge PASS, 6 of 6;
           runs/runA/trace.jsonl
changed:   in my practice run try1 I rejected the first edit with a comment request,
           the agent fixed only rule 2, and the judge failed rule 4. For runA I gave a
           reason that asked it to check the whole spec instead, and it found the
           expiry rule itself

## Entry 8
artifact:  runs/runB, recorded run B, agent code at commit 810ebd3
tool:      my agent (python -m agent run runB --tools mcp --judge), openai
           gpt-5.6-luna, 2026-10-30
prompts:   the default task and BASE_SYSTEM; no rejection reasons; prompts/runB-task.md
review:    the tools came from server.py over stdio. Step 1 had no thought before
           list_files, although BASE_SYSTEM asks for one; the prompt is a request, not
           a lock. Step 2 read discounts.py, spec.md and the test file and ran the
           tests in one turn. Step 3: checked the edit against all five spec rules (it
           had both the case fix and the expiry check) and approved it. The judge
           passed attempt 1, so no lesson was written and NOTES.md stayed empty
checks:    python -m agent check-trace runB: complete, 9 events, 7 tool calls;
           python -m agent replay runB: matches its trace, 9 events
evidence:  HANDOUT Part 6, Run B; RUBRIC B1, B2, B4
risk:      Reflexion's lesson path was never exercised in a recorded run, only in
           tests/test_reflexion.py with a scripted model
dataset:   fixture/ unchanged (check-setup hash 8aaab9373de615a5), task "Make the
           tests pass."
result:    1 attempt; stopped done after 5 model calls, 7 tool calls, 7923 input and
           434 output tokens, $0.0021; visible tests 4 of 4 at step 4; judge attempt 1
           PASS, 6 of 6; no lesson; runs/runB/trace.jsonl
changed:   nothing needed changing in the agent. Because the agent read spec.md before
           its first edit, I did not need a rejection this time. I noted the missing
           thought at step 1 for the report

## Entry 9
artifact:  runs/vendor, the Claude Code fixer agent (.claude/agents/fixer.md) on the
           same bug; evidence/fixer.md
tool:      Claude Code v2.1.314, model Claude Opus 5.5, 2026-10-30
prompts:   "Make the tests pass in runs/vendor/workspace." with the fixer file from
           HANDOUT Part 7C, used unchanged; prompts/fixer-task.md
review:    the first session (10:03 PM) ran in auto mode by mistake, so its edit and
           commands were approved by Claude Code, not by me. I read the edit afterwards:
           one line, code = code.upper(), in discounts.py, nothing in tests/. The fixer
           read spec.md on its own and said rule 4 was not handled, but kept to the
           smallest change as its instructions say. I switched to manual mode and gave
           the task again (10:21 PM); it ran the tests and changed nothing
checks:    python -m agent judge vendor: visible tests PASSED 4 of 4; judge FAILED 1 of
           6, test_rule_4_code_is_rejected_after_its_last_day; only discounts.py differs
           from fixture/; runs/vendor/judge.txt
evidence:  HANDOUT Part 7C; RUBRIC B7, C9
risk:      expired codes are still accepted in runs/vendor/workspace; the fixer's work
           was only checked by the judge because I ran it, the vendor's loop never runs it

## Entry 10
artifact:  REPORT.md
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-30
prompts:   asked for a draft of each section from my runA, runB and vendor results and
           evidence files; prompts/report.md
review:    checked every step number and quoted output against runs/runA/trace.jsonl,
           runs/runB/trace.jsonl, runs/vendor/judge.txt and evidence/; filled in the
           Section 5 token counts from the trace myself; edited wording
checks:    each section within its word limit; Section 1 pasted from python -m agent summary
evidence:  HANDOUT Part 8; RUBRIC C1 to C9
risk:      Section 6's word count depends on whether the quoted description counts

