# roomba-signal-mapping

A methodology for mapping and locating a robot vacuum **without pose data, without cameras, and without touching its firmware** — by layering the weak signals a modern home already produces and interpreting them outside the robot.

**Status: early-stage research.** The framework below is being developed and field-tested against a single iRobot i3+ (`daredevil` firmware). Findings, protocols, and code will land here as they mature. Nothing on this page is a finished product yet.

## The problem

Camera-less robot vacuums (iRobot's i-series and similar) report no position data locally. Their SLAM is a sealed box: sensors in, map out, cloud only. If you want to know *where the robot is* — for automation, diagnostics, or curiosity — the vendor gives you nothing.

But the robot is not silent, and neither is your house. The robot emits telemetry and radio signals; the house is full of devices that can hear them. **Individually, none of these signals localize anything. Layered and cross-checked, they can.**

## The architecture

One principle governs everything here: **all intelligence lives outside the robot.**

- The robot is treated as a dumb effector with a small command vocabulary (start, stop, dock, region-targeted cleans).
- A ground-truth map of the space comes from consumer LiDAR (a recent iPhone Pro and a ten-minute walk), not from the robot.
- Every signal the robot produces — and every signal the house can capture about the robot — is logged losslessly and registered onto that map by an external system with real compute.
- Commands are compiled *down* to the robot's vocabulary; telemetry is interpreted *up* against the map.

The robot never knows it's part of a mapping system. This also means the approach survives firmware changes and transfers across robots.

## The signal layers

**L1 — signals the robot already reports** (local MQTT telemetry, i-series):

| layer | signal | what it tells you |
|---|---|---|
| L1.1 | Wi-Fi rssi/snr/noise | continuous coarse position field; a proven motion witness (variance separates driving from parked) |
| L1.2 | bumper event counts | per-region clutter/geometry fingerprint |
| L1.3 | cliff-sensor detections | floor darkness/edge signature |
| L1.5 | battery drain slope | floor-type transitions (carpet loads motors harder) |
| L1.7 | mission timing structure | travel times and durations as distance/size proxies |
| L1.8 | dock events | the one absolute position anchor |

**L2 — robot emissions your house can receive:**

| layer | signal | what it tells you |
|---|---|---|
| L2.1 | your Wi-Fi mesh's AP-side view | which node the robot associates with; roaming handoffs = zone transitions |
| L2.2 | a carried BLE beacon (an AirTag, a spare beacon, or an old phone taped on top) | multi-receiver RSSI from fixed radios |

**L3 — house sensors detecting the robot:**

| layer | signal | what it tells you |
|---|---|---|
| L3.1 | motion/PIR sensors at known positions | timestamped absolute fixes as the robot passes |

(Gaps in the numbering are deliberate: removed layers keep their retired designators.)

## The research roadmap

1. **R1 — Retroactive feature mining.** Mine existing telemetry logs for per-mission features; test whether known missions already separate by region.
2. **R2 — Sensor-field audit.** Inventory what *your* house can hear: mesh AP-side data, BLE receivers, motion-sensor placement.
3. **R3 — Labeled ground truth.** Short repeated region-targeted missions plus brief human observation.
4. **R4 — Combining method.** How the layers merge — scored fusion, decision rules, or single-signal dominance — is an open question the data decides, not a foregone architecture.
5. **R5 — Adjacency map.** Region connectivity from mission transition sequences, dock as origin.
6. **R6 — LiDAR ground truth.** A consumer LiDAR scan becomes the coordinate frame all layers register onto.

## The signal census (start here in your own home)

Every home differs. Before any of this, walk yours and inventory two lists: what emits (the robot's telemetry, its radios, anything you can attach to it) and what listens (mesh nodes, always-on computers with Bluetooth, motion sensors whose positions you know). The methodology adapts to whatever your census finds; the assumed baseline is only a reasonably signal-rich home and a drawer with an old phone in it.

## Who this is for

Anyone geeky enough to be reading this. If you run a multi-node mesh, keep old phones in a drawer, and think a $200 vacuum refusing to say where it is sounds like a challenge rather than a limitation — this is for you.

## Related projects

- [ha_roomba_plus](https://github.com/johnnyh1975/ha_roomba_plus) — a gold-quality Home Assistant integration for Roomba/Braava. Much of the protocol-level field work behind this project was reported and discussed in its issue tracker (see #106, #107, #108). If you want Roomba control *in Home Assistant*, go there; this project is the standalone mapping-research track.
- [dorita980](https://github.com/koalazak/dorita980) — the canonical documentation of the local Roomba protocol.

## Related Prior Art

This project explores ideas related to U.S. Patent 10,922,396, "Signals-Based Authentication," (M. Young et al.), assigned to Bank of America Corporation.
https://patents.google.com/patent/US10922396B2

This repository is an independent personal project and is not affiliated with or endorsed by Bank of America.

This project cites the patent as intellectual lineage only; it does not implement its claims.

## License

Code is licensed under [MIT](LICENSE). Documentation is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Citation

If you build on this work, see [CITATION.cff](CITATION.cff).

Author: Michael Young — [ORCID 0009-0003-7442-2986](https://orcid.org/0009-0003-7442-2986)
