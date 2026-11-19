# 阶段 B1 交接

## 1. 五条最重要的官方能力结论

1. **DS-BVM-FETC/FCAO 具有数字、模拟和串行三类反馈口。** 数字口是一套 A/B/I 加 U/V/W Hall；模拟口含 Sin/Cos、模拟 Hall 和 AIN；串行口支持 SSI/BiSS-C。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF pp.4–11，§1.2、§2.2、§2.5、§2.6、§2.8。证明接口能力；不证明并发使用、当前配置或台架验证。
2. **系列固件明确支持 Single Feedback / Dual Feedback。** 双反馈时 Feedback1 用于 voltage/current/speed，Feedback2 用于 position；组合受兼容矩阵限制。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.16、23，§4.2、§4.2.1(11)。证明双反馈控制职责；不证明当前 DS-BVM 能直接接两套增量编码器。
3. **`0x6062` 是位置控制器 demand，`0x6064` 是传感器测得实际位置，`0x60F4` 是跟随误差实际值；`0x6063` 没有足够正文定义。** 证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.154–156、176，§6.2.96–6.2.99、§6.2.141。不能把 `6063/6064` 擅自解释为两只反馈，也不能擅写 `60F4` 的符号公式。
4. **PPM/PVM 把轨迹生成保留在驱动器；CSP 把轨迹交给外部但保留位置/速度/电流环；CSV 再把位置环交给外部；CST 再把速度环交给外部。** 证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.29–47，§5.3–5.7。证明模式控制权边界；不证明当前模式、PDO 或实时外环已实现。
5. **CANopen 支持 4 RPDO + 4 TPDO、同步/异步传输和 CiA 402 状态机，但默认可配置 PDO 映射数量为 0。** PDO 还要求合适的 NMT 状态、映射、transmission type 和驱动状态。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.13–27、81–92、140–154。证明协议能力；不证明当前已有 PDO 控制链。

## 2. 三条最容易被误解的手册表述

1. “有 M6/M7/M8 三个反馈口”不等于“三路反馈可同时参加闭环”。
2. “CSP/CSV/CST 每个 PDO 更新周期续写目标”不等于“官方保证 1 ms / 1 kHz”。
3. “Dual Feedback”不等于“`0x6063` 和 `0x6064` 分别对应电机编码器和光栅尺”。

## 3. 四种候选拓扑初步等级

| 拓扑 | 等级 | 一句话边界 |
|---|---|---|
| 光栅尺直接进入 DS-BVM，当前单反馈闭环 | **手册明确支持（能力层）** | Incremental 单反馈和内部位置模式均有官方依据；当前参数、速度/换相来源仍未验证 |
| 光栅尺进入 STM32，STM32 外位置环 + DS-BVM CSV | **手册层面可能可行** | CSV 明确保留速度/电流环，但需要独立电机反馈、周期 PDO/SYNC 和失联安全证据 |
| 光栅尺进入 STM32，STM32 位置/速度环 + DS-BVM CST | **手册层面可能可行** | CST 明确保留电流/转矩环，但实时性和安全要求最高，且无官方周期保证 |
| STM32 外部轨迹规划 + DS-BVM CSP | **证据不足；若两路反馈均为增量式，另有接口冲突** | 光栅只进 STM32 时不会自动成为 DS 位置反馈；若进 DS 做 Feedback2，则型号级第二套 A/B/I 端口未找到 |

## 4. 官方周期保证结论

**未找到官方 1 kHz 保证，也未找到可作为保证使用的最小 PDO 周期、最大 PDO 频率、SYNC 同步误差或最小更新时间。**

- 10 MHz 是数字编码器输入频率，不是控制周期。
- 24 kHz 是模拟 Hall/Sin-Cos 采样频率，不是总线或位置环周期。
- CANopen event timer 的 1 ms 单位和约 2 ms 发送精度不是同步闭环保证。
- EtherCAT ESI 的 `0x60C2=2×10^-3 s` 是候选 XML 的默认插补设置，不是 500 Hz 性能保证；DC/Minimum Cycle Time 字段默认 0 也不能解释为无限快。

## 5. 输出文件

1. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/00_MANUAL_SOURCE_INVENTORY.md`
2. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/01_FEEDBACK_INTERFACE_AUDIT.md`
3. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/02_POSITION_OBJECT_SEMANTICS.md`
4. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/03_OPERATION_MODE_BOUNDARIES.md`
5. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/04_PDO_SYNC_AND_CIA402_AUDIT.md`
6. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/05_CANDIDATE_TOPOLOGY_FEASIBILITY.md`
7. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/06_MANUAL_AMBIGUITIES_AND_EVIDENCE_GAPS.md`
8. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_B1_dsbvm_manual/HANDOFF.md`

## 6. 阶段 B2 最应核查的五项内容

1. **“1 ms 中断”到底是设计、源码还是实物证据：** 定时器配置、任务最坏执行时间、实测抖动、漏周期统计分别有什么证据。
2. **CANopen PDO 链是否真实完整：** NMT Operational、`0x1600/0x1A00` 映射、transmission type、SYNC、`0x6040/0x6041` 状态机和目标对象写入是否都在源码且上台架验证。
3. **师兄平台的反馈与换相拓扑：** 电机编码器、Hall、光栅尺分别接哪里；位置、速度、电角度各来自何处；是否与当前 DS-BVM 型号端口一致。
4. **控制算法的证据层级：** PID、前馈、LQR 等分别属于公式、仿真、代码、HIL 还是实物运动结果；对应 DS-BVM 的哪种模式和对象。
5. **异常与安全处置：** 总线超时、PDO 丢帧、光栅计数异常、目标越界、Quick Stop/STO/限位如何处理，论文与源码是否一致。
