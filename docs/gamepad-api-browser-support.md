# Gamepad API browser support

The [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API)
is widely available, but the *details* differ per browser — especially
vibration. This is what to expect when reading a controller in the browser.

## Feature matrix

| Feature | Chrome / Edge | Firefox | Safari |
|---|---|---|---|
| `gamepadconnected` / `gamepaddisconnected` events | Yes | Yes | Yes |
| Buttons (`.buttons[].pressed`, `.value`) | Yes | Yes | Yes |
| Analog axes (`.axes[]`) | Yes | Yes | Yes |
| Standard mapping (`.mapping === "standard"`) | Yes | Yes | Partial |
| Vibration (`vibrationActuator`, dual-rumble) | Yes | Partial | No |
| Trigger-rumble / haptics | Limited | No | No |

This reflects behaviour observed while building JoyCheck; treat it as a
starting point and test on your target browsers. Support changes over time.

## Things that bite people

**1. You must poll.** Axis movement does not fire events. Read
`navigator.getGamepads()` inside a `requestAnimationFrame` loop and re-fetch
every frame — cached `Gamepad` objects go stale.

**2. A gamepad only appears after activity.** For privacy/anti-fingerprinting,
some browsers won't expose a connected pad until the user **presses a button**.
If nothing shows up, prompt the user to press any button first.

**3. Standard vs. non-standard mapping.** When `gamepad.mapping === "standard"`,
button/axis indices follow the
[standard layout](https://w3c.github.io/gamepad/#remapping) (index 0 = A/cross,
axes 0–1 = left stick, 2–3 = right stick). When it is `""`, indices are
device-defined and you must map them yourself. Joy-Cons and some third-party
pads frequently report a non-standard mapping.

**4. Vibration is not guaranteed.** Check for the actuator before calling it:

```js
const act = pad.vibrationActuator;
if (act && typeof act.playEffect === "function") {
  act.playEffect("dual-rumble", {
    duration: 300, strongMagnitude: 1.0, weakMagnitude: 0.5
  });
}
```

Safari currently exposes no vibration; Firefox support is partial.

**5. Secure context.** The API is only exposed over HTTPS (or `localhost`).

## References

- W3C Gamepad specification — https://www.w3.org/TR/gamepad/
- MDN Gamepad API — https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API
- Test any controller against this in your browser: https://joycheck.io
