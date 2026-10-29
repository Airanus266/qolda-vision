# QOLDA Vision — Hand Motion Lab

**A camera-based research prototype for hand movement interaction, personal gesture learning and honest session-level evaluation.**

An extension of the QOLDA rehabilitation glove concept, designed to work without wearable hardware. This is a software prototype: **no clinical efficacy, patient study or diagnostic capability is claimed.**

## Release 0.2

[Quick start](#run-locally) · [Architecture](docs/ARCHITECTURE.md) · [Model card](docs/MODEL_CARD.md) · [Dataset contract](docs/DATASET_CARD.md) · [Experiment protocol](docs/EXPERIMENT.md) · [Validation](docs/VALIDATION.md)

The release includes dataset JSON save/restore, strict input validation and downloadable experiment reports. Preserve a dataset, reload the app and restore it without losing capture-session boundaries. The same JSON can be evaluated without the browser:

```sh
node scripts/evaluate-json.mjs qolda-dataset.json > report.json
```

`npm run check` checks syntax and UI element references. `npm test` runs the regression suite. A GitHub Actions workflow runs both on Node.js 20 and 22 after upload; no successful remote CI run is claimed in advance.

## What works

- Browser-local webcam hand tracking using Google's pretrained MediaPipe Hand Landmarker.
- Mirrored video with a 21-point skeleton; geometric finger extension and normalized thumb–index distance.
- A temporal open-to-close cycle counter with hysteresis and a minimum dwell time.
- Optional personal open/closed reference calibration.
- A visibly labeled synthetic simulation for trying the interface without a camera.
- Labeled camera-data capture, a personal kNN gesture classifier and leave-one-session-out evaluation against a nearest-centroid baseline.
- CSV exports with source provenance and capture-session IDs; reproducible command-line evaluation.

## Run locally

Requirements: Python 3 for serving; a recent browser. Node.js 20+ is needed only for tests and offline evaluation. No npm install is required.

```sh
python3 -m http.server 8000 --directory dist
```

Open **http://localhost:8000**. Use localhost or HTTPS, not a file:// URL. Grant camera permission. On first camera use, the browser downloads the MediaPipe JavaScript, WASM runtime and model from jsDelivr / Google; internet access is required. Google Fonts is optional: system fonts are a fallback. No API key is required.

## A five-minute demo

1. Open Motion studio and click **Run simulation**. The synthetic badge must be visible. Start a session to explore the temporal counter; export contains `source=synthetic`.
2. Enable your camera. Keep a single hand fully visible, palm generally facing the camera. Starting a different input clears the current movement session and calibration.
3. Set a comfortable closed and open reference if default thresholds do not fit your hand position. Each reference uses the median of about one second of observations.
4. Start a movement session. Open, hold briefly, close, hold briefly. Export numerical observations as CSV. A pause suspends counting; tracking loss discards a pending cycle.
5. Open Gesture learning. Record all four gestures in session 1, start **New session**, change position or lighting, and record the classes again. At least two classes across two independent sessions are necessary for scoring. More independent sessions and participants are preferable.
6. Evaluate, inspect class confusions, and export the dataset. Live camera inference switches to the personal classifier once two labeled classes exist.

## Method

### Attribution and contribution

**Pretrained third-party component:** Google's MediaPipe Hand Landmarker (`@mediapipe/tasks-vision` 0.10.18), loaded only when the user starts the camera. The landmark neural network was not trained by this project. See [Google documentation](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker/web_js).

**Application implementation:** landmark-to-feature geometry, personal range references, temporal counting, collection and provenance workflow, local kNN classification, session-level evaluation, comparison baseline, visualization and exports. AI coding assistance was used to implement this prototype; the project owner should review, test and understand its behavior before describing it as their work.

### Six features

For each of five fingers, extension is the mean of two adjacent internal joint angles divided by 180 degrees (equivalently, their sum divided by 360). Angles near 180 degrees indicate a straighter finger. Thumb geometry is a crude descriptor, not a biomechanical thumb model.

The sixth feature is thumb-tip to index-tip distance divided by wrist-to-middle-MCP length, clipped to [0,3] then divided by 3. The CSV column `pinch_normalized` stores this clipped [0,1] classifier feature; the UI separately shows the **unclipped ratio** in palm-length units.

World landmarks are used for feature extraction when available, screen landmarks otherwise. The four non-thumb extension features are averaged as raw openness. Without personal calibration, `clip((raw - 0.45) / 0.5, 0, 1)` maps this to the displayed scale. With valid calibration, the comfortable closed/open median references replace those defaults. These are geometric proxies, not measured clinical range of motion.

### Repetition counting

An open state (normalized openness >= 0.75) must persist for 180 ms before a closed state (<= 0.25) can complete a cycle after its own 180 ms dwell. Values in the middle interrupt a candidate transition. Persistent closed frames cannot count repeatedly. Tracking loss longer than 400 ms resets the pending state. A source change resets the movement session.

### Classifier and evaluation

- kNN with k=5 (or fewer when fewer observations exist), unweighted Euclidean distance in the six normalized features, deterministic label tie-breaking.
- Vote agreement is a neighbor fraction, **not a calibrated probability**. The model always selects a known class and has no reliable unknown-gesture detector.
- Leave one capture session out in each fold; all predictions for that session use only the other sessions.
- Comparator: a nearest class centroid fitted only on the same training fold.
- Outputs: accuracy, macro F1 and a confusion matrix (actual rows, predicted columns).
- Synthetic observations are excluded from learning and evaluation. No real-world accuracy is reported until the user collects data.

```sh
node --test tests/*.test.mjs
node scripts/benchmark.mjs path/to/qolda-gesture-dataset.csv
```

The benchmark prints JSON; retain this next to your experimental notes. Software unit tests use synthetic fixtures for correctness only and are **not** evidence of real-user model performance.

## Collect an actual experiment

Use `docs/EXPERIMENT.md` to record devices, people, sessions, conditions and observations. Hold out entire participants to measure generalization across people: assign a consistent session/group identifier to each participant before running the command-line evaluation. Do not give one participant multiple unrelated IDs when claiming participant-level evaluation.

Capture-session IDs in the app automatically increment within the tab. They are not participant identities. Browser session-level evaluation alone does not demonstrate generalization across people. Short recordings contain strongly correlated observations; effective sample size is much smaller than frame count. Increasing independent sessions matters more than collecting thousands of near-identical frames.

## Privacy and limitations

- Frames are processed in the browser and are not intentionally uploaded by application code. External dependencies and fonts require network requests.
- Numerical examples and session observations live only in memory. Reloading the tab clears them. Save dataset JSON to preserve and restore labeled examples; CSV is also available.
- Exported CSV files have no images or names, but human movement data should still be collected with permission and handled thoughtfully.
- Occlusion, side views, unusual postures, motion blur and lighting changes can cause errors. Pretrained tracking coverage for people with hand impairments has not been established here.
- Camera inference is synchronous and throttled to about 15 Hz on the main thread. Low-end devices can still experience UI stalls; moving inference to a worker is a follow-up.
- Session export is capped at 36,000 rows; labeled examples at 3,000 to bound browser memory and evaluation cost.
- Calibration is local to the selected input and does not establish a therapeutic assessment.
- The simulator is a kinematic illustration, not a biomechanical model.
- No clinical deployment, prescribed exercises, health recommendations or treatment claims.

## Repository layout

| Path | Purpose |
|---|---|
| `dist/` | Buildless web app; upload these contents for static hosting |
| `dist/core.js` | Pure geometry, state machine, classifier and evaluation |
| `dist/app.js` | Camera integration, collection and UI |
| `tests/core.test.mjs` | Deterministic correctness checks |
| `scripts/benchmark.mjs` | Reproduce evaluation from an exported CSV |
| `docs/EXPERIMENT.md` | Blank real-data research protocol |
| `docs/VALIDATION.md` | Actual software verification and remaining limits |

## Next work

Collect a small consented multi-person dataset; compare session and participant splits; quantify effects of lighting and occlusion; add unknown-gesture rejection; move perception to a web worker; compare camera descriptors with glove sensor signals when hardware is available.

**Author / project owner:** Aibol Namiyash. QOLDA Vision is a continuation of the QOLDA project.
