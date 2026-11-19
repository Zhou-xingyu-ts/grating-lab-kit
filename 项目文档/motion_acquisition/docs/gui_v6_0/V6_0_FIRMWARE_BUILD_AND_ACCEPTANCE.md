# Motion Controller Firmware V6.0 本机构建与离线验收

## 工具和依赖

- Arm GNU Toolchain 15.2.Rel1，GCC 15.2.1；
- CMake 4.4.0；
- Ninja；
- STM32CubeH7 v1.12.1 commit
  `f5c0b7a2b1f6eb26fde150f72edb2d7deb647066`；
- CMSIS Device H7 commit
  `e8d40ae6e2fa06afe5b46d24756f141b363342a7`。

Arm 工具链由官方 Windows x86_64 ZIP 提供，下载 SHA-256：

```text
7936cac895611023ffb22a64b8e426098c7104cb689778c1894572ca840b9ece
```

项目只 vendor 了精确构建所需的 CMSIS core、H743 device header、system
template、startup 和许可证；不包含 HAL、BSP、CubeMX application 或毕业归档
应用。

## 一键交叉构建

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File .\packaging\build_stm32h743_firmware.ps1
```

脚本默认从当前 Windows 用户的 LocalAppData 查找 portable toolchain，也允许：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass `
  -File .\packaging\build_stm32h743_firmware.ps1 `
  -ArmGnuToolchainRoot "<toolchain-root>" `
  -BuildDir "<build-dir>"
```

不依赖固定用户名、盘符或源码路径。

## 输出

```text
build/v6_firmware/stm32h743/
  MotionControllerFirmwareH743.elf
  MotionControllerFirmwareH743.hex
  MotionControllerFirmwareH743.bin
  MotionControllerFirmwareH743.map
  FIRMWARE_BUILD_REPORT.json
```

`FIRMWARE_BUILD_REPORT.json` 固化以下审计：

- ARM ELF、required core/protocol/runtime/diagnostic symbols；
- Flash LOAD segment 非 RWE；
- `malloc/calloc/realloc/free/sbrk/snprintf` 不存在；
- CANopen download/NMT send/FDCAN transmit symbol 不存在；
- production source 没有具体设备写调用；
- `hardware_accessed=false`、`flashed=false`；
- 每个产物的 bytes 和 SHA-256；
- 仍需硬件验证的清单。

## 当前构建指标

```text
text: 13,912 bytes
data: 8 bytes
bss: 53,976 bytes
Flash load: 13,924 bytes / 2 MiB
D1 RAM: 53,840 bytes / 512 KiB
D3 diagnostics: 144 bytes / 64 KiB
```

DTCM 当前只作为运行时栈空间，不以静态 section 假装占满 128 KiB。

## 当前离线测试

| 检查 | 结果 |
|---|---|
| Host C warnings-as-errors build | 通过 |
| CTest core/framing/runtime/protocol | 4/4 |
| Python 全量回归 | 145/145 |
| Python protocol 4.0 → embedded C decoder | 通过 |
| ARM cross-build | ELF/HEX/BIN/map 全部生成 |
| ELF segment | RX Flash；RW RAM；无 RWE |
| Required symbols | 全部存在 |
| Heap/stdio/write symbols | 不存在 |
| 硬件访问/烧录 | 未执行 |

最终发布目录、SHA-256 和发布后复验记录在本文件末尾的“最终发布记录”中。

## 离线验收能证明什么

- portable core 确实进入 H743 ARM 镜像；
- C/ASM/CMSIS/linker/toolchain 是一致可构建的；
- 嵌入式内存有界，且无 application heap；
- protocol request、command ID/status 和 runtime 边界可在主机验证；
- hardware-specific code 可以在明确端口后补入。

## 离线验收不能证明什么

- ELF 能在当前实际板卡启动；
- HSI/SysTick、cache/MPU、IRQ 和 deadline 的真实行为；
- FDCAN/CANopen、三个驱动器或 PC 物理通信；
- PDO/SYNC、循环频率或三轴同步；
- 反馈来源、单位、标定、方向和精度；
- watchdog、E-stop、STO、制动和限位安全；
- 任何电机运动、定位或鲁棒性；
- 第二台 Windows 电脑部署。

## 最终发布记录

验收日期：2026-07-29。证据范围仅为当前 Windows 构建电脑和软件在环。

### 固件产物

```text
MotionControllerFirmwareH743.elf
  33,408 bytes
  SHA-256 8CF4607C0F1743D3C7D5B505921B0A51ED61A2D59D08A81915CF7BDE8E1C82CF
MotionControllerFirmwareH743.hex
  39,227 bytes
  SHA-256 FD82FB437F689411070943A83AD8D7C3E8F48063E4497C2BC6A903D9F1EE7106
MotionControllerFirmwareH743.bin
  13,924 bytes
  SHA-256 F0DA111295D872EF1A990F5BDB05B2C25F76BA355B1466B30644CEC98B73508A
```

独立固件发布包位于 `release/MotionControllerFirmwareV6/`，含产物、sanitized
map/build report、最小可复编源码、官方许可证、构建入口、说明和
`SHA256SUMS.txt`。从该包自己的 `source/` 目录复编得到同一 ELF SHA-256，
证明固件构建不依赖工作区源码路径；使用者仍需自行提供文档指定的 Arm GNU
工具链。

### 测试和兼容

| 证据 | 最终结果 |
|---|---|
| Python 全量回归 | 145 passed |
| CTest | 4/4 passed |
| Python protocol 4.0 → embedded C decoder | passed |
| compileall / pip check | passed / no broken requirements |
| A/B/C、跨进程、timeout SIL | 4/4 durable sessions verified |
| native C completed/stopped/error | 3/3 durable sessions verified |
| 历史采集会话 | 19/19 compatible；212,378 rows |
| 历史原始 CSV | SHA-256 changes = 0 |
| 固件包独立复编 | same ELF SHA-256 |
| Windows onedir GUI | 隔离用户目录、offscreen 连续运行 5 s |
| GUI 未连接自动 helper | 0 |
| 两个发布版 helper | capability/connect/home/setpoint/protective-stop/disconnect passed |
| 发布目录固定路径 | 475 files scanned；0 violations |

发布版 helper 均报告
`simulated=true`、`real_hardware_access=false`、
`device_write_capability=false`；protective stop 明确不是安全等级功能。

### 发布目录

- GUI onedir：`release/MotionAcquisition/`；
- 独立固件包：`release/MotionControllerFirmwareV6/`；
- 两层 `SHA256SUMS.txt` 分别是 Windows 发布目录和固件包的权威校验入口；
- `scripts/v6_release_smoke.py` 可复验 manifest、GUI、两个 helper、固定路径和
  固件 ELF。

发布包已完成本机离线验收，但没有下载到 STM32，也没有访问 CAN、USB、驱动器
或电机。第二台 Windows 电脑和全部现场事项仍延期。
