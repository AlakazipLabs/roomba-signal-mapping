# roomba-signal-mapping

A methodology for mapping and locating a robot vacuum **without pose data (position plus heading), without cameras, and without touching its firmware** — by layering the signals a modern home already produces and interpreting them outside the robot.

**Status: early-stage research.** The framework below is being developed and field-tested against a single iRobot i3+ (`daredevil` firmware). Findings, protocols, and code will land here as they mature. Nothing on this page is a finished product yet.

## The problem

Camera-less robot vacuums (iRobot's i-series and similar) report no position data locally. Their SLAM (Simultaneous Localization and Mapping — the algorithm that builds a map while tracking position on it) is a sealed box: sensors in, map out, cloud only. If you want to know *where the robot is* — for automation, diagnostics, or curiosity — the vendor gives you nothing.

But the robot is not silent, and neither is your house. The robot emits telemetry and radio signals; the house is full of devices that can hear them. **Individually, none of these signals localize anything. Layered and cross-checked, they may.**

## The architecture

One principle governs everything here: **all intelligence lives outside the robot.**

- The robot is treated as a dumb robot with a small command vocabulary (start, stop, dock, region-targeted cleans).
- Every signal the robot produces — and every signal the house can capture about the robot — is logged and registered onto a map by an external system with real compute.
- Commands are compiled *down* to the robot's vocabulary; telemetry is interpreted *up* against the map.

The robot never knows it's part of a mapping system. This also means the approach survives firmware changes and transfers across robots.


## The signal census (start here in your own home)

Every home differs. Before any of this, walk yours and inventory two lists: what emits (the robot's telemetry, its radios, anything you can attach to it) and what listens (mesh nodes, always-on computers with Bluetooth, motion sensors whose positions you know). The methodology adapts to whatever your census finds; the assumed baseline is only a reasonably signal-rich home and a drawer with an old phone in it.

## Who this is for

Anyone geeky enough to be reading this. If you run a multi-node mesh, keep old phones in a drawer, and think a $200 vacuum refusing to say where it is sounds like a challenge rather than a limitation — this is for you.

## Related projects

- [ha_roomba_plus](https://github.com/johnnyh1975/ha_roomba_plus) — a gold-quality Home Assistant integration for Roomba/Braava. Much of the protocol-level field work behind this project was reported and discussed in its issue tracker (see #106, #107, #108). If you want Roomba control *in Home Assistant*, go there; this project is the standalone mapping-research track.
- [dorita980](https://github.com/koalazak/dorita980) — the canonical documentation of the local Roomba protocol.

## Related Prior Art

This project explores ideas related to two U.S. patents assigned to Bank of America Corporation:

- U.S. Patent 10,922,396, "Signals-Based Authentication" (M. Young).
  https://patents.google.com/patent/US10922396B2
- U.S. Patent 11,645,427, "Detecting unauthorized activity related to a device by monitoring signals transmitted by the device" (M. R. Young et al.).
  https://patents.google.com/patent/US11645427B2

The shared idea in both: signals a device already emits are interpreted by a system outside the device, against a baseline it holds. That is the architectural stance of this project — all intelligence lives outside the robot.

This repository is an independent personal project and is not affiliated with or endorsed by Bank of America.

This project cites the patents as intellectual lineage only; it does not implement their claims.

## License

Code is licensed under [MIT](LICENSE). Documentation is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Citation

If you build on this work, see [CITATION.cff](CITATION.cff).

Author: Michael Young — [ORCID 0009-0003-7442-2986](https://orcid.org/0009-0003-7442-2986)
