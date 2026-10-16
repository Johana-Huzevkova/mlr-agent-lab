# snake/ — task notes

Task: a complete Snake game as a single file, `snake/index.html`, that runs in a standard
browser by double-clicking it — no server, no package installation, no internet connection.

Raw check output: [`CHECKS.md`](CHECKS.md). How to run the game and the tests:
[`README.md`](README.md).

## Delivered

| File | What it is |
|---|---|
| `index.html` | The whole game: markup, CSS and JavaScript in one file. Nothing else is needed to play. |
| `tests/run_game_tests.html` | The test suite. Loads the real `index.html` in a `file://` iframe and asserts against the live game. 35 checks. |
| `tests/probe_raf.html` | Throwaway environment probe. Kept because it explains a non-obvious test setting — see "Headless browsers starve rAF". |

## Requirements, and how each is met

| Requirement | How | Verified by |
|---|---|---|
| Single file | All HTML, CSS and JS inline. A scan for `http://`, `https://`, `src=`, `<link>`, `@import`, `fetch(`, `XMLHttpRequest`, `WebSocket`, `integrity=`, `cdn` returns exactly one line: `<link rel="icon" href="data:,">`. That is intentional and is not a network reference — it is an empty `data:` URL, which stops the browser requesting a `favicon.ico` that does not exist (otherwise the one offline-ish request a `file://` page makes is a guaranteed miss). | scan recorded in `CHECKS.md` §1 |
| Opens by double-click | Served as a top-level `file://` document, which is what a double-click does. | `CHECKS.md` §3 |
| No server | The tests load it over `file://` too, including the harness→game frame access. | `CHECKS.md` §4 |
| No install | The game uses no libraries. The tests use no test runner — just a browser you already have. | `CHECKS.md` §4 |
| No internet | Same as single-file, plus a live console capture showing no failed requests. | `CHECKS.md` §3 |

## Engine decisions

- **Grid is 24 × 18.** Canvas draws it, CSS scales the canvas to the container, so the board
  fits any window without changing the playfield. Cell size is recomputed on resize with
  `devicePixelRatio` applied to the backing store, so it stays sharp when the window changes.
- **Tick length starts at 140 ms** and drops 8 ms every 4 apples, floored at 62 ms. Speed
  rises gradually rather than in big jumps, and the floor keeps late game playable.
- **A direction buffer, two entries deep.** Pressing left-then-up within one tick should land
  both turns on consecutive ticks — a one-deep buffer loses one of them under any input burst,
  which reads as dropped inputs. The buffer is capped at two so a stuck key cannot queue up an
  unbounded path.
- **Straight reversals are refused, not executed.** Moving right and pressing left would put
  the head into the neck. It is ignored rather than treated as a death, because that keypress
  is nearly always the player's mistake rather than an intended move.
- **The tail cell is legal to enter** on a step that does not grow, since it vacates on that
  same step. Getting this wrong makes tight coils die for no visible reason. Test 16 pins it.
- **`localStorage` is wrapped in `try/catch`, both ways.** Some browsers treat each `file://`
  page as an opaque origin, where touching `localStorage` throws outright — so this is a real
  path on a double-clicked file, not a hypothetical. Best score falls back to living only in
  the session, and the game is otherwise unaffected. Test 21 pins it.
- **Best score is combined with `max()`, never assigned.** If storage is unavailable, a plain
  assignment would let a reset *lower* the best score you had earned in that session.
- **Walls are a toggle** (off by default, i.e. the edge kills; on, the snake tunnels). The
  board frame is drawn dashed when tunnelling so the rule in force is visible.
- **Growth per apple is selectable, 1/2/3** (1 = classic). This was added because a
  single-file Snake gets old in about a minute, and it is one number in the engine rather than
  a feature. At growth > 1 the extra segments stack on the head cell — documented, because the
  test suite has to skip its duplicate-cell check for that setting.
- **Win state exists.** If the snake occupies every cell, the game ends as a win rather than
  hanging or respawning an apple on top of the snake. Test 22 pins it, and it is the check that
  found the `placeApple` bug below.
- **Sound is WebAudio, built on the first gesture**, and toggleable (`M`). Creating the
  `AudioContext` before a user gesture logs an autoplay warning in a real browser, so nothing
  is constructed until you press a key or a button. `audio.resume()` rejects when there is no
  audio device, and that rejection is asynchronous, so the surrounding `try/catch` cannot see
  it — it needs its own `.catch()` or headless runs collect unhandled rejections.

## Two real bugs the checks found

Both are worth keeping on record because neither was visible from reading the code.

**1. `placeApple()` freed the cell the head was standing on.**

The function built its set of occupied cells and then deleted the *old* apple's cell from it,
on the theory that cell was now free. Usually true. But `placeApple()` is called from inside
the eating branch, *after* the head has already moved onto that cell — so on a nearly full
board the apple could be placed on the snake's own head, and a genuinely full board never
reported a win (it kept finding that one "free" cell forever).

Found by check 22 (board-cleared win), which failed with `expected "win", got ""`. Fixed by
building the occupancy set from the snake alone.

**2. A `window.S` console alias destroyed the game's own state.**

The file declared its state as a top-level `var S`, and also defined `window.S = {…}` as a
handy console alias. In a classic script those are the **same property**: the alias replaced
the state object with a helper. First paint survived (it happens before the alias runs), then
every subsequent frame threw `Cannot read properties of undefined (reading 'x')`. The game was
dead on arrival on a real double-click.

Why the test suite missed it for so long: **every check started by calling `reset()`**, which
repopulated the ruined object before the first post-clobber paint. The bug only existed on the
path "browser loads page → paints" that no test took. Fixed in two steps:

- the whole script is wrapped in an IIFE, so no name is global at all and this class of
  collision cannot recur;
- check `00a` takes the page exactly as the browser left it and asks it to paint, with no
  `reset()` first.

`grep -n "^window\." index.html` should always show one assignment. If a future edit adds a
short global alias, that is the bug coming back.

## Headless browsers starve rAF (explains the odd test clock setting)

`tests/probe_raf.html` measured this in headless Edge with `--virtual-time-budget`:

```
raf=2 timers=220 wallms=3513 maxGapBetweenFrames=21ms
```

`requestAnimationFrame` fired **twice** in 3.5 seconds while `setTimeout` fired 220 times.
Headless Chromium does not meaningfully service rAF, so a game clocked purely by rAF looks
frozen to any automated check, and no test could observe the loop running.

Consequence for the game itself, which is the part that matters: the clock is
`Date.now()`-based and a `setInterval(…, 100)` fallback calls the same `tick()` the rAF loop
does. In a real browser rAF is running and the interval is a no-op (`tick()` early-returns
unless 80 ms have passed since the last advance); in a starved environment it carries the
game. This is also why `tick()` deliberately ignores the timestamp rAF hands it.

Consequence for the tests: the game advances in ~100 ms bursts there, so the autopilot sets
`stepMs = 130` to get one step per burst. At `stepMs = 20` each burst moved the snake ~4 cells
while the planner still planned one cell at a time, so it overshot every apple — score stuck at
0 across 800 steps. That was a harness artefact, not a game defect; the fix was on the harness
side.

## Test approach, and its limits

`tests/run_game_tests.html` loads `../index.html` **by URL into an iframe**, so the browser
parses the actual bytes of the game file — no copy of the game is embedded or rewritten in the
harness. Assertions then reach into that live document through `contentWindow`: real canvas,
real DOM, real `KeyboardEvent` and `TouchEvent`, real game loop. It needs
`--allow-file-access-from-files`; without it the cross-frame access is blocked and everything
reports `BLOCKED` rather than passing.

Coverage worth naming, because it is more than "does it load": a **BFS autopilot plays the game
using arrow keys only** — it reads the board and presses keys, never writing game state, so a
pass means loop + input + apples + growth + death handling all work together on a game the
tests did not drive. Check 33 samples state, HUD text and canvas pixels together while that
game plays. Check 29 places a known board and reads back the pixel at each cell centre, so a
renderer that draws segments at the wrong coordinates fails.

Deliberate limitation: the pixel position check calls `render()` before reading pixels,
because in this headless mode rAF barely runs (see above) and the canvas would legitimately lag
the state. It therefore proves *what* gets drawn, not *when*. `index.html` exposes `render` on
`__SNAKE__` for that reason.

Two more things this does **not** prove, stated plainly:

- It runs headless. No check here watches a window open, so the *appearance* of the game —
  spacing, colour balance, overlay legibility, the feel of the speed curve — is outside what
  the suite can evidence. That gap was closed separately and later: the owner opened the game
  in a browser on 2026-10-16 and confirmed it works correctly. True as of that date, and no
  replacement for repeating it after a visual change.
- Check 34 (`no uncaught errors`) is the only check that would notice a silent visual
  regression. A test suite cannot see the game — which is exactly why the manual check above
  is recorded rather than waved away as unrigorous.

## Commands actually run

See [`CHECKS.md`](CHECKS.md) for full output. Summary:

| What | Result |
|---|---|
| Suite in headless Edge, before fixes | 24/33 |
| Suite in headless Edge, after fixes | 33/34 → 34/34 → **35/35** |
| Suite twice in a row (flakiness) | 34/34 both times, identical |
| Top-level `file://` load, after IIFE fix | 0 uncaught errors (was 4) |
| Suite in headless Chrome | see `CHECKS.md` §6 |
| Mutation test: bug 2 put back on purpose | caught by check `00a`, nothing else — 34/35 |

The mutation test matters more than the passing count: a suite that never fails proves nothing
about itself. Restored from a byte copy afterwards and re-verified (`diff` clean, 35/35).

## Not done / open

- ~~Visual review~~ **done by the owner, 2026-10-16** — opened in a real browser and confirmed
  the game works correctly. Worth recording what this adds over the 35 automated checks: it is
  the only evidence about how the game *looks and feels* (spacing, colour, overlay legibility,
  and the feel of the speed curve), none of which the suite asserts. Nothing was flagged as
  wrong, so no follow-up is recorded here.
- **Browser coverage.** Only Chromium-family was tested (Edge, Chrome). Firefox and Safari are
  untested. The code sticks to long-standing APIs (`canvas 2D`, `addEventListener`,
  `requestAnimationFrame`, `ResizeObserver` behind a feature test, `KeyboardEvent.code`), so
  the risk is low, but low is not verified.
- **Touch and mobile layout** are implemented (swipe to steer, tap to start/pause, on-screen
  D-pad under `@media (pointer: coarse)`) and the swipe handler is unit-tested with synthetic
  `TouchEvent`s, but nothing has been tried on an actual touch device.
- `tests/probe_raf.html` and `tests/view.html` were scratch. `view.html` was deleted; the probe
  stayed. Nothing in the suite depends on either.
