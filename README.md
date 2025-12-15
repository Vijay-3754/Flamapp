# Bezier Rope — Interactive Canvas Simulation

Files
- index.html — Single-file HTML + JavaScript app (open in browser).

Overview
This project implements an interactive, real-time cubic Bézier curve simulation using plain HTML Canvas and raw JavaScript. No third-party libraries or built-in Bézier/physics APIs are used. The two interior control points behave like springy rope nodes driven by mouse/touch or device orientation (iOS). Tangents are computed from the analytic derivative and visualized along the sampled curve.

What I implemented
- Manual cubic Bézier evaluation:
  B(t) = (1−t)^3 P0 + 3(1−t)^2 t P1 + 3(1−t) t^2 P2 + t^3 P3
- Analytic derivative (tangent):
  B'(t) = 3(1−t)^2 (P1−P0) + 6(1−t)t (P2−P1) + 3 t^2 (P3−P2)
- Spring-damping physics for control points P1 and P2:
  acceleration = −k * (position − target) − damping * velocity
  Implemented with small fixed sub-steps for stability (explicit Euler integration).
- Input:
  - Mouse / touch position influences targets; can drag P1/P2 directly.
  - iOS DeviceOrientation (gyro) optional: requests permission where required.
- Rendering:
  - The Bézier curve is sampled with small t increments (0.01) and drawn as a polyline (no canvas bezier APIs).
  - Tangents drawn at configurable t intervals, normalized and rendered as short line segments.
  - Control points and control polygon are shown for debugging and interaction.
- Recording:
  - Built-in canvas recorder using MediaRecorder that captures up to 30s and provides a downloadable WebM file. Use the "Record" button in the UI.

Design choices & reasoning
- Manual sampling and derivative: ensures the curve and tangents come from first-principles math; sampling with dt = 0.01 produces a smooth path while keeping CPU usage reasonable.
- Spring model: a simple linear spring + viscous damping provides responsive yet naturally oscillatory motion. Using a small fixed physics timestep (1/240 s) stabilizes explicit integration and keeps behavior consistent across framerates.
- Physics targets: P1/P2 targets are derived from mouse/gyro offsets relative to the curve center and their "base" positions (positions when relaxed). Dragging overrides the automatic target.
- Performance: requestAnimationFrame for render; physics uses fixed small sub-steps. Drawing uses simple canvas primitives and a modest sampling density to maintain >= 60 FPS on typical modern hardware. The UI allows lowering tangent density (t-step) and influence for performance tuning.

How math & physics are organized in code (high level)
- Math section: vector utilities, cubicBezierPoint(t), cubicBezierDerivative(t)
- Physics section: ControlPoint class implementing manual spring-damping acceleration and Euler integration; fixed physics timestep loop.
- Input section: pointer handling, dragging logic, deviceorientation handler, mapping inputs to control point targets.
- Rendering section: clear frame, draw curve using manual sampling, draw tangents using normalized derivative, draw control points/polygon.
- Recording section: MediaRecorder capturing canvas stream, limited to 30s, produces downloadable WebM blob.

How to run
1. Save `index.html` to disk.
2. Open it in a modern desktop/mobile browser (Chrome, Firefox, Edge, Safari).
3. Interact:
   - Move the mouse or touch the canvas to influence the curve.
   - Click/tap and drag the colored middle control points (P1,P2).
   - On iOS Safari: tap "Enable Gyro" and grant motion permission to use device orientation.
4. Use the "Record" button to capture a up-to-30s screen recording of the canvas. After recording finishes you will be offered a downloadable .webm file.

Recording notes
- The built-in recorder captures the canvas only (so no need for external screen-capture tools).
- The recorder will automatically stop at 30 seconds. You can stop early by pressing Stop.
- If you prefer native OS recording tools:
  - macOS: QuickTime Player -> File -> New Screen Recording (or use the built-in screenshot toolbar: Shift-Cmd-5).
  - Windows: Xbox Game Bar (Win+G) or OBS.
  - iOS: Control Center screen recording (may require enabling microphone / internal audio settings).
- After recording, trim if needed and submit the recording along with the files.

Notes & known limitations
- Gyroscope: iOS requires explicit permission prompt; behavior varies by browser/OS.
- The app attempts to maintain responsive 60 FPS but ultimate performance depends on device CPU/GPU. Use the UI sliders to reduce sampling/tangent density on lower-end devices.
