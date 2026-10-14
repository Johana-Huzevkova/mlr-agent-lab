# setup-project — notes and decisions

Task: create the repository's agent working rules and documentation skeleton.
Verification output lives in [`CHECKS.md`](CHECKS.md); status lives in
[`../PLAN.md`](../PLAN.md).

## Decisions taken

1. **Rules live in `AGENTS.md`, and `CLAUDE.md` only imports them.** One source of truth;
   other agents read `AGENTS.md` directly. The exact line `@AGENTS.md` is the first line of
   `CLAUDE.md` so Claude Code expands it.
2. **One folder per task, starting with this one.** `AGENTS.md` states the rule; this folder
   demonstrates it. A `.gitkeep` was added while the folder was empty so it would be visible
   in `git status`, then deleted once `CHECKS.md` and `NOTES.md` existed — an empty placeholder
   next to real files is just noise.
3. **`.gitignore` added even though it was not in the request.** The task asked to "ensure
   `.claude`, `.venv`, and other private files are excluded where appropriate", and an empty
   repo had no ignore file at all. It is the only file here that is not explicitly listed in
   the request — easy to drop if unwanted.
4. **`.claude/` is ignored wholesale.** It holds this user's local agent state
   (`settings.json` with an auth token, session transcripts, history). If you later want
   shared team settings or skills under `.claude/` in git, un-ignore those specific paths.
5. **`uv.lock` is tracked (not ignored).** Pinning is the point of using `uv`.
6. **The project goal is written as unverified rather than invented.** The repo was empty
   and "MLR" has no expansion anywhere on disk that I could find. `README.md` says so
   plainly and `PLAN.md` lists it as the next open item.

## What would make this setup actually testable

Right now the only meaningful checks are `git check-ignore`, a secret grep, and reading the
files. To get a real verification loop, the next task should:

```bash
uv init --python 3.12          # pyproject.toml + a package placeholder
uv add --dev pytest ruff       # tooling as project dependencies
uv run pytest                  # the first genuine, runnable check
uv run ruff check .
```

Then `README.md` documents those commands as *the* checks, and `PLAN.md` can record actual
pass/fail output instead of documentation review.

## PATH problem worth fixing first

`uv.exe` 0.13.1 exists at `%LOCALAPPDATA%\Programs\uv\uv.exe` but is not on `PATH`, and
`python`/`python3` do not resolve in this shell (only `py` does). Under the "try available
tools before asking me" rule: I located both binaries and confirmed the versions, but adding
a directory to your user `PATH` is a change to your machine outside this repository, so it is
flagged for you rather than silently performed. In PowerShell:

```powershell
[Environment]::SetEnvironmentVariable(
  "Path", $env:Path + ";$env:LOCALAPPDATA\Programs\uv", "User")
```

Then open a new terminal and confirm with `uv --version`.
