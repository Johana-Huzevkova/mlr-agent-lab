# snake/ — the game

A complete Snake game in one file: [`index.html`](index.html).

## Play it

**Double-click `index.html`.** That is the whole procedure — no server, no install, no internet
connection, nothing else to open. It works straight out of the file.

| Control | Action |
|---|---|
| Arrow keys or `W` `A` `S` `D` | steer |
| `Space` or `Enter` | start / pause / resume / new game |
| `P` | pause |
| `R` | new game |
| `M` | sound on, off |
| Swipe, or the on-screen D-pad | steer on a touch device |
| Tap the board | start / pause on a touch device |

`Walls` toggles the edge rule (on = the edge kills, off = the snake tunnels through and comes
back on the other side; the board frame is dashed while tunnelling). `Apples per step` sets how
many segments one apple adds — 1 is classic, 3 grows fast. Best score is kept in
`localStorage` when the browser allows it, and for the session when it does not.

## Verify it

You need a Chromium browser you already have (Edge or Chrome). No test runner, no packages.

```bash
# Windows, from anywhere — Edge:
"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" ^
  --headless=new --disable-gpu --virtual-time-budget=60000 ^
  --allow-file-access-from-files ^
  --user-data-dir="%TEMP%\snake-profile" ^
  --dump-dom "file:///D:/MLR/snake/tests/run_game_tests.html"

# The report is in the dumped HTML, in <pre id="snake-test-report">:
#   ... --dump-dom "..." | findstr /C:"TOTALS"
```

Expected: `TOTALS 35/35 PASS`. Read the report, not the exit code — `--dump-dom` exits 0 even
when the page threw.

What it covers, how it was run, every failure found along the way, and what it does **not**
prove: [`NOTES.md`](NOTES.md) and [`CHECKS.md`](CHECKS.md).

## Files

| File | Purpose |
|---|---|
| [`index.html`](index.html) | The game. Everything it needs is inside this file. |
| [`tests/run_game_tests.html`](tests/run_game_tests.html) | The 35 checks. Loads the real `index.html` in a `file://` iframe and asserts against the live game. |
| [`tests/probe_raf.html`](tests/probe_raf.html) | Scratch probe. Explains one odd test setting; not part of the suite. |
| [`NOTES.md`](NOTES.md) | Design decisions, the two real bugs the checks found, known limits. |
| [`CHECKS.md`](CHECKS.md) | Raw command output for every claim made here. |

## Engine, briefly

Fixed 24 × 18 grid, drawn on a canvas that scales to the window. The snake moves on a timer
that starts at 140 ms per step and shortens as the speed level rises. Turns queue two deep, so
a quick left-then-up lands as two turns instead of losing one. Growing by one segment per
apple is the default; the score is the number of apples, and the best one survives a restart.

The whole script is wrapped in a function — see the `window.S` story in `NOTES.md` for the
reason, which is a real bug rather than style.
