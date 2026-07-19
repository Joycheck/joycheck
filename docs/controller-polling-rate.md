# Measuring controller polling rate in the browser

**Polling rate** is how often a controller reports its state per second, measured
in hertz (Hz). A wired pad may poll at 125, 250, 500, or 1000 Hz; higher and
more *consistent* rates mean lower, steadier input latency. This note covers how
polling rate is measured in the browser and the one big caveat most guides miss.

You can run the measurement on any pad here:
**[Controller polling rate test](https://joycheck.io/polling-rate-test/)**.

## The caveat: the browser sees an *observed* rate, not the USB rate

The Gamepad API is read by polling `navigator.getGamepads()` inside a
`requestAnimationFrame` loop — and `requestAnimationFrame` is capped to the
display's refresh rate (typically ~60 Hz, or ~120/144 Hz on high-refresh
screens). So a browser cannot directly observe a true 1000 Hz USB polling rate;
it observes how often the reported state *changes* within the animation-frame
budget.

That is why a browser tool reports an **observed update rate**, not the raw USB
descriptor rate. It is still useful: it exposes stalls, dropped updates, and the
practical difference between a wired and a Bluetooth connection.

## How the measurement works

Count how many frames produce a *changed* controller state over a fixed window,
then normalize to per-second:

```js
let last = null, changes = 0;
const start = performance.now();

function tick(now) {
  const pad = [...navigator.getGamepads()].find(Boolean);
  if (pad) {
    const sig = pad.axes.map(a => a.toFixed(4)).join(",") + "|" +
                pad.buttons.map(b => b.value.toFixed(3)).join(",");
    if (sig !== last) { changes++; last = sig; }
  }
  if (now - start < 1000) requestAnimationFrame(tick);
  else console.log("observed updates/sec:", changes);
}
requestAnimationFrame(tick);
```

Move a stick slightly during the test so genuine updates are produced; a
perfectly still pad reports no changes and reads as 0.

## Wired vs. Bluetooth

- **Wired (USB):** highest and most consistent observed rate; lowest latency.
- **Bluetooth:** typically lower and more variable, with occasional gaps.

For competitive play, wired is the safer choice when latency consistency matters.

## Try it

Measure your own pad's observed rate and compare wired vs. Bluetooth at
[joycheck.io/polling-rate-test](https://joycheck.io/polling-rate-test/) — free,
in-browser, nothing leaves your machine.

## References

- W3C Gamepad specification — https://www.w3.org/TR/gamepad/
- MDN: Using the Gamepad API — https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API
