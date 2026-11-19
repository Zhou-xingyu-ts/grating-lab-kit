# Motion Controller Firmware V8.0.0 independent offline verification

## Final verdict

**PASS_WITH_LIMITATIONS**

V8.0.0 is accepted as the new local offline STM32H743 controller-firmware
baseline. It is materially stronger than the V7 firmware candidate and is
maintainable through both the Arm GNU and native Keil/µVision lanes. No open
blocking or high-severity local software defect remains in the reviewed scope.

This verdict does not change the Windows desktop version: Motion Acquisition
remains V7.0.0. Protocol 4.0 and the authoritative raw-count 16-column
`samples.csv` contract are unchanged.

## Why this implementation route was selected

The existing portable core, embedded protocol/runtime and typed ports were
cohesive and already independently tested. A rewrite or GNU/Keil code fork
would have increased risk. V8 therefore keeps one business implementation and
adds only toolchain-specific startup/link seams plus evidence-backed local
repairs. The installed H7 Device Family Pack is absent, so the project uses the
already vendored, provenance-pinned STM32CubeH7/CMSIS subset rather than silently
downloading or copying an unknown framework. No local board/HAL/FreeRTOS/FDCAN
asset was strong enough to justify inventing a physical target port.

## Final software evidence

| Object | Result | Verdict |
|---|---:|---|
| Python full regression | 170 passed | PASS |
| Host CTest | 7/7 | PASS |
| ASan/UBSan CTest | 7/7 | PASS |
| V8 Keil/source/config regression | included in Python; exact paths/hashes | PASS |
| Historical/session focused tests | 5 passed | PASS |
| Frozen historical file hashes | 60 checked, 0 mismatches | PASS |
| `compileall` | passed | PASS |
| `pip check` | no broken requirements | PASS |
| Independent post-integration review | no critical/high local blocker | PASS_WITH_LIMITATIONS |

The standard C suite gained a behavioral H743 fault-frame test, so V8 has seven
CTest programs rather than the six in the V7 baseline. The full Python count
also includes seven V8 native-project/report checks.

## Dual-toolchain evidence

Both final reports bind every shared C source by individual SHA-256 and share
this ordered 15-source aggregate:

`ABC8F2E8DBA777DEB41EA2EDA1D3FBD8E037978E923A7A6F55DB03921302253F`

### Arm GNU 15.2.1

- ELF SHA-256:
  `3F820668BE0131AE8719EFC104E69787ACFA238FBD4B4F1744C9A175FB6A6D16`
- BIN SHA-256:
  `51B3A5703FEFE9BEC0C7D95EC35CE0A767A91E3A56663AD494128F5FF9A81CA4`
- size: text 29,092 bytes; data 8 bytes; BSS 71,880 bytes;
- strict compile, load-segment, required/forbidden symbol and source-write gates
  passed.

### µVision 5.43.1 / Arm Compiler 6.24

- native debug AXF SHA-256:
  `8DC390C81F98AD52EF185CE181D8E9B82CC307E7FCE4DF0C38F2AC8191E5DA0B`
- BIN SHA-256:
  `077F50E1E97033417624B8144AA5EA206EA77A153DA103BB01675998A788E464`
- 0 errors, 0 warnings; Code 22,648, RO 5,820, RW 592, ZI 79,488 bytes;
- initial SP `0x20020000`, Thumb reset vector, hard-float FPv5-D16, 8 KiB
  DTCM stack, D1 RW/ZI, D3 UNINIT diagnostics and zero application heap passed;
- no pre/post wrapper, flash or debug action exists in the project.

GNU and Keil bytes are not expected to match. The standalone package rebuild
does prove a byte-identical GNU ELF and byte-identical Keil HEX/BIN load images.
The Keil debug AXF itself contains build-path debug strings, so the distributable
copy replaces only equal-length path bytes and is then checked by `fromelf` to
produce the exact accepted BIN.

## Substantive firmware advances

- Command identity/capacity and setpoint ownership are fail closed.
- Stop/protective-stop/disable/disconnect attempt all three axes and retain
  partial-failure evidence.
- Deadline catch-up is O(1) and uses actual runtime time.
- Oversize PC framing immediately requires resynchronization and faults/closes
  the session.
- CANopen boot-up, first heartbeat, raw-observation authority and EMCY fault
  epochs are consistent under host/virtual execution.
- Safety/CANopen health produces a non-clearable internal protective-stop latch,
  retries all axes, reports `ERROR/FAULT`, and gates future command ingress.
- A hardware-watchdog callback can only refresh after all required liveness and
  a current valid healthy safety sample; the H743 target leaves it unavailable.
- Retained D3 diagnostics are versioned/checksummed and capture reset/build/
  toolchain identity plus stacked and SCB fault context.
- H743 time reads restore PRIMASK exactly; basic and FP-extended exception frame
  addresses are alignment/range checked.

The detailed fixed ledger is `V8_0_DEFECT_LEDGER.csv`.

## Release boundary

The release is `release/MotionControllerFirmwareV8` and contains separate GNU
and Keil artifacts, raw-build reports, a normalized comparison, source, native
project, build scripts, licenses, documentation and `SHA256SUMS.txt`. The source
package rebuild was exercised through both toolchains. The Windows onedir was
not renamed or rebuilt as V8 because no desktop product feature/version changed;
the existing independently verified V7 desktop remains the compatible host.

## Precise limitations

### PARTIALLY_VERIFIED

- Fault capture: address/decode logic is behaviorally tested and target code
  builds in both compilers, but no real Cortex-M fault has executed.
- Hardware watchdog: software gating is tested, but no board watchdog is bound,
  timed or observed.

Repeated failed health-stop retries intentionally prioritize continued stop
attempts; they can overwrite the bounded event ring quickly. Counters and the
latest events remain, but hardware integration should add transition/rate-limited
telemetry without reducing retry frequency.

### BLOCKED_BY_HARDWARE

- board identity, clock, cache/MPU, reset execution, SysTick accuracy and jitter;
- PC UART/USB/Ethernet transport;
- FDCAN instance/pins/clock, transceiver, termination and physical frames;
- actual Node-IDs, drive identity/firmware, DS-BVM EDS/object dictionary;
- NMT, SDO download, PDO, SYNC, Controlword, mode writes and real drive commands;
- feedback wiring/authority/scaling, raw-count direction and timestamps;
- E-stop, STO, brake, limits and hardware watchdog;
- single-axis or three-axis motion, cycle/synchronization, robustness, accuracy;
- second Windows computer.

Protective stop remains a requested/latching non-safety-rated software intent.
It is not confirmation that a physical axis stopped and is not E-stop or STO.

## Next approved starting point when hardware becomes available

Start with `V8_0_HARDWARE_BRINGUP_ENTRY.md`: confirm the exact board and safety
release, rebuild both lanes, flash only the unavailable-port image, inspect
build/reset/fault diagnostics and measure the scheduler before binding any PC
transport or FDCAN port. Do not begin with drive writes or motion.
