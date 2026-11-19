# V8.0 future hardware bring-up entry

This is a blocked future entry, not permission to connect, flash or move.

## Evidence to obtain before a first flash

- Exact H743 board/baseboard identity, schematic revision, supply, BOOT/reset,
  oscillator and debugger voltage/reference.
- Confirmed memory device and a reviewed clock/cache/MPU policy.
- A lab release record for debugger-only power-up with all motor/drive/CAN
  connections absent or otherwise independently made safe.

## First software sequence after release approval

1. Rebuild the released source through both accepted toolchain lanes and verify
   package hashes and map contracts.
2. Flash only the no-real-port V8 image; do not enable FDCAN or drive output.
3. Verify reset vector, reset cause, build ID, reset count, SysTick monotonicity
   and retained fault block through debugger observation.
4. Deliberately test a benign software fault under a separate approved plan and
   confirm stacked PC/LR/SCB diagnostics and no watchdog refresh.
5. Measure scheduler time/jitter before selecting real task rates.

## Gates before physical communication

- Select and verify the PC transport, buffering, electrical levels and failure
  behavior before binding `ByteStreamPort`.
- Confirm FDCAN instance/pins/clock, external transceiver, CANH/CANL/GND and
  termination before binding the CAN port. MCU logic pins never connect directly
  to CANH/CANL.
- Obtain three actual Node-IDs, identities/firmware and the correct DS-BVM EDS or
  supplier object evidence. Do not reuse virtual IDs.
- Begin with analyzer/loopback and read-only discovery. NMT, SDO download, PDO,
  SYNC, Controlword and mode writes require a new review and authorization.

## Gates before any motion

Feedback authority/count direction/overflow/timestamp, STO, E-stop, brake,
limits, phase order, Hall/encoder wiring, low-energy single-axis release and
independent mechanical containment must all be closed. Software protective stop
and watchdog policy do not satisfy these safety gates.

The second Windows computer, real cycle/synchronization, three-drive network,
motion, precision and robustness all remain `BLOCKED_BY_HARDWARE`.
