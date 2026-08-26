# Knee Rehab Circuit Timer — Design

A single-page knee rehab circuit timer, built as one self-contained `index.html`
file and hosted on GitHub Pages.

## Constraints

- **One file.** All HTML, CSS, and JS inline. No build step, no bundler, no npm.
- **No external dependencies.** No CDN links, no web fonts fetched at runtime, no audio files. It works fully offline once loaded.
- **Hosted on GitHub Pages** at the repo root. Develop for that context — served over HTTPS from a static host, not opened from the filesystem.
- **Mobile-first.** Primary use is a phone propped on the floor while I'm on a mat. The timer must be legible from about four feet away, and every control must be hittable one-handed with sweaty hands. Assume a ~390px viewport as the design target; desktop is secondary.
- Vanilla JS. No framework.

## What it does

Runs a guided circuit of knee rehab exercises, timing each set and each rest interval, announcing transitions with audio, and displaying the form cue for the current exercise while it runs.

## Program

A single program, run in the listed order. Each exercise completes all of its
sets before moving to the next — this is a straight progression, not a rotating
circuit.

1. Wall sit
2. Side-lying hip abduction
3. Eccentric step-down
4. Spanish squat
5. Bent-knee calf raise

## Exercise data

Define this as a JS data structure at the top of the script so it's easy to edit. Two timing types:

- `hold` — a straight countdown for `holdSeconds`
- `reps` — duration is computed as `reps × secondsPerRep`, with an optional tempo metronome

Exercises marked `perSide: true` run as two separate timed blocks (Left, then Right) within each set, with a short 10-second switch-sides interval between them.

| Exercise | Type | Sets | Detail | Rest between sets | Per side |
|---|---|---|---|---|---|
| Wall sit | hold | 5 | 45s hold | 30s | no |
| Side-lying hip abduction | reps | 3 | 15 reps @ 5s (1s up, 1s pause, 3s lower) | 45s | yes |
| Eccentric step-down | reps | 3 | 10 reps @ 5s (4s lower, 1s up) | 45s | yes |
| Spanish squat | reps | 3 | 10 reps @ 8s (2s down, 5s hold, 1s up) | 60s | no |
| Bent-knee calf raise | reps | 3 | 15 reps @ 5s (1s up, 1s pause, 3s lower) | 45s | no |

## Form cues (display these during the exercise, not buried in a menu)

Each exercise needs a short **setup** line, a **cue** line, and a **common mistake** line. Use this copy:

**Wall sit**
- Setup: Back flat on the wall, feet shoulder-width, about 18 inches out. Slide down to 45–60°.
- Cue: Shins vertical, knees over ankles, drive down through your heels.
- Mistake: Going to a full 90° squat — that increases pressure behind the kneecap.

**Side-lying hip abduction**
- Setup: On your side, bottom leg bent, top leg straight and stacked. Back against a wall.
- Cue: Lead with the heel, toes rotated slightly down, lift 30–45°, lower over 3 seconds.
- Mistake: Rolling the hip back and swinging the leg forward — that's the hip flexor doing the work.

**Eccentric step-down**
- Setup: Stand on one leg on a 4–6 inch step, other foot hanging off the edge.
- Cue: Lower the free foot over a slow four-count, tap the heel, push back up in one second.
- Mistake: Letting the standing knee cave inward. It tracks over the second toe.

**Spanish squat**
- Setup: Heavy band anchored at knee height, looped behind both knees. Step back into tension.
- Cue: Sit straight down with shins vertical, torso upright, hold 5 seconds at ~60°.
- Mistake: Letting the shins travel forward. The band exists to keep them vertical.

**Bent-knee calf raise**
- Setup: Flat ground, knees bent 20–30°.
- Cue: Hold that knee bend the whole set. Rise, pause one second, lower over three.
- Mistake: Straightening the knee as you rise — that hands the work back to the gastroc.

## Timer behavior

- **Timestamp-based, not interval-accumulated.** Compute remaining time from a target timestamp using `performance.now()`; do not decrement a counter inside `setInterval`. Drift over a 15-minute session is unacceptable.
- Render with `requestAnimationFrame`; keep the displayed number to whole seconds.
- **10-second "Get ready" countdown** before the first block and before each new exercise. Shorter switch-sides interval (10s) between Left and Right.
- **Rest intervals** count down and auto-advance.
- Handle backgrounding: if the tab is hidden and comes back, recompute from the timestamp rather than assuming the session paused.

## Audio

Generate all sound with the Web Audio API — oscillator nodes, no files.

- Short beep at 3, 2, 1 remaining.
- Distinct, lower tone at zero / block complete.
- A different, brighter tone at full session completion.
- **Tempo metronome** for `reps` exercises: an optional click track that marks the phases of each rep (e.g. a click on the concentric, a softer tick on the eccentric beats) so I can actually hit a 4-second lower without counting. Default on, toggleable.
- Initialize the AudioContext on first user gesture (iOS Safari will not play otherwise) and handle the suspended-context resume case.

## Screen wake

Use the Screen Wake Lock API so the phone doesn't sleep mid-set. Re-acquire the lock on `visibilitychange`. Fail silently and continue if unsupported.

## Controls

Large tap targets, minimum 48px. Needed:

- Start / Pause / Resume (space bar also toggles on desktop)
- Skip to next block
- Back to previous block
- +15 seconds to the current block
- Restart current set
- End session

## Screen states

1. **Home** — last session date, "Start" button, a settings affordance, version.
2. **Running** — current exercise name, set X of Y, side indicator when applicable, big countdown, the setup/cue/mistake copy, progress through the session, controls.
3. **Rest** — countdown plus a "Next up: …" preview.
4. **Complete** — session summary.

## Persistence

`localStorage`, wrapped in try/catch. Store: sound on/off, metronome on/off, get-ready duration, rest multiplier, and last session date. Include a "Clear all data" option in settings.

## Settings

- Sound on/off
- Metronome on/off
- Get-ready duration (5 / 10 / 15s)
- Global rest multiplier (0.75× / 1× / 1.25×) for scaling rest as conditioning improves
- Clear data

## Versioning

Display an app version on the home screen, in `x.y` format only (e.g. **1.1**):
bump the minor for changes and the major for large reworks.

## Design direction

Take a real point of view rather than a default dashboard look. Some grounding:

- This is a clinical-adjacent tool used while lying on a floor in pain. It should feel calm, legible, and unhurried — not a gamified fitness app with streaks, confetti, or motivational copy. No exclamation points.
- The countdown number is the hero. It should be enormous and readable at a glance from the floor, and its treatment should be the most memorable thing on the page.
- Rest and work states should be distinguishable by color and layout instantly, without reading — I'll be glancing, not studying.
- Copy in the interface should be plain and instructional. Buttons name what happens.
- Respect `prefers-reduced-motion`. Visible keyboard focus. Sufficient contrast for outdoor/bright-room use.
- Avoid the standard AI-design defaults: cream background with a serif display face and terracotta accent, or near-black with a single acid-green accent. Pick something specific to this brief.

## Quality bar

- Works over HTTPS from GitHub Pages, and offline once loaded.
- No console errors.
- Correct behavior when skipping backward past a side-switch or across an exercise boundary.
- Timer accuracy verified across a backgrounded tab.
