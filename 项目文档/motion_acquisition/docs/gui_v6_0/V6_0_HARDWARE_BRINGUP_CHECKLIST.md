# Motion Controller Firmware V6.0 未来上板联调清单

本清单是未来现场工作的入口，不表示当前已经放行。V6 阶段禁止烧录、接线、
CAN 访问和运动。

## Gate 0：资料关闭

- [ ] 确认开发板完整型号、原理图、H743 封装和芯片 revision；
- [ ] 确认 HSE/HSI、供电、boot pin、SWD 探针和调试器；
- [ ] 确认 FDCAN instance、TX/RX pin、AF、收发器型号、standby/silent pin；
- [ ] 确认 CANH/CANL/GND、隔离、终端和静态电压；
- [ ] 确认 PC—STM32 选择 UART、USB 或 Ethernet 及其引脚/电气；
- [ ] 确认三台驱动器型号、固件、Node-ID、bitrate 和官方 EDS/对象字典；
- [ ] 确认反馈接入驱动器还是 STM32，以及供电、电平、接口和方向；
- [ ] 确认 E-stop、STO、制动、限位、watchdog 和安全责任；
- [ ] 完成当前执行层的上电前放行，而不是引用本软件文档代替。

任何一项会改变板卡、闭环位置、反馈来源或安全方案时，先更新 capability/profile
和端口设计，不直接在 core 中写条件分支。

## Gate 1：只启动，不接 CAN/电机

- [ ] 由 CubeMX/官方 BSP 生成与实际板卡一致的时钟和 pin 基线；
- [ ] 将 V6 runtime 作为 application module 接入，不改 core API；
- [ ] 首次只烧录无 CAN/无驱动 write 的诊断版本；
- [ ] 用 SWD 检查 Reset_Handler、main、SysTick 和 fault handler；
- [ ] 检查 `g_motion_controller_diagnostics` magic/version/reset/fault；
- [ ] 测量实际时钟和周期，不用交叉构建配置代替；
- [ ] 配置并验证硬件 watchdog；不允许 Windows 喂安全 watchdog。

## Gate 2：PC transport，只握手

- [ ] 选定并记录 byte transport、baud/USB class/IP、buffer 和超时；
- [ ] 实现 bounded read/write port 和 response envelope encoder；
- [ ] capability 仍声明 hardware/write false；
- [ ] 验证断帧、粘包、超长、重连、boot ID、connection ID；
- [ ] GUI 卡顿/断开不得影响实时 task；
- [ ] transport 失败时不得触发任何设备写。

## Gate 3：CAN passive/read-only discovery

- [ ] 先验证 transceiver、termination、bitrate 和接收错误计数；
- [ ] 不发送 NMT/PDO/SDO download；
- [ ] 只读取 boot-up/heartbeat 和已批准的 read-only identity/status；
- [ ] 逐台确认实际 Node-ID，不把测试 fixture 1/2/3 视为现场配置；
- [ ] 读取并记录 0x1018、0x1000、0x1001、0x6041、0x6061、0x6064、
  0x6502 等已批准对象；
- [ ] 明确 0x6064 为 0 时可能是反馈未配置，不能直接判断静止；
- [ ] 任一节点失联、身份变化或状态未知时保持无运动权限。

## Gate 4：反馈和安全

- [ ] 确认每轴 feedback source、raw unit、方向、溢出和 validity；
- [ ] 对 STM32 直读反馈验证计数、采样时刻和丢样；
- [ ] 对驱动反馈验证 PDO/SDO 来源和更新时间；
- [ ] 检查 limits/STO/E-stop/watchdog 的电平、极性和 fail-safe 状态；
- [ ] 物理安全链必须独立于 PC 和普通 CANopen；
- [ ] protective stop、controlled stop、quick stop、STO 的语义分别定义。

## Gate 5：未来写控制（不属于 V6）

只有新版本、独立审批和现场安全条件同时满足后才能开始：

- reviewed CiA 402 state transition；
- exact object/PDO map；
- startup/shutdown/recovery policy；
- mode、unit、limit 和 trajectory policy；
- 先单轴、无负载/低风险，再三轴；
- 独立 E-stop/STO 和可恢复故障流程；
- 每次写动作可追踪、可中止、可对账。

V6 固件本身没有这些写路径，不能直接进入 Gate 5。
