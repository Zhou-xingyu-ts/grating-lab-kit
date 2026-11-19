# Route A 回归检查卡：Windows → PCAN → DS-BVM

## 目的与边界

本卡用于 V9 对既有 Route A 固定单轴现场基线的**离线兼容回归**，以及未来另行授权的现场回归准备。它不是连接 PCAN、发送 CANopen、Servo On 或运动许可。

Route A 的物理基线是 Windows → PCAN → DS-BVM。V9 只能纳管既有行为，不能扩大到多轴、任意轨迹、实时性或新模式。

## 0. 现场前硬门

离线阶段必须全部满足：

- [ ] Route Manager 列举 A 路时没有打开或枚举 PCAN/CAN；
- [ ] A 路在 Offline/SIL 中只能选择 Fake/回放预检；
- [ ] Python SDO 下载调用仍仅存在于 dsbvm_direct.py；
- [ ] 旧独立直控窗口和 motion-acq-direct 入口仍存在且不被 Route Manager 替换；
- [ ] 原始正式会话、16 列 CSV 和 checksums.sha256 没有字节变化；
- [ ] 路线状态 unknown 时禁止切换或下达后续命令；
- [ ] 本卡对应的定向测试、全量回归和哈希结果已填入 V9 离线验证报告。

未逐项完成前，不得填写 READY_FOR_LAB_TEST。

## 1. Fake/回放回归项目

| 项目 | 预期离线判据 | 证据位置 | 结果 |
|---|---|---|---|
| 固定计划 | 仅覆盖既有 +5000/-5000 counts 固定往返语义，不增加新动作 | [待填] | [待填] |
| 起始顺序 | Fake SDO/NMT 写入与既有 DirectMotionAcquisitionService/DsBvmDirectController 逐项一致 | [待填] | [待填] |
| 采集 | 仍为既有七对象采集，控制与采集使用同一 CANopen client | [待填] | [待填] |
| 到位判据 | 保持 bit12 的诚实记录与 bit10、位置变化、最终到位的既有兼容逻辑 | [待填] | [待填] |
| 正常结束 | 调用既有正常收尾，而非通用 protective-stop | [待填] | [待填] |
| 主动停止 | 调用既有 Halt→Disable 语义 | [待填] | [待填] |
| 异常结束 | 调用既有异常收尾；无法确认 Disabled 时进入 unknown | [待填] | [待填] |
| 历史会话 | 2026-08-01 正式会话 checksum、完整性、回放和分析仍通过 | [待填] | [待填] |
| 独立入口 | 直控窗口和 motion-acq-direct 行为不因 Route Manager 改变 | [待填] | [待填] |

## 2. 路线切换检查

1. 在 Route A 还处于 running、stopping、fault 或 unknown 时，Route Manager 必须拒绝切换到 B/C。
2. 只有 Route A 的专属关闭完成且 Disabled 已确认时，才可释放共享主站锁。
3. 若关闭确认丢失，界面须显示“使能状态未知”；用户只能记录故障并走后续现场处置，不能用重启 GUI 伪造关闭成功。

## 3. 未来现场回归的单独授权门

本节只列出未来所需证据，**不授权执行**：

| 门 | 需要的单独证据/批准 |
|---|---|
| 台架 | 父工作区上电放行、线路验收、未闭合缺口的适用关闭记录 |
| 主站排他 | GM1/STM32/其他 PCAN 主站均不在控制共享底座 |
| 设备与网络 | PCAN、Node ID、bitrate、DS-BVM identity 和总线电气的现场确认 |
| 动作 | 对每一项 NMT、CiA 402、SDO 下载、启动、停止和运动的逐动作授权 |
| 安全 | STO/E-stop、制动、限位、机械约束和人员分工的现场放行 |

即使 V9 离线回归全部通过，现场动作也必须在本卡之外另行得到明确授权。

## 4. 签核与证据

| 项目 | 待填写 |
|---|---|
| 源码基线/哈希 | [待填] |
| Fake/回放测试记录 | [待填] |
| 全量回归记录 | [待填] |
| 正式会话 checksum 对账 | [待填] |
| 无硬件 I/O 声明 | [待填：本轮未访问真实硬件] |
| 残余限制 | [待填] |
