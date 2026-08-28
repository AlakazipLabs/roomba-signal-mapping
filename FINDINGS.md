# Findings

An append-only log of results from field-testing the signal-layering approach. Newest entries at the bottom.

**Standing caveats, applying to every entry:**

- All work is against a single robot (iRobot i3+, `daredevil` firmware) in a single home. n=1 environment, always.
- Every result is **environment-specific**: radio readings are a property of one home's walls and access-point placement; bump densities are a property of one home's furniture. Nothing here transfers as values — only as method.
- Every claim carries one of three tags:
  - **validated** — reproduced against independent ground truth
  - **preliminary** — measured with a sound method, but thin samples or single sessions
  - **hint** — observed once; direction only, no weight

---

## 2026-08-28 — First retroactive mining of telemetry logs

**Method.** Two weeks of the robot's local telemetry (MQTT — Message Queuing Telemetry Transport — shadow messages) were logged losslessly and mined for per-mission features: duration, Wi-Fi RSSI (Received Signal Strength Indicator) statistics, battery drain rate, and lifetime-counter deltas (cliff detections, bumper events). Mission windows were reconstructed from mission-status messages, using the robot's own cycle-close reports as boundaries.

**Extractor validation, before believing anything.** The extraction was checked against eleven missions that had been directly observed live: reconstructed durations and battery figures matched observation in all eleven. Two failure modes were found and are excluded from claims: one mission's window is unmeasurable (a telemetry gap spans it), and four rapid-sequence test missions carry ambiguous labels (command echoes from adjacent tests landed closer in time than their own).

**Findings.**

1. **Battery drain rate separates mission type** — *preliminary*. Full cleaning missions cluster at ~0.9–1.1 %/min (five missions); mapping-survey (`train`) runs at ~0.33–0.41 %/min (two missions). Consistent with surveying-without-vacuum, and a usable activity signal on its own.
2. **Wi-Fi RSSI ranks rooms but cannot yet distinguish them** — *preliminary*. Per-region mean RSSI orders sensibly (largest central region strongest, farthest zone weakest), but the same region measured on two different days drifted ~3 dB — as large as the between-region gaps at current sample sizes (5–13 samples per short visit). Consequence, imposed by the data: single visits are insufficient for fingerprinting; repeated-visit sampling is required and is the next experiment.
3. **Bump density as a clutter fingerprint** — *hint*. The robot's bumper counter ticked ~35 events/min in a cramped dock-adjacent zone versus ~17–27 events/min on open floor. The direction the clutter hypothesis predicts, observed exactly once.

**Limitations.** Region sample sizes are small throughout; counter deltas are only valid for missions with tight snapshot brackets; all values are one home's values.
