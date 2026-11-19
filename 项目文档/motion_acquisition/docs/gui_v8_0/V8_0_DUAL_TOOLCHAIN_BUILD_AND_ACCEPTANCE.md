# V8.0 dual-toolchain build and acceptance

## Shared contract

`controller_firmware/firmware_sources.json` is the machine-readable contract
for both embedded frontends:

- STM32H743IIT6, Cortex-M7 Thumb;
- FPv5-D16, hard-float ABI;
- C11, `STM32H743xx`, `MC_EMBEDDED_TARGET=1`;
- the same 15 product C translation units and include directories;
- 2 MiB flash, 8 KiB DTCM stack, D1 ordinary RW/ZI, D3 retained diagnostics;
- no application heap, flash/debug step or physical CAN transmit.

Startup and link descriptions are intentionally toolchain-specific. GNU uses
the vendored ST GCC startup and project linker script. Keil uses the reviewed
AC6 C vector/startup and scatter file. Their binary bytes are not expected to
match; their source/config/safety semantics must match.

## GNU lane

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File packaging\build_stm32h743_firmware.ps1 `
  -ArmGnuToolchainRoot "<arm-gnu-15.2-root>"
```

Accepted local tool: Arm GNU Toolchain 15.2.Rel1. The script builds ELF/HEX/BIN/
map, checks load segments, required and forbidden symbols, scans product sources
for forbidden write paths, and writes `FIRMWARE_BUILD_REPORT.json`.

## Native Keil lane

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File packaging\build_stm32h743_keil.ps1 `
  -KeilRoot "<Keil-MDK-root>"
```

Accepted local tools: µVision 5.43.1 and Arm Compiler for Embedded 6.24. The
script invokes `uVision.com -r` on the native target, then uses `fromelf` for
binary/symbol/attribute evidence. It requires 0 errors and 0 warnings and audits
vector values, hard-float attributes, memory regions, heap absence, required/
forbidden symbols, fixed paths and all false hardware/write flags.

The STM32H7 DFP is not installed and is not silently downloaded. Both lanes use
the vendored STM32CubeH7 v1.12.1 / CMSIS Device H7 1.10.6 subset and licenses.

## Acceptance interpretation

`PASS` for a lane means its source compiled, linked and passed the listed local
post-link audits. It does not prove reset execution, clocks, SysTick accuracy,
FDCAN, physical transport, CANopen interoperability, watchdog timing, motion or
safety. Those claims remain `BLOCKED_BY_HARDWARE` until separately released.
