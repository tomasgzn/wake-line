# Wake Line

A small collection of self-contained, retro-styled browser games. No build step, no
dependencies — each game is a single HTML file you open directly in a browser.

## Games

### `slalom.html` — Wake Line
A 2D retro water-ski slalom game. You're on the rope behind the boat, chase-cam view:
hold an edge to swing wide, round all six skier buoys on alternating sides, and cross the
wake between each. Clear a pass and the tow rope shortens a notch ("15 OFF" → "41 OFF") —
that's the whole difficulty curve. Miss one buoy and the run is over.

- Course layout: green pre-gates (markers), a red entry gate (must pass through), six red
  boat-path buoy pairs aligned with the skier buoys, a red exit gate.
- Pick a starting rope length on the title screen. Score = buoys rounded; best score is
  saved in `localStorage`.
- Controls: `←/A` `→/D` edge, `P` pause, `M` mute. Touch buttons + tap-to-edge on mobile.
- Physics constants (`CONTROL`, `CENTERING`, `DAMPING`, `THETA_MAX`, buoy/gate geometry) are
  grouped and commented at the top of the `<script>` for tuning.

Published as a Claude Artifact: https://claude.ai/code/artifact/985da22a-21e2-45d3-8465-1f6b06cbabad

### `tetris.html` — Stackfall
A neon block-stacking arcade game (7-bag randomizer, hold, ghost piece).

## Publishing note

`slalom.artifact.html` is a generated file (git-ignored): it's `slalom.html` with the
`<!doctype>/<html>/<head>/<body>` skeleton stripped, which is the form the Claude Artifacts
host expects. Regenerate it from `slalom.html` when publishing an update.
