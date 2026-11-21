# Flamey's Official Animation Studio: Blocks extension

One big HTML file (`Flamey_s_Official_Animation_Studio_extensions.html`, ~1.6 MB). All scripts are inline `<script>` tags. Joel (the owner) works in HTML5 canvas and browser formats and likes direct, short feedback.

## Checking your work
- Syntax: pull the inline scripts out into one file and run `node --check` on it.
- Behaviour: Playwright (chromium) against the file. Call `bkEval({t:'blockname', v:[...]})` directly. To draw a block: `bkBlockEl({t, v}, {palette:true})` into a div, then screenshot.
- To open the real editor in a test: push `"blocks"` onto `EXI`, add a `<div class="v v-blocks">` next to `.v-edit`, call `go('blocks')`, then `document.querySelector('.bkpanel').__setFull(true)`. Stub `upgLocked` to return false.
- Loops only run when `playing` is true and `EXI` includes `"blocks"`.
- The network is blocked in the test sandbox. Mock `window.fetch` for the definition block (`lookdef`).

## How blocks work
- `BLK` defines each block: `{cat, shape, tpl, a, run, [col], [lazy], [loop]}`. `tpl` uses `%1 %2` for inputs. Shapes: `hat`, `cmd`, `cap`, `c` (one mouth), `c2` (two mouths, `because`), `c3` (three mouths, `trycatch`), `rep` (round), `bool` (pointed), `list` (square).
- `a` entries are `[type, default, options]` with types `num`, `str`, `text`, `bool`, `menu`.
- The tray order is the key order of `BLK` within each category. `BKCAT` holds the categories: watch, settings, operators, control, keep, commands, function, rigs, looks, edit, variables.
- `BKS` is the list of scripts `{x, y, b:[blocks]}`; a block is `{t, v:[values or {r: block}], s, s2, s3}`. Saved to localStorage `oss_blocks`. Variables in `oss_blkvars`.
- Running: `bkEval` / `bkVals` / `bkRunList` / `bkRunStack`. Control flow is done with thrown sentinels: `BK_STOP`, `BK_BREAK`, `BK_CONT`, `BK_RESTART`, `BK_FAIL`, `BK_KILL`, and return objects `{bkRet:true, value}`. `bkRunList` rethrows these; loops use `bkLoopBody`.
- Frame hats are `whenframe`, `whenfromto`, `whencond` (see `bkFrameHat`, `bkHit`); `ineditor` runs when a project opens. `bkPlayFrame` fires hats while playing and `bkExportFrame` while exporting (`bkExp` holds export state).
- `bkTry` counts open `try` blocks; `bkFast` is above 0 inside `fast`; `bkKill` is the set of stopped command blocks; `BKCMD` holds running commands.
- Drag and drop: `bkFindSnap` (a cap block can only go at the end of a list or into an empty mouth), `bkDragStart`. The snap height `gh` has to be measured before the ghost is removed.
- Fullscreen editor: anything appended to `document.body` is hidden while fullscreen. Append to `document.fullscreenElement || document.body`.

## Decisions Joel made (keep these)
- No "stop all" block; `stop` ends only its own script. Two `stop` blocks must not stack.
- `<>` truth rule (`bkTruthy`): a number anywhere in the text decides (0 is false); otherwise words are true unless "false"; an empty box is false.
- `infinity of`: repeat M = N+1, L = M^M until L carries E (stop at 1e21); if it goes infinite, the previous step is the answer.
- `() of ()` is the mod block. `() uses ()` has the text first, the part second. The grey keep blocks and the yellow command blocks live in their own categories; the `when <>` hat is the only block in Function. The C-shaped `when <>` is an if block and stays in Control.
- `() out ()` and `() in ()` were dropped on purpose.
- The Zipper no longer loads frames (it was slow); the Upgrades "frames eaten" counter no longer counts for new zips.

## Defaults I guessed that Joel never answered
- `eat` eats from the end of the text; `infinity of` stops at 1e21; the `when <>` hat fires once on false to true; `randify ()` is a random whole number from 1 up to the input; `atan2 ()` has one box and works as arctangent in degrees.
