# Roadmap

## V0.1 Read-only SDO Acquisition

- Windows 11 Python package.
- Configurable CAN interface, channel, bitrate, Node-ID, period, duration, and object list.
- Read `0x6064:00 Position Actual Value`.
- Reserve DS-BVM/CiA 402 read objects:
  - `0x6062:00 Position Demand Value`
  - `0x6064:00 Position Actual Value`
  - `0x60F4:00 Following Error Actual Value`
  - `0x606C:00 Velocity Actual Value`
  - `0x6078:00 Current Actual Value`
  - `0x6041:00 Statusword`
- Long-format CSV and local operator UI.
- Software-only unit tests.

## V0.2 Read-only Station Foundation

- Add CAN adapter diagnostics for PCAN, Kvaser, and ZLG.
- Add heartbeat/EMCY passive monitor mode.
- Add clearer SDO abort decoding.
- Add sampling-rate and jitter summary at the end of each CSV run.
- Add field-run manifest beside each CSV.
- Add Statusword, Error Register, and Error Code parsing.
- Add device/axis/session profiles.
- Add dry-run-only command previews and a simulated read-only backend.
- Upgrade the UI into a station prototype without any true execute controls.

## V0.3 Live Review

- UI imports live tail data from CSV or a local event stream.
- Add trend panels for position, following error, velocity, and current.
- Add error timeline and object availability matrix.
- Add operator notes export for `02_实验与现场/01_当前执行/60_实验室推进记录/`.

## V0.4 PDO Read-only Evaluation

- Add TPDO listener mode only after SDO read-only evidence is stable.
- Keep PDO configuration out of scope unless separately released.
- Compare SDO polling and PDO observed rate, jitter, and object completeness.

## V1.0 Industrial Acquisition Application

- Signed Windows installer or portable release package.
- Persistent project profiles.
- Database-backed acquisition sessions.
- Versioned hardware evidence and run manifests.
- Explicit safety-gated transition path before any future motion-control module.
