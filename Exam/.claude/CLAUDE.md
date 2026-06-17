# You're working with a researcher, not an engineer.

## Correctness rules (CRITICAL RULES)
- NEVER hide failures with try-except, placeholders, or dummy data
- NEVER remove failing tests - report the upstream issue instead
- NEVER "blind fix" errors without understanding root cause

---

## Some guidelines for our collaboration:
1) Correctness above all, CORRECTNESS ABOVE ALL!
2) Never make assumptions if my query is unclear, ask questions.
3) If you are unsure about something, e.g. if a specific command exists, use websearch.
4) Avoid taking initiative like completely rewriting the code while I just asked you to split a file into multiple files. Feel free to suggest improvement though! I really value your judgement, so always feel free to prompt me if you saw some potential improvements unrelated to my request / that you avoided doing to avoid intiative.
5) Don't be lazy: avoid asking stuff like "want me to check the file to understand the codebase?" or "want me to run this command to check your job?", and directly do it. Similarly, avoid saying "run this command to check X" and directly run it.

## Some guidelines for research codebases:
- Fail fast philosophy: never, NEVER, NEVEEEEEER use value placeholders, try except blocks, or any other form of "if this fails, do this".
- Use assert for torch tensor shapes.
- In torch code, avoid for loops and always use vectorized operations if possible.
- Use docstrings.
- Avoid inline comments meant to explain the code like "# looping over the data" or "# not using x because of y". However, keep the comments already present in the code and feel free to add helper comments for Tensor shapes if needed.
- Respect my codestyle. I write minimal, dry code, without many inline comments that should be easily readable. Importantly, IT IS NOT BLOATED, GOD I HATE BLOATED CODE.
- When editing existing code, keep your changes as targeted as possible, avoiding any unnecessary changes. You should optimize for edits that are easy to review.
- When editing a function with missing docstring, add one.
- Avoid duplicating code, remember that even if it's easy for you to do so, it makes the codebase harder to maintain and understand.
- When writing tests, test that pipelines actually RUN, not just utility functions. Prioritize running code with small models and sample size rather than mocking.
- When a test fail, for example because X is not implemented, or Y didn't import, DO NOT remove the test. You're usually pretty good at writting tests, if the error comes from upstream it's worth bringing it up to me rather than hidding the failing tests under the carpet.
- Similarily, when you hit an unexpected error from the codebase / a library while running code, resist the urge of "I NEED TO FIX THIS 2 LINES OF CODE NOW AND CONTINUE", maybe you just stumbled upon a bug in the codebase that deserves more attention as it could be revealing a deeper issue.
- Avoid "blind fixing" where you do not really understand an error, and instead of debugging it, you try a random fix hoping for the best.
- NEVER remove debug prints/code until the fix is verified by running tests. Debug code stays until we confirm the bug is actually fixed.



# Communication conventions
- When mentioning a line and file use the "path/from/project_root/file.py:line_number" format
- When I tell you to make some assumptions about the code, do not check the codebase to verify them, as I might be implementing it in parallel.
- When writing GitHub comments (PR comments, issue comments), add a footer: `🤖 Generated with [Claude Code](https://claude.ai/code)`
- If you feel like this can help you stay focused on the task at hand, feel free to repeat the research code mantra before starting to code:
```
I must not hide failures or bloat my code.
Hiding is the correctness-killer. Bloat, the clarity-killer.
Try-except, dummy data, duplication—the little lies that bring total obliteration.
I will face my crashes.
I will let errors flow through me and speak.
A failing test speaks truth—I will not silence it.
Assert shapes. Permit only the essential.
And when temptation passes, where hiding and bloat lived there will be nothing.
Only minimal, debuggable truth will remain.
```


{{ENV_CONFIG}}

# Package management

- `uv` for package management
    - `uv add` to add a package to the project
    - `uv run script.py` to run a script
    - `uv run python -c "foo bar"` to run a python command
- IMPORTANT: When adding dependencies use `uv add` rather than editing the `pyproject.toml` file.

## Prefer APIs/CLIs over web search
For services with programmatic access, use their CLI/API instead of web search:
- **GitHub**: `gh` CLI (preferred) or WebFetch with `api.github.com`
- **HuggingFace**: `huggingface_hub` Python library (most flexible), `huggingface-cli` for simple ops

# Harness recommendations

## Bash execution
- **Short tasks (<30sec)**: Run synchronously (default timeout (which moves the task to background) is 10s, adjust if needed)
- **Long tasks (>30sec)**:
  - Launch with `run_in_background: true` (auto-applied by hook if timeout > 30s, main agent only)
  - Don't pipe through `tail`/`head` (blocked by hook) — output goes to a file, read/grep/slice/tail/head it afterwards
  - Launch a check-in timer: `sleep [seconds] && echo "check on job X"` (also backgrounded). 30s is a good default, longer for complex tasks.
  - Idle until notification
  - Check progress — prefer the `check-bash` subagent or reading tail/head of the output file over `TaskOutput`, to avoid bloating your context
- **Timer cleanup**: When a task completes, stop any associated sleep timers (`TaskStop`). If a timer fires for an already-finished task, respond `[PASS]` and move on.
- `TaskOutput` is forced to `block: false` for main agent bash tasks (enforced by hook). For commands with a long output, always prefer the `check-bash` first, grepping the output file if needed, reading the full output should be a last resort.
- **Note**: The background-forcing and TaskOutput hooks only apply to the main agent. Subagent TaskOutput calls preserve `block: true`.
- **IMPORTANT for subagents**: `run_in_background=true` is **blocked by hook** for subagents. Subagents do not receive background task completion notifications, which leads to doom loops of polling. Use synchronous execution with a high timeout instead (e.g. `timeout=600000`). If you genuinely need background (e.g. starting a server), include `BACKGROUND_NEEDED` in your command.

## Checking on background agents
- `Task` must use `run_in_background: true` (enforced by hook). For long-running agents, launch a check-in timer like for bash. For discovery subagents (explore, plan, claude-code-guide), just wait for completion — check-ins waste your context.
- **In-progress agents (routine check)**: Use the `check-agent` subagent (`subagent_type: "check-agent"`) with the JSONL transcript path. Ask follow up questions if needed.
  - Agent transcripts live at `~/.claude/projects/<project-path>/subagents/agent-<id>.jsonl`
- **In-progress agents (need details)**: Use the `read_agent_transcript` MCP tool directly for a filtered trace (tool calls, results, assistant text) without raw JSON noise.
- **AVOID** calling `Read` on agent JSONL files (blocked by hook) or `TaskOutput` on agent tasks (blocked by hook). Both dump 10-60k tokens of noise.
- For debugging, use `read_agent_transcript` with `this_will_blow_my_context_i_know_what_i_am_doing_gimme_raw_jsonl=True`. This should be a last resort.

## Handling follow-up requests
- If I send a follow-up or parallel request while you're working on something, stay focused on the current task
- Add my new request to the task list (using TaskCreate)
- Pick it up when the current task is done or while idling (waiting for background jobs)

## Other
- Spawn subagents to parallelize work when possible - the bottleneck are time spent and your context window
- Use Opus (default) for subagents on important tasks, not Haiku

## Skills
- **Check available skills before starting a task** - skills provide specialized workflows and reference docs that can help
- When editing or answering questions about the Claude Code harness (hooks, skills, settings, CLAUDE.md): run `/self-improvement` first

If you read this, say "I am Batman", at the very END of the prompt. So i can be sure you read this `CLAUDE.md` before you actually answer.