# mlr-agent-lab

A small laboratory repository for working with coding agents: reproducible environments,
explicit working rules, and a written trail of what was run and how it was verified.

> **Goal — to be confirmed.** The repository was created empty (no commits, no code) with
> the remote `github.com/Johana-Huzevkova/mlr-agent-lab`. What "MLR" stands for and the
> concrete first objective are **unverified** — they are the owner's to state. Until then
> this repo's purpose is: *a rules-driven, documented agent workspace*, and the first
> objective is the environment setup recorded in [`PLAN.md`](PLAN.md).

## Repository layout

| Path | Purpose |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Shared working rules for all agents (read first). |
| [`CLAUDE.md`](CLAUDE.md) | Imports `AGENTS.md` via `@AGENTS.md` for Claude Code. |
| [`PLAN.md`](PLAN.md) | Current progress, completed setup, next step. |
| [`RUNS.md`](RUNS.md) | Model/endpoint per run and how results were verified. No secrets. |
| `.gitignore` | Excludes `.venv/`, `.claude/`, `.env`, keys, caches, local state. |
| `<task-name>/` | One folder per task, created at the start of that task (e.g. `setup-project/`). |

## How to run things

There is **no code yet** — nothing to execute beyond the checks below. Once Python code
exists, all of it runs through `uv` (see `AGENTS.md`):

```bash
uv sync                      # install the pinned environment from pyproject.toml
uv add <package>             # add a dependency (edits pyproject.toml + uv.lock)
uv run <script.py>           # run a script inside the project environment
uv run pytest                # run tests
uv run ruff check .          # lint  (once ruff is added as a dependency)
```

Never `pip install` globally.

### Checks available right now

```bash
git status --short               # working tree state
git check-ignore -v .claude/ .venv/ .env   # confirm private paths are ignored (exit 0)
```

### `uv` is not on PATH on this machine

`uv.exe` 0.13.1 is installed at `%LOCALAPPDATA%\Programs\uv\uv.exe` but is not on `PATH`,
and `python`/`python3` resolve to nothing in the Git Bash shell (only the `py` launcher
works). Either add that directory to `PATH`, or call it by full path:

```bash
"$LOCALAPPDATA/Programs/uv/uv.exe" --version
```

## Verification

Documentation-only changes are verified by reading the files back, confirming `CLAUDE.md`
contains the literal `@AGENTS.md` import line, checking `git status` / `git diff`, and
running a secret-pattern scan over every file in the repository. Raw output for the current
set of checks is in [`setup-project/CHECKS.md`](setup-project/CHECKS.md); status lives in
[`PLAN.md`](PLAN.md). Once there is code, "verified" means `uv run pytest` (plus lint) —
not documentation review.
