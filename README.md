# Knee Rehab Circuit Timer

A single-page, offline-capable timer for a knee rehab exercise circuit.
Everything lives in one self-contained `index.html` — no build step, no
dependencies — and is served as a static site from GitHub Pages.

**Live app:** https://jkastl.github.io/knee-strength/

## The circuit

Wall sit → Side-lying hip abduction → Eccentric step-down → Spanish squat →
Bent-knee calf raise. Each exercise finishes all its sets before the next
begins. Per-side exercises run Left then Right with a short switch interval.

## Features

- Big, glanceable countdown; work / rest / get-ready states are colour-coded.
- Timestamp-based timer (no drift), survives backgrounding the tab.
- Web Audio countdown beeps, transition tones, and a per-rep tempo metronome.
- Screen Wake Lock so the phone doesn't sleep mid-set.
- Settings: sound, metronome, get-ready length, rest multiplier — saved to
  `localStorage`. Controls: pause, skip, back, +15s, restart set, end.

The full spec lives in [`design.md`](./design.md).

## Development

Open `index.html` in a browser, or serve the folder
(`python3 -m http.server`). Deployment is just committing to `main`; GitHub
Pages serves the repo root.

<!-- agents:begin -->
## Notes for agents

- **Single source of truth:** the entire app is `index.html` (inline HTML/CSS/JS).
  There is no framework, bundler, or test suite. `design.md` is the spec.
- **Workflow for this repo:** commit straight to `main` and push — no feature
  branches, no PRs, no unit tests. Keep the branch list clean.
- **Hosting:** GitHub Pages from `main` at repo root. `.nojekyll` is present so
  the file is served verbatim. Target context is HTTPS on a phone, not `file://`.
- **Versioning:** `APP_VERSION` in the `<script>` is the source of truth, shown
  on the home screen as `v<version>`. Use `x.y` format only (no patch segment):
  bump the minor for changes, the major for large reworks. Mention the bump in
  the commit message.
- **Timer invariant:** remaining time is always derived from an absolute
  `performance.now()` target (`endTime`), never decremented in a loop. Preserve
  this — it's what keeps the timer accurate across backgrounded tabs.
- **Block model:** a session is a flat `blocks[]` list (getready / work / switch /
  rest / complete) built by `buildBlocks()`. Skip/back move by one block, which
  is what makes crossing side-switch and exercise boundaries behave.
<!-- agents:end -->
