# JoyCheck

**Free, browser-based gamepad tester. Spot stick drift in about 30 seconds. No install, no signup, nothing leaves your browser.**

🎮 **[Open the tester → joycheck.io](https://joycheck.io/)**

## What it does

JoyCheck reads your controller through the [W3C Gamepad API](https://www.w3.org/TR/gamepad/) and shows the live state of every input on each animation frame, with no smoothing or averaging applied on top. Connect any controller the browser recognizes and check:

- **Buttons** — face buttons, bumpers, triggers, D-pad, stick clicks
- **Analog sticks** — live X/Y values and center offset (this is how you spot drift)
- **Triggers** — analog pressure from 0.00 to 1.00
- **Vibration** — fire the rumble motors to confirm haptics
- **Deadzones** — visualize where the stick stops registering

Works with Xbox (Series and One), PlayStation DualSense and DualShock 4, Nintendo Switch Pro, Joy-Con, 8BitDo, GuliKit, and generic PC pads.

## Why it exists

Most controller testers make you install software, create an account, or send your inputs to a server. JoyCheck runs entirely client-side. The controller data never leaves your machine, there is no telemetry, and it is free.

## Tools

- [Gamepad tester](https://joycheck.io/) — the full diagnostic
- [Stick drift test](https://joycheck.io/stick-drift-test/)
- [Controller vibration test](https://joycheck.io/controller-vibration-test/)
- [Deadzone tester](https://joycheck.io/deadzone-tester/)
- [Calibration hub](https://joycheck.io/calibration-tester/)

## About

Built by Taimoor Bamazai, founder of [Elites Algorithm Limited](https://elitesalgorithm.com/).
