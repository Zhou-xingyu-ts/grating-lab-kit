# Motion Controller Firmware V7.0 构建与离线验收

## 构建

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File .\packaging\build_native_controller.ps1

powershell -NoProfile -ExecutionPolicy Bypass `
  -File .\packaging\build_stm32h743_firmware.ps1

.\.venv\Scripts\python.exe `
  .\tools\check_embedded_protocol_roundtrip.py `
  .\build\v7_controller\motion_controller_embedded_protocol_stdio_probe.exe
```

ARM build uses the documented portable Arm GNU 15.2.Rel1 input and minimal
official STM32H743 CMSIS/startup set. No flasher or device driver is invoked.

## 离线验收项目

- native C warnings-as-errors and CTest；
- firmware lifecycle/liveness/watchdog；
- bounded frame stream and response egress backpressure；
- Python protocol encoder → embedded decoder/dispatcher/encoder → Python model；
- virtual CANopen boot-up/heartbeat/EMCY/SDO/NMT/timeout；
- default-disabled H743 node/transmit configuration；
- ARM ELF/HEX/BIN/map, memory regions, active runtime symbols and forbidden
  functions；future FDCAN ISR/mainline queue is compiled by the ARM target and
  exercised by host CTest, but is intentionally link-GC'd until a physical
  FDCAN port exists；
- Python regression, historical CSV hashes and release portability；
- standalone firmware package rebuild and Windows onedir smoke。

Windows 发布复验入口为：

```powershell
.\.venv\Scripts\python.exe .\scripts\v7_release_smoke.py `
  .\release\MotionAcquisition --source-root .
```

## 当前电脑最终结果

- native C：warnings-as-errors，CTest `6/6`；
- Python：`149 passed in 21.08s`，`compileall` 通过，`pip check`
  无损坏依赖；
- embedded protocol：7 请求/7 响应、11,921 response bytes、4 events，
  Python 模型全部解码；
- 历史采集：`19/19` 会话、`212,378` 行兼容，源 CSV SHA-256
  改变 `0`；
- H743：text `24,816` B、data `8` B、BSS `71,592` B；
- H743 ELF SHA-256：
  `20DA773D6B729B7D3069998CC16AD3F24B977EED644ED867D496F661920BFCC3`；
- 独立固件源码重编译得到同一 ELF SHA-256；
- Windows GUI、Python SIL、native C helper 隔离冒烟通过，GUI 5 秒内
  未自动启动 helper，固定路径违规 `0`。

发布目录：

```text
release\MotionAcquisition
release\MotionControllerFirmwareV7
```

任何 ARM cross-build、虚拟 CANopen 或当前电脑冒烟结果都不等于上板、
现场总线、运动、安全、精度、同步或第二台 Windows 验证。
