# Plexus Drum

Motion-to-drum interactive web app. Use hand gestures to select drums and movement velocity to control volume.

**Live demo**: https://wvweeratouch.github.io/plexus-drum/
**Mirror** (fullscreen visual): https://wvweeratouch.github.io/plexus-drum/mirror.html

## How It Works

Uses MediaPipe HandLandmarker to track hand landmarks from webcam. A dense **cyan plexus wireframe** connects all detected landmarks — every pair of points within 400px draws a connecting line with distance-based opacity.

### Gesture → Drum Mapping (Hands Mode)

Finger spread (average pairwise distance between all 5 fingertips) selects the drum:

| Spread | Gesture | Drum |
|--------|---------|------|
| < 0.15 | Fist | Kick |
| 0.15 – 0.40 | Mid | Snare |
| > 0.40 | Open hand | Hi-hat |

**Velocity** (average movement of ALL 21 landmarks from previous frame, normalized `× 15`, capped at 1) controls **volume**.

### Body Mode

Switches to PoseLandmarker tracking joints [0,11-16,23-28]. No gesture mapping — uses velocity zones instead:
- Movement detected: Kick + Snare together
- High velocity: adds Hi-hat

### Echo Delay (always on by default)

- Delay: 180ms
- Feedback: 35%
- Lowpass on feedback: 3kHz
- Wet send: 30%

### Retrigger

Default: 50ms (adjustable 10–300ms via slider). Controls minimum time between drum hits.

## Visual

- **Plexus**: Dense cyan wireframe connecting all landmarks. Line opacity = `(1 - dist/400) × 0.5 × confidence`. Nodes at 3px with 7px glow halos. Color: `#00e5ff`.
- **Waveform**: AnalyserNode frequency bars in cyan.
- When no hands detected: "Show your hands" centered text.

## Controls

| Control | What it does |
|---------|-------------|
| **Vol** | Master volume 0–100 (default 80) |
| **Mode** | "Hands" = hand landmarks + gesture mapping, "Body" = pose skeleton + velocity zones |
| **Echo** | Feedback delay (ON by default): 180ms, 0.35 feedback, 3kHz lowpass, 0.3 wet |
| **Retrig** | Retrigger interval 10–300ms (default 50ms) |
| **Visual** | "Plexus" = cyan wireframe, "Waveform" = frequency bars |
| **Persons** | 1–3 people tracked simultaneously |

## Dashboard

Right-side overlay showing:
- Tracking status (green dot)
- Number of hands/poses detected
- Velocity percentage bar
- Finger spread value
- Current gesture → drum mapping

## Layout

- Main visual: full screen
- Camera preview: small 160×90 overlay, bottom-left
- Controls: top bar
- Dashboard: right side overlay

## Mirror

Open `mirror.html` in a second window/tab (same browser). Receives landmark data via `BroadcastChannel('plexus-drum')` and renders the same cyan plexus wireframe. Fades to black after 2s of silence.

## Tech

- MediaPipe tasks-vision 0.10.14 (ESM from jsdelivr CDN)
- Raw Web Audio API synthesis (sine sweeps, filtered noise)
- Single HTML files, no build step
