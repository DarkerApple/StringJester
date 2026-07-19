# MARIONETTE

*A 2D horror rhythm game for five strings.*

You are the puppeteer above a paper theater. Five strings run from your wooden
control cross down to a marionette performing a play on the lit stage below.
The play is a reenactment of something that really happened. The silhouettes
in the front row are keeping score.

**One file. No build step, no dependencies, no external assets.** Open
[`index.html`](index.html) in any modern browser and play.

## The signature mechanic — string travel time

When you trigger a string, a pulse spawns at the crossbar and travels down the
string at a fixed speed. The puppet's limb moves only when the pulse arrives:

```
arrivalTime = inputTime + stringLength / pulseSpeed
```

Judgment (Perfect ±35ms · Good ±70ms · Early/Late ±120ms) is always evaluated
against **arrivalTime**, never inputTime — so you play *ahead* of the music by
exactly the travel latency. A hollow ring contracts onto each crossbar hook:
press the moment it lands, then watch your pulse fly. Deeper stages pay out
longer strings; Act IV's lead time exceeds half a second.

## Controls

| Input | Action |
|---|---|
| `A S D F G` | head · handL · handR · legL · legR (remappable; a spatial ASDFG→left-to-right preset is in settings) |
| `← →` or `J K` | crossbar tilt (sustained tilt rocks the puppet into a walk) |
| tap / drag | tap the actual string hooks; drag the bar to tilt |
| `Esc` | pause |

First run offers a two-phase calibration: audio offset (tap what you hear) and
input offset (tap a silent visual pulse), measured separately and stored in
`localStorage`.

## Failure model — the show never stops

There is no health bar and no fail-out.

- **Slack** — miss a note and that string goes slack: the limb hangs dead with
  a visible catenary sag until you hit the next note on that string cleanly.
- **Tangle** — trigger strings against the chart's cross order (or mash
  adjacent strings) and the pair twists into a literal helix: its input
  bindings swap. Twists stack to three.
- **Untangle** — while tangled, a pale call-and-response appears inside the
  song: watch the shimmer descend, echo it back on the beat. The puppet stands
  idle center-stage while you do, and the audience's unease climbs.

The end-of-song grade (accuracy, max combo, slack-frames, peak twists,
idle-frames) feeds the story state — never a retry gate.

## Note types

`pull` (tap) · `hold` (sustained tension) · `tilt` (match an angle curve) ·
`resist` (she moves on her own — pulling during the window is the miss) ·
`chord` (simultaneous strings).

## Campaign — five acts

1. **The Rehearsal** — nursery flats, short strings, music-box waltz.
2. **The House** — the same choreography, re-scored and staged literally.
3. **The Resistance** — the puppet moves before your pulses arrive.
4. **The Deep Stage** — maximum depth, longest latency, failing footlights.
5. **The Bow** — mid-song she looks up; the camera pulls back; the strings
   continue upward from your own wrists, and the final section inverts the
   mechanic: pulses arrive from above and you *react* instead of anticipate.

Bundled charts carry a small `synth` score rendered offline through the Web
Audio API at load — zero external assets. Progress, grades and the audience's
memory of your failures persist in `localStorage`.

## Custom maps — first-class

Charts are JSON (`marionette.chart/1`) and use the identical loader as the
campaign. Import by dropping a `.chart.json` on the page, by URL, or via the
shareable deep link:

```
index.html#/chart=<url-of-json>        (also accepts data:application/json;base64,…)
```

`meta.audio` may be a URL (resolved relative to the chart URL) or you can
attach a local audio file; bundled-style charts may embed a `synth` score
instead. Three example charts of increasing scope ship in Free Play,
exercising every note type.

### Editor (`index.html?edit=1`)

Waveform scrub, snap-to-grid 1/1–1/16, five string lanes plus tilt and event
lanes, place/drag/delete, hold/tilt duration by edge-drag, live playtest from
the cursor (`Space`), palette and stage editing, JSON export/copy — round-trip
lossless. Nothing autosaves (localStorage is reserved for settings,
calibration and campaign progress): export your work.

The full chart format is documented in-game under **Help**, including stage
flats (paper-cut polygon layers with parallax), footlights, `puppetDepth`, and
events (`flatSwap`, `depth`, `audience`, `scare`, `line`, `footlights`,
`invert`, `pullback`, `bow`, `blackout`).

## Accessibility

Remappable keys · adjustable judgment window · reduce-flashing toggle ·
disable-jumpscares toggle (scares become a neutral cue) · visual metronome.

## Technical notes

- Single `AudioContext` clock; song position is `ctx.currentTime - startTime`
  against a decoded `AudioBuffer` — never rAF deltas or `audio.currentTime`.
- Inputs are timestamped from `event.timeStamp` and mapped onto the audio
  clock, not the processing frame.
- Fixed-timestep simulation at 120 Hz, decoupled from render; render is a
  pure function of the state object.
- Static flats pre-render to offscreen canvases; footlights are the dominant
  (and only diegetic) light source, so all shadows project upward onto the
  backdrop.
- Dev hooks: `MARIONETTE.dev.selftest()` (logic checks) and
  `MARIONETTE.dev.autotest('ex1')` (plays a chart through the real input
  pipeline and returns the results screen data).
