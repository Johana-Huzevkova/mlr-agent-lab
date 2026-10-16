# PLAN.md

Living record of where this project stands. Update it in the same change as the work.
Working rules: [`AGENTS.md`](AGENTS.md). Run log: [`RUNS.md`](RUNS.md).

## Current state (updated 2026-10-16)

Repository is at the **"first task delivered"** stage: documentation, plus one working,
tested artifact in `snake/`.

- **`snake/`** — a complete Snake game as a single `index.html`, playable by double-clicking
  the file with no server, install or network. Verified by a 35-check browser suite run over
  `file://` in headless Edge 155 and Chrome 155 — both **35/35**. Two real bugs were found by
  those checks and fixed. Decisions in `snake/NOTES.md`, raw output in `snake/CHECKS.md`,
  how to play and how to re-run the checks in `snake/README.md`.
- No Python exists in this repository, so the `uv` rules have not yet applied to anything.
  No package was installed and no interpreter was configured during this task.
- **Manually verified by the owner in a real browser** on 2026-10-16: the game works
  correctly. This is the check no automated run could substitute for, and it closes the
  "never looked at by a human" gap recorded during the run.
- **Still unverified:** Firefox / Safari (not installed here) and a real touch device.

Everything below describes the 2026-10-14 setup stage that preceded it.

- Remote `origin` → `https://github.com/Johana-Huzevkova/mlr-agent-lab.git` (the owner's
  existing public repository — no repository was created by any agent).
- Branch `main`. This commit contains only the documentation skeleton. Verify the current
  state with `git log --oneline -3` and `git status -sb`; this file deliberately does not
  paste in its own commit hash or push result, because it could not know them.
- Before this work the working tree contained only `.git/` (no `README.md`, no `PLAN.md`, no
  source, no `.gitignore`), and the remote had **zero refs** (`git ls-remote origin` returned
  nothing), so nothing existing was overwritten locally or remotely.

## Completed — Snake task (2026-10-16)

- [x] `snake/index.html` — complete game, one file, no dependencies, no network references
      (scanned: the only match for any URL/loader pattern is an empty `data:` favicon).
- [x] Playable by double-click — verified as a top-level `file://` document with zero uncaught
      console errors, which is the case a test iframe does **not** reproduce.
- [x] `snake/tests/run_game_tests.html` — 35 checks against the live game over `file://`,
      including a BFS autopilot that plays using arrow keys only and reaches score 4.
- [x] Verified in two browsers: headless Edge 155.0.4283.45 **35/35**, headless Chrome
      155.0.8059.82 **35/35**; twice in a row in Edge to look for flakiness.
- [x] Two real bugs found by those checks and fixed: `placeApple()` could put the apple on the
      snake's own head (breaking the win condition), and a `window.S` console alias clobbered
      the game's state (`var S` at top level *is* `window.S`), killing the game on a real
      double-click. Fixed by IIFE-wrapping the script.
- [x] Mutation-checked: the bug was put back deliberately and exactly one check — the new
      regression guard — failed. Restored from a byte copy, `diff` clean, 35/35 again.
- [x] `snake/README.md`, `snake/NOTES.md`, `snake/CHECKS.md` — how to play, why the engine is
      shaped as it is, and the raw output of every check.
- [x] Opened and confirmed working by the owner in a real browser (2026-10-16) — the one check
      no automated run produces evidence for.
- [ ] Not done: Firefox / Safari (not installed here); a real touch device.

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
| Microsoft Edge | 155.0.4283.45 | added 2026-10-16. Used to verify `snake/`. `msedge --version` hangs with a fresh profile — read `(Get-Item …).VersionInfo.ProductVersion` instead |
| Google Chrome | 155.0.8059.82 | added 2026-10-16. Same caveat about `--version` |

## Next step

1. **Confirm the project goal** — what "MLR" means and the first real objective, then
   replace the placeholder paragraph in `README.md`.
2. **Put `uv` on PATH** (or standardise on the full path) so `uv run` works from a fresh
   shell, and re-verify with `uv --version` in a new terminal.
3. **Initialise the Python project** with `uv init` (target Python 3.12, which is installed)
   → `pyproject.toml`, then `uv add --dev pytest ruff` and add a first smoke test so
   `uv run pytest` is a real, meaningful check. (Still outstanding: nothing in `snake/` is
   Python, so `pytest` remains un-run and the repo still has no Python environment.)
4. **Commit `snake/`.** As of this edit it is still uncommitted: `git status --short` shows
   ` M PLAN.md`, ` M README.md`, ` M RUNS.md` and `?? snake/`, and the agent's `git add` was
   blocked by the permission layer, so staging and committing fell to the owner. After the
   commit, confirm it landed rather than trusting this file — `git log --oneline -2`,
   `git status -sb`, `git ls-remote origin`. This file sits inside the commit it would
   describe, so it lists commands instead of claiming an outcome (same reasoning as Run 2).
   To re-verify the game itself: `snake/README.md` → "Verify it".

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
  - Checked for the `snake/` files on 2026-10-16: a scan for the local account name and
    `C:\Users\…` paths across `snake/`, `README.md` and `PLAN.md` returns nothing (browser
    paths are written as `%TEMP%` / `…\` placeholders), and a secret-pattern scan is clean.
    The browser profile directories the test commands use are placeholders in the docs, not
    real ones.
- **`snake/` was reviewed by a human.** The owner opened the game in a browser on 2026-10-16
  and confirmed it works correctly. Nothing about the appearance was flagged as wrong, so
  nothing here contradicts that; if you later spot something, it belongs in `snake/NOTES.md`
  as a finding no automated check produced.

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
- 2026-10-16 — First code task: `snake/` — single-file Snake game (`index.html`), 35-check
  browser test suite, `README.md` / `NOTES.md` / `CHECKS.md`. Verified over `file://` in
  headless Edge and Chrome (35/35 each). Two real bugs found and fixed by those checks; the
  fix for the second was confirmed by re-introducing it and watching exactly one check fail.
  Root `README.md` and `PLAN.md` updated to match reality (the "no code yet" claim was made
  false by this task). Nothing committed. Full narrative in `RUNS.md` → Run 3.
