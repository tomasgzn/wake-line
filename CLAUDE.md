# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A collection of self-contained retro browser games. Each game is a single HTML file
with inline CSS and JS — **no build step, no dependencies, no package manager, no tests.**

- `slalom.html` — "Wake Line", a 2D water-ski slalom game (chase-cam, fake 3D projection).
- `tetris.html` — "Stackfall", a neon Tetris (7-bag randomizer, hold, ghost piece).
- `index.html` — landing page linking to both games (the GitHub Pages site root).
- `slalom.artifact.html` — **generated, git-ignored.** See "Publishing" below.

## Running

Open the `.html` file directly in a browser. To serve locally: `python3 -m http.server`
then visit `http://localhost:8000/slalom.html`. There is nothing to build or lint.

## Live site

Published at **https://tomasgzn.github.io/wake-line/** via GitHub Pages (legacy build,
source: `main` branch, `/` root — no Actions workflow, no login for visitors). Every push
to `main` redeploys. The repo is **public** for this reason. `index.html` is the site root.

## Publishing (slalom only)

`slalom.html` is a full standalone document. The Claude Artifact is published from
`slalom.artifact.html`, which is `slalom.html` with the outer skeleton removed, because
the Artifacts host wraps its own `<!doctype>/<html>/<head>/<body>` and reset `<style>`.

To regenerate: copy `slalom.html`, delete the leading `<!doctype html>` through the
opening `<body ...>` tag (including the small reset `<style>` block), and delete the
trailing `</body></html>`. Everything between — starting at the first real markup —
stays verbatim. Then publish via the Artifact tool passing the existing artifact `url`
(tracked in the `artifact-publish-workflow` auto-memory) so it updates in place.

Keep `slalom.html` and `slalom.artifact.html` in sync — every edit to the game goes in
`slalom.html` first, then the artifact copy is regenerated.

## Architecture — both games share one shape

Each game's `<script>` is a single IIFE with this layout, top to bottom:

1. **Tunables** — named constants grouped and commented at the top. In `slalom.html`
   these are the physics/geometry (`CONTROL`, `CENTERING`, `DAMPING`, `THETA_MAX`,
   `BUOY_OFFSET`, `GATE_*`, camera `FOCAL`/`CAM_*`, plus the `STEPS`/`LABELS` rope-length
   progression). Tune the game by editing these, not the loop.
2. **Module-level mutable state** — a `state` string state machine
   (`idle | running | paused | over`) plus loose game vars, all reset in `startGame()`.
3. Audio via WebAudio oscillator blips (`ac()`, `blip()`), gated by a mute flag.
4. `update(dt)` — fixed-timestep physics with an accumulator; `render()` — all Canvas 2D
   drawing, split into `drawX()` helpers. `loop(t)` runs both via `requestAnimationFrame`.
5. Input: keyboard + pointer/touch, dispatched through the `state` machine. Mobile gets
   on-screen buttons and tap-to-edge.
6. `localStorage`, always wrapped in try/catch: `wakeline.best` (best buoy count),
   `wakeline.scores` (leaderboard: array of `{id,name,score,pass,off,date}`, kept sorted
   and capped at 50), `wakeline.name` (last name entered) / the tetris equivalent.

### slalom.html specifics

- **Fake 3D**: `project(x, z)` maps world metres to screen px against `FOCAL`/`HORIZON`/
  camera. The skier's position is polar — `theta` (swing angle off the rope) and `omega`
  (angular velocity) integrated in `update()`; `skierX()`/`skierZ()` derive world coords.
- **Course**: `layCourse()` builds 6 buoys (alternating sides), a red entry gate, 6 red
  boat-path guide pairs, a red exit gate, green pre-gates. `judgeBuoy()`, `judgeGate()`
  (entry) and `judgeExitGate()` check rounding/passage with `FORGIVE`/`GATE_FORGIVE` slack.
  `PREROLL` is the run-up before buoy 1 — long enough to settle before the entry gate.
- **Difficulty**: rounding buoy 6 then skiing through the exit gate (`judgeExitGate()`)
  calls `nextPass()`, which shortens the rope one notch through `STEPS`/`LABELS`
  ("15 OFF" → "41 OFF"). Missing any buoy or either gate → `gameOver()`.
- **Leaderboard**: `gameOver()` calls `setupNameEntry()`; the overlay collects a name and
  `commitScore()` writes to `wakeline.scores`. `renderBoards()` fills the rail's
  `#boardMini` (top 5) and the overlay's `#boardList` (top 6, new entry highlighted).
- `prefers-reduced-motion` is respected via the `RM` flag.

### tetris.html specifics

- `SHAPES` defines piece cells; `rotated()` rotates by coordinate transform (O-piece
  exempt). `refillQueue()` implements the 7-bag. `collides()` is the single placement
  test used by move/rotate/drop/lock. `ghostY()` projects the hard-drop landing row.

## Git — commit and push regularly

This repo is kept under Git + GitHub (`github.com/tomasgzn/wake-line`, private, `origin`,
default branch `main`) specifically so that **no work is ever lost**. As you do work:

- Commit early and often — after each self-contained change, not just at the end of a task.
  Don't let uncommitted work pile up across multiple edits.
- Keep commits **scoped and clean**: one logical change per commit, with a descriptive
  message saying what changed and why.
- **`git push` to `origin` after every commit** so the remote always reflects the latest
  state. Never leave finished work sitting only in the local repo.
- `slalom.artifact.html` is git-ignored (regenerated from `slalom.html`).

See the `use-git-and-github` auto-memory for full context.
