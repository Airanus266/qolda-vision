# Software validation

Date: 2026-10-28.

## Completed
- JavaScript syntax checks: `node --check dist/app.js` and `node --check dist/core.js` passed.
- `node --test tests/*.test.mjs`: 12 tests passed, 0 failed. Covers known joint angles, feature invariance to rigid transformations and scale, invalid landmark rejection, synthetic movement range, temporal dwell/hysteresis, startup behavior, tracking-loss reset, neighbor voting, session-held-out evaluation, exclusion of synthetic observations, insufficient-session guards and CSV escaping/provenance.
- A deterministic 20-second simulated movement run at 67-ms steps produced 5 cycles. This checks the complete synthetic-feature-to-counter path; it does not establish camera performance.
- Browser inspection: initial motion view rendered; synthetic mode displayed synthetic provenance, a hand skeleton and changing metrics; session controls changed state; learning view rendered; attempting to record simulated input was rejected; attempting evaluation with no real dataset was rejected.
- Desktop visual inspection performed at the available preview viewport. Responsive CSS is included but was not tested on physical mobile devices.

## Still requires the project owner's computer
- Actual webcam acquisition and MediaPipe inference on a real hand.
- Checking the skeleton alignment, calibration extremes and repetition counts on the target device.
- Collection of a consented real-data dataset and real held-out metrics.
- Testing camera denial and model-download failure on the target browser/network.
- Any assessment of accessibility for people with motor impairments.

No real-user classifier accuracy, therapeutic result or patient study is reported. Cloud-browser animation scheduling appeared throttled, so its displayed FPS is not a target-device performance measurement.

## Release 0.2 validation

19 automated tests passed, including JSON roundtrip, input rejection, independent feature arrays and report metadata. Syntax and UI-reference checks passed locally. New import/export controls have not been exercised in a browser or on a real camera. The GitHub Actions workflow has been supplied but has not run on the user’s GitHub account.

Final archive review: removed an obsolete source-download link; verified all 24 archived files match reviewed source, ZIP integrity and local HTML asset references.
