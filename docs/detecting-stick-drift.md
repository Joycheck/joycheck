# Detecting analog stick drift in the browser

**Stick drift** is when a controller reports stick movement while you are not
touching it. In the Gamepad API it shows up as axis values that are not ~0 at
rest. This note explains how to read it numerically and — the part most guides
skip — how to tell a *deadzone problem* apart from *real hardware failure*.

## The core idea

At rest, a healthy stick reports both axes very close to `0.0`. Drift is any
persistent non-zero reading with your hands off the controller.

```js
function readRest(pad) {
  const [lx, ly, rx, ry] = pad.axes;             // standard mapping
  return {
    left:  Math.hypot(lx, ly),                   // 0 = centered
    right: Math.hypot(rx, ry),
  };
}
```

`Math.hypot(x, y)` gives the stick's distance from center (its magnitude),
which is more meaningful than either axis alone.

## Measure, don't guess — sample over time

A single frame can catch electrical noise. Average the resting magnitude over a
short window (e.g. ~1 second) so you measure sustained drift, not a blip:

```js
async function measureDrift(getPad, ms = 1000) {
  const samples = [];
  const start = performance.now();
  while (performance.now() - start < ms) {
    const pad = getPad();
    if (pad) samples.push(readRest(pad).left);
    await new Promise(requestAnimationFrame);
  }
  const avg = samples.reduce((a, b) => a + b, 0) / samples.length;
  const max = Math.max(...samples);
  return { avg, max };
}
```

## Interpreting the number

Values are magnitude from center on a 0–1 scale. Rough, practical bands:

| Resting magnitude | Reading |
|---|---|
| `< 0.05` | Healthy — normal noise, a small deadzone hides it. |
| `0.05 – 0.10` | Minor drift — usually correctable with a deadzone. |
| `0.10 – 0.20` | Noticeable drift — will affect aim/movement in games. |
| `> 0.20` | Severe — likely genuine hardware wear (potentiometer). |

These are guideline bands from testing, not a spec. Calibrate against your own
sample of controllers.

## Deadzone vs. real failure

This is the distinction that matters:

- **Deadzone problem:** the stick reads a small non-zero value at rest but is
  otherwise smooth and accurate through its range. A software/deadzone tweak
  fixes it. Most "my stick drifts a little" cases are this.
- **Hardware failure:** the resting value is large, or it *jumps around*
  (high `max` relative to `avg`), or the drift grows over minutes of use. A
  deadzone only masks it; the potentiometer is wearing out.

A quick heuristic: if `max - avg` is large, the signal is jittery (favor
hardware); if `avg` is high but stable, a deadzone may still rescue it.

## Applying a deadzone (radial)

Radial deadzones preserve direction better than per-axis ones:

```js
function applyDeadzone(x, y, dz = 0.08) {
  const mag = Math.hypot(x, y);
  if (mag < dz) return [0, 0];
  const scaled = (mag - dz) / (1 - dz);          // rescale so travel stays 0..1
  return [(x / mag) * scaled, (y / mag) * scaled];
}
```

## Try it

You can run this whole measurement on any controller, in your browser, at
[JoyCheck](https://joycheck.io/stick-drift-test/) — leave the sticks untouched
and it reports the resting magnitude and a drift severity read in about 30
seconds. Nothing leaves your browser.

## References

- W3C Gamepad specification — https://www.w3.org/TR/gamepad/
- MDN: using the Gamepad API — https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API
