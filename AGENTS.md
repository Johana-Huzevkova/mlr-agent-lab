# AGENTS.md — Working Rules

Shared operating rules for AI agents (and humans) working in this repository.
`CLAUDE.md` imports this file, so one edit keeps every agent in sync.

## Project context — read first

1. **Read `README.md` and `PLAN.md` before starting each task.** They define the goal and
   the current state. Do not start work from memory alone.
2. **`RUNS.md`** records which model/endpoint ran which task and how the result was checked.
3. If the docs contradict reality, trust reality, fix the docs, and say so in your summary.

## Record work in the repository

- Keep decisions, progress, and next steps in repository files (`PLAN.md`, task folders),
  not only in chat. Chat history is not durable project state.
- **Start every new task by creating a folder named for that task** (e.g.
  `data-ingestion/`, `pilot-run/`). Put drafts, notes, scratch scripts, and intermediate
  outputs for that task inside it. Repository-level entry points (`README.md`, `PLAN.md`,
  `AGENTS.md`, `RUNS.md`) stay at the root.
- Write down the command you ran and its result, not just "it works".

## Tooling

- **Try the available tools before asking me to perform a technical step manually.** Attempt
  the command, read the error, check PATH and tool locations yourself. Only ask when you are
  genuinely blocked (missing credentials, an approval only I can grant, a decision only I can
  make).
- **Use `uv` for all Python work:**
  - `uv run <script.py>` / `uv run <tool>` to run things
  - `uv add <package>` to add dependencies (this edits `pyproject.toml`)
  - `uv sync` to install the pinned environment
  - **Never use global `pip install`.** Not `pip install`, not `python -m pip install`.
    If `uv` is missing, install or locate `uv` — do not fall back to global pip.
- **Record relevant tool versions and commands.** When you set up or change an environment,
  note tool versions (`uv --version`, `python --version`, ...) in `PLAN.md` or the task
  folder, and record how to run the code in `README.md`.
- Known local detail: on this machine `uv.exe` is installed at
  `%LOCALAPPDATA%\Programs\uv\uv.exe` but is **not on PATH**. Either add that directory to
  PATH, or invoke it by full path and record which one you used.

## Verification

- **Run meaningful checks before claiming completion**: the tests, the linter, the actual
  script, or a command that demonstrates the behaviour you claim. If a check is impossible,
  say which one you could not run and why — do not describe it as done.
- Report results honestly, including failures and partial results. Never present an unrun
  check as passing, and never invent output, versions, or file contents.
- Mark anything you could not verify as **unverified** rather than guessing.

## Tests and disagreements

- **Never weaken a test just to make it pass.** No lowering assertions, deleting cases,
  adding `xfail`/`skip`, or loosening thresholds to get green.
- **If you think a test or a requirement is wrong, explain the disagreement first** and wait
  for a decision. Changing behaviour and the test together, quietly, is not acceptable.

## Secrets and publishing

- **Never publish secrets or private data.** No API keys, tokens, `.env` files, credentials,
  or private datasets in commits, logs, `RUNS.md`, or anything pasted externally.
- **Ask me before publishing anything** — including `git push`, opening a pull request,
  deploying, releasing, or posting to an external service. Local file edits and local
  commands are fine; anything that leaves the machine needs approval first.
- Do not run `git add`, `git commit`, or `git push` unless I explicitly ask.

## Current environment (recorded 2026-10-14)

Verified with `--version` on this machine:

| Tool | Version | On PATH? |
|---|---|---|
| Claude Code CLI | 2.1.299 | yes |
| git | 2.43.0 (windows) | yes |
| gh | 2.102.0 | yes (`%USERPROFILE%\.local\bin`) |
| uv | 0.13.1 | **no** — `%LOCALAPPDATA%\Programs\uv\uv.exe` |
| Python (launcher `py`) | 3.14 (default), 3.12.1, 3.11 | `py` yes, `python` no |
| Node.js / npm | not installed | — |
