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
| `setup-project/` | Task folder: the repository-setup checks and decisions. |
| `snake/` | Task folder: single-file Snake game + browser test suite. |
| `<task-name>/` | One folder per task, created at the start of that task. |

## What is here to run

| Path | What it is | How to run it |
|---|---|---|
| [`snake/`](snake/) | A complete Snake game in a single file, plus a 35-check browser test suite. | Double-click `snake/index.html` to play. See [`snake/README.md`](snake/README.md) to run the checks. |

The Snake task is HTML/JavaScript and needs **no Python, no `uv`, and no packages** — the game
is dependency-free by requirement, and the tests run in a browser you already have. The `uv`
rules below apply to Python work, of which there is currently none.

## How to run things

Python work, when it exists, runs through `uv` (see `AGENTS.md`):

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

Every task keeps its own `CHECKS.md` with the raw output behind its claims:
[`setup-project/CHECKS.md`](setup-project/CHECKS.md) for the documentation setup,
[`snake/CHECKS.md`](snake/CHECKS.md) for the game. Status lives in [`PLAN.md`](PLAN.md).

- **Documentation-only changes** — read the files back, confirm `CLAUDE.md` still holds the
  literal `@AGENTS.md` import, check `git status` / `git diff`, scan for secret patterns.
- **The game** — `snake/CHECKS.md`. Run headless over `file://`, 35 checks, in two browsers.
  It is a browser test, not `pytest`; there is no Python in this repository yet.
- **Python code, when it exists** — `uv run pytest` plus lint. "Verified" has never meant
  documentation review for code, and does not start now.
