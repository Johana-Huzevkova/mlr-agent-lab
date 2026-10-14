# PLAN.md

Living record of where this project stands. Update it in the same change as the work.
Working rules: [`AGENTS.md`](AGENTS.md). Run log: [`RUNS.md`](RUNS.md).

## Current state (2026-10-14)

Repository is at the **"configured shell, no code yet"** stage.

- Remote `origin` → `https://github.com/Johana-Huzevkova/mlr-agent-lab.git` (the owner's
  existing public repository — no repository was created by any agent).
- Branch `main`. This commit contains only the documentation skeleton. Verify the current
  state with `git log --oneline -3` and `git status -sb`; this file deliberately does not
  paste in its own commit hash or push result, because it could not know them.
- Before this work the working tree contained only `.git/` (no `README.md`, no `PLAN.md`, no
  source, no `.gitignore`), and the remote had **zero refs** (`git ls-remote origin` returned
  nothing), so nothing existing was overwritten locally or remotely.

## Completed setup

- [x] Inspected project state: empty repo, single remote, no commits, no prior docs.
- [x] `AGENTS.md` — working rules (read docs first, decisions in repo files, try tools
      before asking, `uv` only / never global pip, record versions and commands, run real
      checks, never weaken a test, no secrets, ask before publishing, one folder per task).
- [x] `CLAUDE.md` — contains the exact line `@AGENTS.md` plus Claude-specific pointers.
- [x] `README.md` — goal (partly unverified, see below) and how to run things.
- [x] `RUNS.md` — one entry per agent run: model/endpoint (redacted for the public repo, see
      open items), the request, and how the result was verified.
- [x] `.gitignore` — excludes `.venv/`, `.claude/`, `.env*`, keys/certs, caches, `*.db`,
      `data/private/`, editor and OS noise.
- [x] `setup-project/` — task folder for this task: `CHECKS.md` (raw output of every check)
      and `NOTES.md` (decisions taken and the PATH fix awaiting you).
- [x] Prepared for publication: remote verified, secrets scanned, identifiers redacted, owner
      approval obtained on the file list and diff. This is intended to be the first commit on
      `main`, pushed to the existing `origin`. Verify rather than trust: `git status -sb`,
      `git log --oneline`, `git ls-remote origin`.

### Tool versions recorded

Verified by running each `--version` on this machine (2026-10-14):

| Tool | Version | Notes |
|---|---|---|
| Claude Code CLI | 2.1.299 | on PATH |
| git | 2.43.0.windows.1 | on PATH. Commit identity for this repo is set **locally** to the owner's GitHub noreply address (`<numeric-id>+<login>@users.noreply.github.com`, via `git config user.email`) so the public history does not carry a personal email; global config untouched. `core.autocrlf=true`, but all files here are LF so it is inert |
| gh | 2.102.0 | `%USERPROFILE%\.local\bin\gh.exe` |
| uv | 0.13.1 | **not on PATH** — `%LOCALAPPDATA%\Programs\uv\uv.exe` |
| Python | 3.14 (default), 3.12.1, 3.11 | reachable via `py` launcher only; `python`/`python3` not on PATH |
| Node.js / npm | not installed | — |
| Quarto | installed at `...\Programs\quarto\bin` (version not checked) | on PATH |

## Next step

1. **Confirm the project goal** — what "MLR" means and the first real objective, then
   replace the placeholder paragraph in `README.md`.
2. **Put `uv` on PATH** (or standardise on the full path) so `uv run` works from a fresh
   shell, and re-verify with `uv --version` in a new terminal.
3. **Initialise the Python project** with `uv init` (target Python 3.12, which is installed)
   → `pyproject.toml`, then `uv add --dev pytest ruff` and add a first smoke test so
   `uv run pytest` is a real, meaningful check.
4. Only after that: the first feature task, in its own folder.

## Open items / needs review by the owner

- The intended meaning of **MLR** and the concrete project goal — unverified.
- `.gitignore` ignores `.claude/` (local agent state). If you want shared agent settings or
  skills under `.claude/` committed instead, say so and I will un-ignore specific paths.
- `uv.lock` is currently **not** ignored (recommended to commit it). Flip the comment in
  `.gitignore` if you prefer otherwise.
- The repo is **public**, so anything committed here is published. Before the first push the
  owner approved redacting the LLM gateway hostname / model aliases from `RUNS.md` and the
  local Windows username from `setup-project/CHECKS.md`. Keep that in mind for future files:
  no infrastructure identifiers, no local paths, no credentials.

## Log

- 2026-10-14 — Repository setup: `AGENTS.md`, `CLAUDE.md`, `README.md`, `PLAN.md`,
  `RUNS.md`, `.gitignore`, `setup-project/`. No code, no commits.
- 2026-10-14 — Publication pass: remote and pre-existing refs verified, secret scan re-run,
  gateway identifiers and the local username redacted with the owner's approval. **This file
  is part of the commit it describes**, so it states the commit message rather than a result:
  commit `Add project instructions and setup documentation` on `main`, pushed to the existing
  `origin`. Confirm the outcome independently with `git log --oneline -1`, `git status -sb`,
  and `git ls-remote origin` — none of which can be truthfully reported from inside the
  commit. Full narrative in `RUNS.md` → Run 2.
