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
2. **Wi-Fi RSSI ranks rooms but cannot yet distinguish them** — *preliminary*. Per-region mean RSSI orders sensibly (largest central region strongest, farthest zone weakest), but the same region measured on two different days drifted ~3 dB — as large as the between-region gaps at current sample sizes (5–13 samples per short visit). Consequence, imposed by the data: single visits are insufficient for fingerprinting. Repeated-visit sampling was subsequently run (see 2026-08-31) and returned a negative for the room pair tested.
3. **Bump density as a clutter fingerprint** — *hint*. The robot's bumper counter ticked ~35 events/min in a cramped dock-adjacent zone versus ~17–27 events/min on open floor. The direction the clutter hypothesis predicts, observed exactly once.

**Limitations.** Region sample sizes are small throughout; counter deltas are only valid for missions with tight snapshot brackets; all values are one home's values.

---

## 2026-08-31 — Repeat-visit sampling: two rooms do not separate by radio

**In plain terms: we tested whether two rooms could be told apart by radio signal alone. They cannot — and we are publishing that because a negative result is still a result.**

**Method.** The 2026-08-28 entry identified repeated-visit sampling as the necessary next experiment. It was run: alternating short visits to two rooms, thirteen visits each, twenty-six in total, under identical conditions. Room labels were verified against the robot's own command echoes, aligned to mission-start timestamps rather than grouped by time window — an earlier attempt using the naive alignment mislabelled one visit, and was caught and corrected before analysis.

**Findings.**

1. **Two rooms do not separate by mean signal strength** — *validated negative*. Gap between room means 0.99 dB, Mann-Whitney U = 61.5, p ≈ 0.25. An earlier six-visit sample had suggested a 1.85 dB gap at p ≈ 0.09. **Doubling the sample moved the result toward the null**, which identifies the earlier gap as small-sample noise rather than a real effect that was underpowered.
2. **Neither do the other two radio measurements** — *validated negative*. Signal-to-noise ratio p = 1.00, noise floor p = 0.76. All three radio channels the robot exposes are negative for this room pair.
3. **A deep-signal-null tail appears in one room only** — *hint*. Readings at or below −54 dBm occurred exclusively in one of the two rooms across all twenty-six visits. That is a distribution-tail feature rather than a difference of means, and it is untested.

**Consequence for method.** Mean signal strength is not a sufficient room classifier for adjacent rooms in this home, which argues against any fusion approach that averages this signal. Finding 3 suggests that if radio is to separate rooms here, the information lies in the shape of the distribution rather than its centre.

**Limitations.** One home, one robot, two rooms. A negative here does not generalise to other room pairs or other homes — rooms separated by more wall or more distance may well separate cleanly.

---

## 2026-09-01 — Radio signal separates moving from stationary — but does not confirm a dock

**In plain terms: you can tell whether the robot is driving or sitting still from the wobble in its Wi-Fi signal alone, and, correcting an earlier entry, the same measurement shows the opposite of what we first reported: a robot that fails to dock is sitting right on top of its dock's radio signal the whole time, so signal strength alone cannot tell a docked robot from one circling the dock and failing.**

**Method.** Wi-Fi RSSI (Received Signal Strength Indicator) is published by the robot roughly every ten seconds while it is awake. Four windows from a single day were compared, each a distinct activity, 423 samples in total. No additional hardware — this is the robot's own radio telemetry.

**Findings.**

1. **Signal variance separates motion from stillness** — *validated*. Standard deviation of RSSI was **0.85 dB while stationary** (58 samples) against **6.90–9.53 dB while driving** (306 samples across two mission types). An earlier measurement of the same effect gave 1.6 dB against 7.8 dB on roughly a quarter of the sample. This reproduces and tightens it. The stationary window here is a robot stopped **mid-floor**, not docked, which separates "not moving" from "sitting near the access point" — a distinction the earlier measurement could not make.
2. **Signal strength does not confirm a dock — correction of an earlier claim** — *validated (the refutation); the residual is preliminary, n=1*. An earlier version of this entry reported an eleven-minute failed dock attempt as ranging −38 to −59 dBm and never approaching dock-level signal. Re-reading the raw log, that was wrong: the trail ran **−47 to −23 dBm, mean −33.4** (61 samples), and **93% of it was at or above the docked signal level** (a robot parked on this dock overnight reads −44 to −35, mean −35.5). The robot spent almost the entire failed hunt at docked-level radio or stronger, and still ended in `error 18`. **So radio proximity is necessary but not sufficient: signal strength alone cannot separate "docked" from "next to the dock and failing to connect."** The correct reference band came from 35 later docked-contact measurements; the earlier entry compared against the wrong band.
3. **A dock hunt moves less than cleaning does** — *hint*. Signal variance during the hunt (5.14 dB) was lower than during cleaning (6.90 dB) or surveying (9.53 dB), consistent with searching a restricted area rather than covering ground.
4. **The robot's live surface is 51 fields** — *validated*. Of 276 distinct values the robot reports about itself, only 51 changed at any point during a full day of testing. The remainder are identity, capability flags, firmware versions and configuration. Offered as the robot-side equivalent of the signal census this methodology asks readers to perform on their homes.

**Why the correction matters for method.** The earlier claim implied a signal trail could tell "searched the wrong area" from "reached the dock and failed to connect." It cannot — both look identical in radio strength here, because the failing robot is at the dock's radio level throughout. The useful conclusion is the negative one: **do not build a dock-confirmation, or a proximity-triggered dock command, on signal strength alone.** What separates the two failures has to come from elsewhere.

**Limitations.** One robot, one home, one day. Finding 2's two windows differ in more than the task: the robot's dock-localisation state also differed between them, so task and localisation state are not separated here. Decibel values are properties of this home's walls and access-point placement, and transfer only as method.
