# 阶段 A 交接

## 五条最重要结论

1. 当前代理台架已验证 Ri0040 作为 DS-BVM 的单一数字位置反馈，并存在真实的单反馈位置闭环运动记录；本阶段不据此判断未来反馈拓扑。
2. 7 月 12 日的真实分工是 Servo Studio 经 USB/COM 发运动命令，USB-CAN/Python 经 CANopen SDO upload 只读采集；当前 Node-ID 为 1、波特率为 1 Mbps，6 月的 `node 622` 不是并列当前值。
3. `motion_acquisition` 当前真实 CAN 路径只有 SDO upload；没有 PDO、CiA 402 写控制或实时外位置环。0x6040/0x607A 仅存在于 dry-run 预览。
4. 当前数据支持：运动中 7 对象约 50 Hz；Servo Off 静止时 7 对象约 100 Hz、3 对象约 200 Hz。manifest 的 `estimated_rate_hz` 是工作段吞吐口径，不能替代实际轮次率。
5. 当前数据支持方向相关差异和参数敏感性，但不支持因果归因、正式定位精度、自研尺性能、三轴性能或实时闭环能力；`0.5 μm/count` 仍是暂定工程换算。

## 五个最大证据缺口

1. 缺少 counts 与外部位移基准之间的正式标定及不确定度说明。
2. 缺少统一的实际轮次率、周期起点误差和完整调度抖动定义；当前 `deadline_miss_count` 只表示读取工作段 overrun。
3. 缺少逐对象真实读取时刻，同轮对象只能按轮对齐，不能称严格同步采样。
4. 方向不对称只有现象和参数相关性证据，缺少隔离摩擦、重力、间隙、线缆、负载等因素的因果对照。
5. 缺少独立 CAN 总线抓包及 PDO/CiA 402/实时外环实物证据；manifest 安全字段本身只是运行声明。

## 六个输出文件

1. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/00_SOURCE_INVENTORY.md`
2. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/01_CURRENT_SYSTEM_FACTS.md`
3. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/02_MOTION_ACQUISITION_AUDIT.md`
4. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/03_CURRENT_DATA_CLAIM_BOUNDARY.md`
5. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/04_EVIDENCE_GAPS.md`
6. `01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_研究基线审计/phase_A_current_system/HANDOFF.md`

## 第二阶段资料优先级

1. **第一优先：** DS-BVM Hardware Manual / Reference Manual 中反馈接口、对象字典、PDO以及 PPM/PVM/CSP/CSV/CST 通信与运行模式章节，用于确认对象语义、反馈接口边界和模式能力。
2. **第二优先：** 师兄论文与 STM32/CANopen 遗留源码中关于 1 ms 周期、PDO、时间基、状态机和实物验证的对应证据，用于区分“论文/仿真描述”“源码存在”和“真实台架验证”。

本交接不选择未来反馈拓扑或控制算法。
