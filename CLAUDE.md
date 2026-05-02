# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

```bash
python3 -m http.server 8080
# Open http://localhost:8080 in a browser
```

Must be served via HTTP — opening via `file://` blocks MediaPipe model loading due to CORS.

## Architecture

Single-file web app (`index.html`). No build step, no package manager, no dependencies to install. All external dependencies (MediaPipe FaceLandmarker) are loaded from CDN at runtime.

**Rendering pipeline per frame (`renderLoop`):**
1. Mirror-flip the webcam frame onto the canvas (`ctx.scale(-1, 1)`)
2. Run `faceLandmarker.detectForVideo()` → get 468 landmarks per face
3. Mirror-correct landmark x-coords (`x = 1 - x`)
4. Call `drawFilter(landmarks)` which applies all visual effects in order:
   - Squish the face vertically (canvas pixel manipulation via `ctx.drawImage` self-copy)
   - Enlarge and spread eyes (`LEFT_EYE` / `RIGHT_EYE` landmark groups)
   - Draw chihuahua ears anchored to landmark `#10` (top of head)
   - Draw black dog nose over nose tip (landmark `#4`)
   - Draw teeth when mouth openness exceeds threshold (`MOUTH_OPEN_TOP #13` vs `MOUTH_OPEN_BOTTOM #14`)

## Key Helpers

| Function | Role |
|---|---|
| `lm(landmarks, idx)` | Converts normalized landmark to canvas pixel coords |
| `bbox(landmarks, indices)` | Bounding box + center for a group of landmark indices |
| `drawEar(tipX, tipY, w, h, lean, flip)` | Draws one chihuahua ear with fur strokes and pink inner |
| `drawTeeth(lipBox, mouthTop, mouthBot, openness)` | Draws teeth only when mouth is open |

## Landmark Index Groups

All landmark index sets are defined in the `IDX` object at the top of the script. Coordinates from MediaPipe are normalized 0–1; `lm()` and `bbox()` convert them to canvas pixels. Because the canvas is mirror-flipped, `LEFT_EYE` corresponds to the viewer's right eye.

## Filter Tuning Constants

- `SQUISH = 0.78` — vertical compression ratio for the face squish effect
- `SPREAD = 1.22` — horizontal spread multiplier for eye separation
- GPU delegate is tried first; falls back to CPU automatically on failure
