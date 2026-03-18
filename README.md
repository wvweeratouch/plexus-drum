# Plexus Drum

Motion-to-drum interactive web app. Move your hands or body in front of a camera to trigger percussive sounds.

**Live demo**: https://wvweeratouch.github.io/plexus-drum/
**Mirror** (fullscreen visual): https://wvweeratouch.github.io/plexus-drum/mirror.html

## How It Works

Uses MediaPipe HandLandmarker and PoseLandmarker to track joint positions from webcam video. Velocity of the fastest-moving joint drives drum synthesis via raw Web Audio API (no libraries).

### CEING Machinegun Trigger

The trigger system comes from the CEING experiment:

- Every frame: compute max velocity across all tracked joints
- Smooth: `vel = vel * 0.7 + raw * 0.3`
- Dead zone: skip if velocity < threshold (adjustable, default 0.05)
- Map velocity to `speedNorm` (0-1), compute retrigger interval: `150 - 135 * speedNorm` (floor 15ms)
- On every hit: **kick(v*0.6) + snare(v) always together**. HiHat(v*0.7) added when velocity > 0.03
- Volume = `min(speedNorm * 1.2, 1) * volScale`

This produces the "machinegun" effect -- faster movement = shorter retrigger = dense drum rolls.

## Controls

| Control | What it does |
|---------|-------------|
| **Vol** | Master volume 0-100 (default 80) |
| **Mode** | "Hands" = hand landmarks only, "Body" = pose skeleton (shoulders, elbows, wrists, hips, knees, ankles) |
| **Echo** | Feedback delay: 120ms delay, 0.35 feedback, 0.3 wet mix |
| **PoseT** | Velocity threshold 0.01-0.20 (lower = more sensitive) |
| **Visual** | "Flashes" = hit dots at joint positions, "Waveform" = frequency bars from audio analyser |
| **Persons** | 1-3 people tracked simultaneously |

## Mirror

Open `mirror.html` in a second window/tab (same browser). It receives landmark data via `BroadcastChannel('plexus-drum')` and displays fullscreen hit flashes. Fades to black when no data arrives. Cursor hidden.

## Tech

- MediaPipe tasks-vision 0.10.14 (ESM from jsdelivr CDN)
- Raw Web Audio API synthesis (sine sweeps, filtered noise)
- Single HTML files, no build step
