# snake/ — CHECKS

Raw output of the checks run for this task, 2026-10-16. Decisions and interpretation:
[`NOTES.md`](NOTES.md). Working rules: [`../AGENTS.md`](../AGENTS.md).

Every block below is copied from the tool output of the command shown above it. Nothing here
is paraphrased, and anything I could not run is labelled as such.

Environment used:

| Tool | Version | How established |
|---|---|---|
| Microsoft Edge | 155.0.4283.45 | `(Get-Item …\msedge.exe).VersionInfo.ProductVersion` |
| Google Chrome | 155.0.8059.82 | `(Get-Item …\chrome.exe).VersionInfo.ProductVersion` |
| git | 2.43.0.windows.1 | `git --version` (recorded earlier in `../PLAN.md`) |
| uv | 0.13.1 | `"$LOCALAPPDATA/Programs/uv/uv.exe" --version` |

Note on how the versions were read: `msedge --version` / `chrome --version` **did not work
here** — with a fresh `--user-data-dir` the call hangs, and without one it prints
"opening in existing browser session" instead of a version. The file metadata is the reliable
route on this machine. This is the reason `AGENTS.md`'s "try `--version`" rule needed
adjusting for browsers specifically.

## §1 — no network references in the game

Pattern (ripgrep, case-insensitive) over `snake/index.html`:

```
https?://|src=|<link|@import|fetch\(|XMLHttpRequest|WebSocket|integrity=|cdn
```

Result — one hit, and it is not a network reference:

```
8:<link rel="icon" href="data:,">
```

That empty `data:` URL is deliberate: it stops the browser asking for a `favicon.ico` that
does not exist. Everything else the pattern can match is absent, so the game makes no request
of any kind.

## §2 — globals the game leaves on `window`

Pattern `^(window\.|…)` over `snake/index.html`:

```
579:window.addEventListener('pagehide', function () { clearInterval(clockId); });
658:window.addEventListener('resize', resizeCanvas);
674:window.__SNAKE__ = {
```

Two listener registrations and one property. The script is IIFE-wrapped, so no `var` becomes a
global — see the `window.S` bug in `NOTES.md` for why this line matters.

File sizes for reference: `index.html` 689 lines / 24,001 bytes;
`tests/run_game_tests.html` 878 lines.

## §3 — loads as a top-level `file://` document (the double-click case)

Command:

```bash
msedge --headless=new --disable-gpu --hide-scrollbars --no-first-run \
  --user-data-dir=<empty temp dir> --window-size=760,900 --virtual-time-budget=5000 \
  --enable-logging=stderr --v=0 \
  --screenshot=<out.png> --dump-dom "file:///D:/MLR/snake/index.html"
```

Captured page console messages (`INFO:CONSOLE`, browser-extension lines excluded):

```
INFO:CONSOLE:6150] "[ProtocolLaunch] handler constructed; wiring port events."
```

That single line comes from an installed browser extension, not the game — the game's own
`index.html` appears nowhere in it. Uncaught-error count in the same capture:

```
--- uncaught? ---
0
```

Before the IIFE fix the same command produced four lines of:

```
INFO:CONSOLE:409] "Uncaught TypeError: Cannot read properties of undefined (reading 'x')",
  source: file:///D:/MLR/snake/index.html (409)
```

— which is how bug 2 in `NOTES.md` was found in the first place, and is the only reason the
"before" state is known.

A screenshot was written (73,550 bytes) but **not reviewed**: reading it back was blocked by
this session's working-directory permissions, and you declined the run that would have written
it into the repo. The game's appearance is therefore unverified.

## §4 — the suite, headless Edge, final state of the file

Command:

```bash
msedge --headless=new --disable-gpu --virtual-time-budget=60000 \
  --allow-file-access-from-files --user-data-dir=<empty temp dir> \
  --dump-dom "file:///D:/MLR/snake/tests/run_game_tests.html"
```

Report extracted from the dumped DOM (`<pre id="snake-test-report">`):

```
game frame  : file:///D:/MLR/snake/index.html
PASS  00a a freshly loaded page paints before any test calls reset
PASS  01 the real snake/index.html loaded in a file:// frame and started
PASS  02 the page really has the game: canvas, 2D context, HUD
PASS  03 fresh game: ready, 3 segments, apple on a free in-bounds cell
PASS  04 one step moves the head one cell and keeps the length
PASS  05 arrow key turns the snake
PASS  06 WASD turns the snake too
PASS  07 two turns queued between steps both land, in order
PASS  08 eating: +1 score, +1 segment, apple relocated
PASS  09 growth per apple is honoured (3)
PASS  10 the growth control drives the engine through the DOM
PASS  11 the apple never lands on the snake, over many board layouts
PASS  12 speed rises every SPEED_EVERY apples and the tick shortens
PASS  13 walls on: the edge is fatal
PASS  14 walls off: the same move crosses to the far edge
PASS  15 biting a mid-body segment is fatal
PASS  16 moving into the cell your tail just vacated is safe
PASS  17 a straight reversal is refused, not turned into suicide
PASS  18 pause freezes movement and the clock; resume continues
PASS  19 death is final until you start a new game
PASS  20 the best score survives a reset; the run score does not
PASS  21 localStorage is not required to play (file:// safe)
PASS  22 board-cleared win fires when no cell is free
PASS  23 keyboard ownership: game keys consumed, control keys left alone
PASS  24 the buttons drive the game
PASS  25 the overlay mirrors the state
PASS  26 the death message says what happened
PASS  27 on-screen score follows the engine when the HUD refreshes
PASS  28 touch swipe turns the snake
PASS  29 the renderer paints each segment and the apple at its own board cell
PASS  30 the game advances by itself once running (real rAF loop)
PASS  31 the loop really stops on pause and starts again on resume
PASS  32 autopilot plays with arrow keys only and reaches a score of 4
PASS  33 state, HUD and the painted board stay consistent during a self-played game
PASS  34 no uncaught errors or rejected promises in the whole run
TOTALS 35/35 PASS
```

This block was diffed against the captured output after writing, and matches line for line
(36 result lines). One line had been mistyped during transcription and was corrected from the
capture, so treat the file, not this note, as the record.

Two consecutive runs, immediately before the IIFE change, to look for flakiness — the
autopilot and timing checks depend on wall-clock behaviour:

```
run 1 exit=0 … TOTALS 34/34 PASS
run 2 exit=0 … TOTALS 34/34 PASS
```

(34 rather than 35 because check `00a` did not exist yet.)

## §5 — progression, so the passing count means something

The suite did not go green on the first attempt. Failures found and fixed, in order:

| Run | Result | What was wrong |
|---|---|---|
| 1 | 24/33 | check 22 exposed the real `placeApple` bug (bug 1 in `NOTES.md`); checks 15/16 were bad fixtures in the tests; 29–32 failed because headless rAF never runs (`NOTES.md`, "starved rAF") |
| 2 | 30/33 | `beep()` produced an unhandled rejection; the autopilot ignored wrap distance; check 31 could pass vacuously |
| 3 | 31/33 | the autopilot replanned every 25 ms while the starved clock stepped ~4 cells per 100 ms burst, so it overshot every apple |
| 4 | 33/34 | check 34 resolved `undefined` on success and the runner treats anything but `'ok'` as failure — a harness bug, not a game bug |
| 5 | 34/34 | check 33 stopped after 20 samples, before enough steps had happened |
| 6 | **35/35** | after the IIFE fix and check `00a` |

## §6 — same suite, headless Chrome, current file

Command as §4 with `chrome.exe` and a fresh profile:

```
exit=0
game frame  : file:///D:/MLR/snake/index.html
TOTALS 35/35 PASS
```

All 35 checks passed with the same check names as §4. An earlier Chrome run passed 34/34, but
that was the pre-IIFE file — this §6 run is the one that covers the current file.

## §7 — does the suite actually catch the bug it was extended for?

With `index.html` deliberately broken by re-inserting a state-clobbering line immediately
before `window.__SNAKE__ = {`:

```
FAIL  00a a freshly loaded page paints before any test calls reset  -- Cannot read properties of undefined (reading 'x')
TOTALS 34/35 PASS
```

Exactly one check failed — the new one — which confirms both the diagnosis in `NOTES.md` and
that `00a` has teeth. The other 34 stayed green, which is the point: they were green *with the
bug present*, which is precisely how the bug escaped in the first place.

Restored from a byte copy and verified:

```
restored; sanity re-run:
TOTALS 35/35 PASS
--- diff vs good copy (should be empty) ---
identical
```

## §8 — rAF starvation measurement

`tests/probe_raf.html` counted rAF callbacks against `setTimeout` callbacks under
`--virtual-time-budget=12000` in headless Edge:

```
raf=2 timers=220 wallms=3513 maxGapBetweenFrames=21ms
t=1000ms: raf=2 timers=64 elapsed=32 wall=1009ms
paused +1000ms (elapsed should be unchanged: 32): raf=2 timers=220 elapsed=32 wall=3513ms
```

Two rAF callbacks in 3.5 s. This is why the game carries a `setInterval` clock fallback and why
the autopilot uses `stepMs = 130`.

## §9 — not run

- **Firefox / Safari.** Not installed on this machine; not tested. Untested.
- **A real touch device.** Swipe handling is tested with synthetic `TouchEvent`s only.
- **Visual review by a human — done, but not by the agent and not via the screenshot.** The
  owner opened the game in a browser on 2026-10-16 and confirmed it works correctly. The PNG
  this run produced was still never opened by anyone; the human check happened directly on the
  page, which is better evidence anyway. See `NOTES.md` → "Not done / open".
- **`uv` / `pytest` / `ruff`.** Nothing in this task is Python, so `AGENTS.md`'s `uv` rules did
  not come up: no package was installed, no interpreter was configured. The one Python helper
  drafted along the way was discarded before it ran, at your instruction.
