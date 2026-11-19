# v0.2.0-readonly-station-foundation Release Checkpoint

Date: 2026-07-10

This checkpoint freezes `motion_acquisition` as a read-only station foundation for the DS-BVM precision-stage project. It is not a motion-control console and does not release motion functionality.

## Release Position

- Package version: `0.2.0`
- Checkpoint label: `v0.2.0-readonly-station-foundation`
- Scope: read-only CANopen SDO acquisition, status interpretation, CSV + manifest evidence, local station UI prototype, and dry-run-only station architecture preview.
- Out of scope: real drive enable, jog, home, move, fault reset execution, parameter writes, PDO configuration, or NMT state changes.

## Implemented Real Read-Only Functions

- CANopen SDO upload-only client remains the real hardware access path.
- Configurable CAN interface, channel, bitrate, Node-ID, SDO timeout, retry count, sampling period, duration, and object list.
- Read-only object table includes:
  - `0x1000:00 Device Type`
  - `0x1001:00 Error Register`
  - `0x1018:01 Identity Vendor ID`
  - `0x603F:00 Error Code`
  - `0x6062:00 Position Demand Value`
  - `0x6064:00 Position Actual Value`
  - `0x60F4:00 Following Error Actual Value`
  - `0x606C:00 Velocity Actual Value`
  - `0x6078:00 Current Actual Value`
  - `0x6041:00 Statusword`
- Long-format CSV logging with per-object success/failure rows.
- Statusword bit parsing and CiA402 state classification.
- Error Register and Error Code formatting.
- SDO timeout / abort classification.
- Passive heartbeat / boot-up / EMCY frame classification.
- Sampling statistics for cycle elapsed, deadline miss, failed reads, total object reads, and estimated rate.
- Run manifest generation beside CSV output.

## Dry-Run Station Foundation

- `DeviceProfile` and `AxisProfile` define device/axis metadata without accessing hardware.
- `StationSession` and `ExperimentRun` provide session/run structure for future station workflows.
- `ConnectionState` models disconnected / connecting / connected / degraded / faulted states.
- `SafetyState` and permission results describe why future write-like commands are blocked.
- Dry-run command objects exist only as model objects:
  - `Jog`
  - `MoveAbsolute`
  - `FaultReset`
  - `QuickStop`
- `dry_run.py` produces `would_write` / `blocked` previews only.
- `SimulatedReadOnlyBackend` supports UI and tests without CAN access.

## UI Station Prototype

The local UI has been upgraded from a simple acquisition page into a read-only station prototype:

- connection and axis configuration area;
- read-only object selection area;
- read-only command/config preview;
- statusword and CiA402 state display;
- error/alarm area;
- sampling statistics area;
- CSV + manifest import/review area;
- trend placeholder for imported/read-only data;
- `DRY-RUN ONLY` future command preview;
- safety boundary panel.

The UI intentionally does not provide real Servo On, Jog, Move, Home, Enable, Controlword write, Target Position write, or hardware execute controls.

## Safety Boundary

The v0.2.0 checkpoint preserves these real-hardware constraints:

- `canopen_client.py` remains upload-only.
- `CanopenSdoClient` remains the only real CANopen hardware access entry.
- `dry_run.py`, `commands.py`, `safety_state.py`, `profiles.py`, `session.py`, and UI code do not import `canopen_client.py`.
- No `sdo.download` path is implemented.
- No NMT Operational send path is implemented.
- No PDO configuration write path is implemented.
- No real CAN control transmit path is implemented.
- No real write to `0x6040 Controlword` is implemented.
- No real write to `0x607A Target Position` is implemented.
- Dry-run command previews are not executable against hardware.

## Verification Results

Python tests:

```text
python -m pytest
31 passed in 0.47s
```

Playwright UI smoke result:

```text
Page: http://127.0.0.1:8766/index.html
Title: Motion Acquisition Station
Desktop width: no horizontal overflow
Mobile width 390 px: no horizontal overflow
DRY-RUN ONLY text: present
Forbidden true-execution labels checked: Servo On, Home, Enable Axis, Execute Move, Run Jog
Forbidden true-execution labels found: none
```

Static safety checks:

```text
sdo.download: none
.sdo.download: none
OPERATIONAL: none
Controlword write: none
Target Position write: none
```

## Hardware Verification Status

Real DS-BVM / CAN adapter hardware verification has not been performed for this checkpoint.

This release confirms software structure, offline tests, mock/simulated behavior, UI loading, static no-write boundaries, and manifest/CSV compatibility. It does not prove:

- physical CANH/CANL/GND wiring correctness;
- termination correctness;
- actual DS-BVM Node-ID;
- actual DS-BVM bitrate;
- drive firmware/object availability;
- successful SDO upload on the real bench;
- data scaling correctness against a known physical displacement;
- safe motion release.

## Next Real Read-Only Hardware Verification Plan

Before field execution, confirm the current bench release documents in `02_实验与现场/01_当前执行/` remain authoritative.

Recommended first real-hardware checks:

1. Confirm adapter driver and channel with vendor software, for example PCAN-View or the selected CAN adapter utility.
2. Confirm CANH/CANL/GND, termination, and shared reference ground from evidence-backed wiring documents.
3. Confirm Node-ID and bitrate from DS-BVM/Servo Studio evidence or passive heartbeat evidence.
4. Run `motion-acq list-objects` offline to confirm the local software environment.
5. Run one read-only SDO upload for `0x6064:00 Position Actual Value`.
6. If successful, run a short `log-sdo` with `position_actual`, `statusword`, `error_register`, and `error_code`.
7. Save CSV and manifest together.
8. Record command line, adapter, channel, bitrate, Node-ID, observed result, errors, and conclusion boundary in `02_实验与现场/01_当前执行/60_实验室推进记录/`.
9. Do not treat successful read-only acquisition as motion release.

No real write command is required or allowed for this verification plan.
