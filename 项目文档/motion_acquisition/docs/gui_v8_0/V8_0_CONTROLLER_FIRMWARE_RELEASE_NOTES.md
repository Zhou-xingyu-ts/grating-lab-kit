# Motion Controller Firmware V8.0.0 release notes

## Version boundary

V8.0.0 is the version of the independently released STM32H743 controller
firmware baseline. The Windows Motion Acquisition desktop remains at the
independently accepted V7.0.0 baseline. Protocol 4.0 and the authoritative raw-
count, 16-column `samples.csv` contract are unchanged.

This is an offline firmware release. It has not been flashed or executed on an
STM32, and it has not accessed CAN, a drive, feedback hardware or safety IO.

## What materially changed from firmware V7

- Added a maintained native µVision project that directly builds the same 15 C
  sources as the GNU target with Arm Compiler 6.24. The project has its own AC6
  startup/vector and scatter files; it does not wrap CMake, flash or debug.
- Added `firmware_sources.json` as the source/CPU/FPU/ABI/define/memory contract
  and permanent tests that compare it with CMake and `.uvprojx`.
- Made fixed status storage fail closed: only terminal records are reclaimable,
  active-capacity exhaustion is explicit, and duplicate/generation identities
  remain reconcilable.
- Made position-setpoint takeover transactional and terminalized the superseded
  command only after the replacement intent succeeds.
- Made multi-axis stop, protective-stop, disable and disconnect best effort over
  all axes with aggregated failures and per-axis evidence.
- Replaced unbounded deadline catch-up with O(1) advancement and passed actual
  runtime time into the core.
- Made oversize framing immediately latch `RESYNC_REQUIRED`, close the logical
  session and fault the runtime instead of discarding an attacker-declared byte
  count.
- Hardened CANopen reboot/first-heartbeat/observation authority semantics in
  host and virtual tests without adding physical transmit.
- Added versioned/checksummed retained diagnostics, build/toolchain identity,
  reset cause, stacked fault registers and SCB fault status.
- Moved H743 time/fault logic behind a target seam, restored PRIMASK correctly,
  and added a hardware-watchdog port that remains unavailable in the target.
- Bridged valid safety samples and CANopen node fault/timeout to a non-safety-
  rated internal protective stop and lifecycle fault.

## Compatibility

- Protocol: `4.0`, unchanged.
- Windows desktop/API/history/session format: unchanged.
- Raw acquisition authority: counts and the original 16 CSV columns, unchanged.
- Physical target ports: unavailable; nodes unassigned; real transmit/write
  capability false.

## Current verdict

The final local verdict is recorded in
`V8_0_INDEPENDENT_OFFLINE_VERIFICATION_20260801.md`. Cross-build, native Keil,
host and virtual results are not hardware proof. All board, transport, CANopen
network, motion, timing, synchronization, feedback, safety and accuracy claims
remain `BLOCKED_BY_HARDWARE`.
