# Flamey's Official Animation Studio: Blocks extension

One big HTML file (`Flamey_s_Official_Animation_Studio_extensions.html`, ~1.6 MB). All scripts are inline `<script>` tags. Joel (the owner) works in HTML5 canvas and browser formats and likes direct, short feedback.

## Where the studio lives
- Website (GitHub Pages, updates by itself from `main`): https://flameyter.github.io/Flamey-s-Animation-Studio/
- itch.io page: https://flameyter.itch.io/flameys-animation-studio (upload `itch/flamey-studio-itch.zip` once; it shows the website)
- Joel's YouTube channel: **Flamey :)**, https://www.youtube.com/@CrazyPWAF (about 2.65K subscribers). Videos made in the studio go here.

## Version
- The version is shown at the bottom of the main menu (`#studiover`, "Version 5.96"). It started at 5.7. Add 0.1 for a big round of new things and 0.01 for a round of fixes, and update it when pushing a round.

## Checking your work
- Syntax: pull the inline scripts out into one file and run `node --check` on it.
- Behaviour: Playwright (chromium) against the file. Call `bkEval({t:'blockname', v:[...]})` directly. To draw a block: `bkBlockEl({t, v}, {palette:true})` into a div, then screenshot.
- To open the real editor in a test: push `"blocks"` onto `EXI`, add a `<div class="v v-blocks">` next to `.v-edit`, call `go('blocks')`, then `document.querySelector('.bkpanel').__setFull(true)`. Stub `upgLocked` to return false.
- Loops only run when `playing` is true and `EXI` includes `"blocks"`.
- The network is blocked in the test sandbox. Mock `window.fetch` for the definition block (`lookdef`).

## How blocks work
- `BLK` defines each block: `{cat, shape, tpl, a, run, [col], [lazy], [loop]}`. `tpl` uses `%1 %2` for inputs. Shapes: `hat`, `cmd`, `cap`, `c` (one mouth), `c2` (two mouths, `because`), `c3` (three mouths, `trycatch`), `rep` (round), `bool` (pointed), `list` (square).
- `a` entries are `[type, default, options]` with types `num`, `str`, `text`, `bool`, `menu`.
- The tray order is the key order of `BLK` within each category. `BKCAT` holds the categories: watch, settings, operators, control, keep, commands, function, rigs, audio, looks, edit, variables.
- `BKS` is the list of scripts `{x, y, b:[blocks]}`; a block is `{t, v:[values or {r: block}], s, s2, s3}`. Saved to localStorage `oss_blocks`. Variables in `oss_blkvars`.
- Running: `bkEval` / `bkVals` / `bkRunList` / `bkRunStack`. Control flow is done with thrown sentinels: `BK_STOP`, `BK_BREAK`, `BK_CONT`, `BK_RESTART`, `BK_FAIL`, `BK_KILL`, and return objects `{bkRet:true, value}`. `bkRunList` rethrows these; loops use `bkLoopBody`.
- Frame hats are `whenframe`, `whenfromto`, `whencond` (see `bkFrameHat`, `bkHit`); `ineditor` runs when a project opens. `bkPlayFrame` fires hats while playing and `bkExportFrame` while exporting (`bkExp` holds export state).
- `bkTry` counts open `try` blocks; `bkFast` is above 0 inside `fast`; `bkKill` is the set of stopped command blocks; `BKCMD` holds running commands.
- Drag and drop: `bkFindSnap` (a cap block can only go at the end of a list or into an empty mouth), `bkDragStart`. The snap height `gh` has to be measured before the ghost is removed.
- Fullscreen editor: anything appended to `document.body` is hidden while fullscreen. Append to `document.fullscreenElement || document.body`.

## Decisions Joel made (keep these)
- Blocks are for creating things that are not assets or characters, and for enabling things. No blocks that make assets or characters (Joel makes those himself). The Rigs, Make assets, Edit text and Fonts blocks were removed for this reason; only the original `x of ()` and `y of ()` stay in Rigs. Converter, Zipper and Scenery don't need blocks (Scenery only gives assets and backgrounds to use in Edit video). Effects doesn't need blocks either: extra effects come as an extension turned on in the Extensions page.
- Blocks are also for working around things that have bugs, and for things that can't be made without blocks. The test for a new block: if Joel can already do it himself in the studio, no block; if he can't, add one.
- No "stop all" block; `stop` ends only its own script. Two `stop` blocks must not stack.
- `<>` truth rule (`bkTruthy`): a number anywhere in the text decides (0 is false); otherwise words are true unless "false"; an empty box is false.
- `infinity of`: repeat M = N+1, L = M^M until L carries E (stop at 1e21); if it goes infinite, the previous step is the answer.
- `() of ()` is the mod block. `() uses ()` has the text first, the part second. The grey keep blocks and the yellow command blocks live in their own categories; the `when <>` hat is the only block in Function. The C-shaped `when <>` is an if block and stays in Control.
- `() out ()` and `() in ()` were dropped on purpose.
- Audio blocks (`talk with audio` = play once, `add sound … as a background` = loop under the whole video, `play sound … about () times` = whole sound back to back N times, `all sounds`) must be in the exported video, because videos go to YouTube. A block never restarts its sound while it is still going (`BK_SND`); while exporting, sounds go into the recording through `bkSndOut`.
- Weather extension (`id: "weather"`, add-on for Effects): adds a Weather group to Add an effect. 🌧 Rain is drawn in code in the style of Joel's hand-drawn rain GIF, not a copy of it (Joel asked for that): his blue #009fff, thin round-ended streaks slanted about 9°, short and long drops, light at the start and then heavier (`rainDraw`). Strength = number of drops; See-through (`e.see`, default 35%) makes it faint. When its last frame passes, no new drops start but the falling ones finish (`fxOn`, `RAIN_TAIL`). More weather effects go in the same group.
- No physics or ragdoll blocks (Joel does that himself): the old `make () able to handle physics` block and the planned `if bounce, play sound` were dropped.
- The Looks blocks `display () at x: () y: ()` and `sticker () at x: () y: ()` were removed (they place assets, which Joel does himself).
- The Zipper no longer loads frames (it was slow); the Upgrades "frames eaten" counter no longer counts for new zips.

## Defaults I guessed that Joel never answered
- `eat` eats from the end of the text; `infinity of` stops at 1e21; the `when <>` hat fires once on false to true; `randify ()` is a random whole number from 1 up to the input; `atan2 ()` has one box and works as arctangent in degrees.
