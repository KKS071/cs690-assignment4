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
artifact:  Part 0 setup: .env and .gitignore
tool:      none (setup by hand), 2026-10-29
prompts:   none
review:    TODO: my provider, model, and the two prices
checks:    python -m agent check-setup: TODO result
evidence:  HANDOUT Part 0
risk:      TODO

## Entry 2
artifact:  agent/tools.py at commit TODO-SHA
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for Part 1 written from the comments in the starter file; prompts/code-parts1-5.md
review:    TODO: what I read line by line, what I changed or rejected, and why
checks:    pytest tests/test_tools.py: 16 passed
evidence:  HANDOUT Part 1; the comments above each tool
risk:      TODO: what the public tests do not cover

## Entry 3
artifact:  agent/policy.py at commit TODO-SHA
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for Part 2 written from the comments in the starter file; prompts/code-parts1-5.md
review:    TODO: what I read line by line, what I changed or rejected, and why
checks:    pytest tests/test_policy.py: 7 passed
evidence:  HANDOUT Part 2; the comments above gate_for, budget_stop, is_stuck
risk:      TODO: what the public tests do not cover

## Entry 4
artifact:  agent/loop.py at commit TODO-SHA
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for Part 3 written from the comments in the starter file; prompts/code-parts1-5.md
review:    TODO: what I read line by line, what I changed or rejected, and why
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3; the comment above run_agent, steps 1 to 5
risk:      TODO: what the public tests do not cover

## Entry 5
artifact:  agent/reflexion.py at commit TODO-SHA
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for Part 4 written from the comments in the starter file; prompts/code-parts1-5.md
review:    TODO: what I read line by line, what I changed or rejected, and why
checks:    pytest tests/test_reflexion.py: 5 passed
evidence:  HANDOUT Part 4; the comments above judge, write_lesson, reflexion
risk:      TODO: what the public tests do not cover

## Entry 6
artifact:  server.py at commit TODO-SHA
tool:      Claude (claude.ai), model Claude Opus 5.5, 2026-10-29
prompts:   asked for Part 5 written from the comments in the starter file; prompts/code-parts1-5.md
review:    TODO: what I read line by line, what I changed or rejected, and why
checks:    pytest: 40 passed; python -m agent check-mcp: all checks passed
evidence:  HANDOUT Part 5; the server.py docstring
risk:      TODO: what the public tests do not cover

