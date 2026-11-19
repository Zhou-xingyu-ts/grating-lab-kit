# Motion Acquisition V7.0 Release Notes

V7.0 是 **STM32H743 Offline Controller Integration Candidate**。

2026-07-30～2026-07-31 的独立本机离线复核确认并修复了 helper 清理与
readiness、响应关联、首次断连收尾、重复 JSON、CANopen Node-ID、
重复命令/连接代际、会话外路径和完整性裁决，以及依赖清单范围等 9 类软件
缺陷。修复保持 protocol 4.0、16 列 raw CSV 和版本号不变。详见
`V7_0_INDEPENDENT_OFFLINE_VERIFICATION_20260730.md`。

新增：

- boot/self-test/standby/operational/degraded/fault 生命周期；
- scheduler/realtime/mainline/health liveness 与 latched watchdog fault；
- protocol 4.0 allocation-free response encoder；
- 两帧有界 egress、partial write 和 backpressure；
- Python/C 双向 byte-stream probe；
- C11 atomic SPSC CAN frame queue；
- 三节点 CANopen boot-up/heartbeat/EMCY、只读 SDO upload 和 timeout；
- read/control 两级发送授权与虚拟 NMT；
- 可配置 Scheme A/B/C responsibility；
- 扩展 debugger diagnostics；
- H743 ARM cross-build 中包含生命周期、协议和 active CANopen 运行链；
  无物理 FDCAN 调用点的 queue/NMT 路径由 ARM 源码编译与主机 CTest 证明，
  不人为阻止链接器移除未使用函数。

保持：

- V1～V6 采集、回放、分析、实验数据和控制记录兼容；
- raw counts 与 16 列 `samples.csv` 权威；
- protocol 4.0 和既有桌面模型；
- PySide6 GUI、Python SIL 和 native C reference。

明确未包含：

- 真实板卡/FDCAN/PC transport 驱动；
- 真实三个 Node-ID；
- DS-BVM EDS/PDO/控制模式；
- SDO download、controlword/mode write、PDO/SYNC；
- 反馈/安全 IO 和硬件 watchdog；
- 真实运动、同步、精度或第二台 Windows 证明。

本轮离线加固不改变上述现场边界，也没有增加任何真实设备写控制路径。
