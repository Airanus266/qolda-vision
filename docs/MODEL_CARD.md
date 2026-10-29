# Model card

Status: research prototype; no real-user evaluation supplied.

## System components

1. Google MediaPipe Hand Landmarker, accessed using the pinned Tasks Vision 0.10.18 web package and the hand_landmarker float16/1 model asset. This is third-party pretrained perception.
2. Six deterministic geometric features derived from landmarks.
3. A personal k-nearest-neighbor gesture classifier, fitted by retaining camera examples collected by the user. It starts with no examples.

## Intended use

Educational experiments in hand interaction and small personal gesture datasets. The application supports four labels: Open palm, Fist, Pinch, Point.

## Evaluation

Leave-one-capture-session-out accuracy, macro F1 and confusion matrix, with a nearest-centroid comparator on identical training folds. Metrics are calculated from actual collected examples only. A generated JSON report preserves the protocol and sample/group counts.

There is no reference population dataset, published benchmark result, user study or demonstrated therapeutic effect included in this release. Unit tests exercise code correctness using synthetic fixtures.

## Limitations

Gesture labels, sensor geometry and thresholds are chosen manually. kNN vote agreement is not a confidence calibration. Unknown gestures are assigned to a known class. Finger occlusion and uncommon postures can degrade landmark quality. Healthy-user results would not establish applicability to people with motor impairments. Dependence between sessions from the same person can inflate apparent generalization.

## Appropriate reporting

State the number of people, recording sessions, device, class balance and split unit. Describe the model as a personal classifier on top of pretrained landmarks. Distinguish individual contributions, third-party components and AI-assisted implementation.
