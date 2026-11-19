# V8.0 controller architecture and diagnostics

## Stable responsibility split

```text
Motion Acquisition V7 desktop
  protocol 4.0 intent, monitoring and records
        |
Motion Controller Firmware V8 on STM32H743
  fixed-capacity task lifecycle, three-axis coordination,
  scheduler, node health, protocol and diagnostics
        |
typed unavailable ports in this release
        |
future reviewed PC transport / FDCAN / drive / feedback / safety bindings
```

V8 preserves the portable core and typed-port architecture. It does not create
GNU and Keil business implementations, introduce an RTOS, select board pins or
invent a DS-BVM EDS/object map. Scheme A remains the first intended physical
route, while the semantic `DrivePort` and responsibility model preserve the
bounded B/C substitution seam.

## Fail-closed runtime behavior

- Fixed command-status capacity never evicts active records.
- Multi-axis stop paths attempt every axis and retain failure evidence.
- Oversize framing requires an explicit stream reset and closes the session.
- Valid asserted safety state or CANopen timeout/fault requests an internal
  protective stop and latches a lifecycle fault.
- A future hardware-watchdog callback is serviced only after every required
  liveness domain advances and no lifecycle fault/stale mask is present.

Internal/software protective stop remains a non-safety-rated intent. It is not
E-stop, STO, brake control or proof that a physical drive stopped.

## Retained diagnostic block

The `.diagnostics` block is linked as D3 `NOLOAD`/`UNINIT`. Preservation requires
matching magic, schema version, structure size and checksum. It records:

- firmware semantic version, build ID, toolchain ID/version and reset count;
- raw reset-cause bits and last firmware fault;
- exception return, stack pointer, R0-R3/R12/LR/PC/xPSR;
- CFSR, HFSR, MMFAR, BFAR, ICSR and SHCSR;
- port/lifecycle/node/scheduler/frame/health/watchdog counters.

The fault handler validates a candidate stack frame against known H743 RAM
windows before dereferencing it. It then records the context with interrupts
disabled and stops without refreshing the watchdog. These mechanics compile in
both toolchains but remain unexecuted until a separately approved board session.
