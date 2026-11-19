# Motion Acquisition V9.0.0 声明—证据矩阵

## 当前声明—证据矩阵（2026-08-09）

| 声明 | 可使用的准确措辞 | 当前证据 | 状态 |
|---|---|---|---|
| V9 离线软件回归 | “V9 当前 Python 测试集已在本地离线通过。” | 全量 **257 passed / 0 failed / 0 skipped，40.00 s**，最终记录为 `build/v9_offline_completion_20260809_23/FINAL_PYTEST_EXECUTION.md`；定向静态 + ESI **31 passed，1.26 s** | 可用；不外推为现场验证 |
| Route B portable 候选 | “Route B 的 Host CTest、Arm GNU target 与 Keil target 已完成离线构建。” | `build/v9_offline_completion_20260809_15/`；Host CTest 非空且零失败，两个 target 构建成功 | 可用；未烧录、未上板 |
| Route C 官方 ESI 审计 | “本地官方 ESI 的只读审计产物已生成并可按输入 hash 追溯。” | `build/v9_offline_completion_20260809_22/`；输入 SHA-256 `92733290d5adc3fad608b78b174043123cfae44c2c9d834a574c8b10010cb69d`；JSON、Markdown、SHA256SUMS 齐全 | 可用；不代表 GM1/从站现场实配 |
| 最终离线候选 | “最终离线候选已在源码目录外完成离线启动与协议烟雾验证。” | `build/v9_offline_completion_20260809_27/FINAL_CANDIDATE_LAUNCH_VERIFICATION.md`：`MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 的默认 GUI 与未连接 Direct 窗口启动/关闭，SIL localhost protocol 4.0 hello/connect/poll/disconnect 均 PASS；415 条 SHA-256 清单通过，且无固定路径、原始 data、logs 或 CSV | 可用；仅离线软件候选 |
| 硬件访问边界 | “上述证据未访问真实硬件。” | `real_hardware_access=false`、`device_write_capability=false`；未打开、枚举或连接真实端点 | 可用 |
| 最终离线结论 | `READY_FOR_LAB_TEST` | 全部离线软件证据已具备，含外部工作目录验证 | 可用；仅离线软件就绪 |

STM32 板型/收发器/FDCAN 与总线电气、GM1 当前工程与 Modbus 映射、DS-BVM identity/PDO/SYNC/DC、反馈和全部安全/机械项仍是现场待确认项，不能由本矩阵的离线证据关闭。

## 规则

本矩阵将“能说什么”与“需要什么证据”绑定。没有证据编号、哈希、命令输出或现场记录的条目，当前状态必须是“待验证”，不能被叙述性文字替代。

## 历史待填写矩阵（已由当前矩阵替代）

| 声明 | 允许的准确措辞 | 所需证据 | 当前状态 |
|---|---|---|---|
| 路线模型 | “Route Manager 的无 I/O 注册、状态机、资源锁与配置校验已通过离线测试。” | 对应单元/集成测试、全量回归、无端点打开证据 | 待填写 |
| Route A 纳管 | “Route A 仍委托既有 DirectMotionAcquisitionService，离线 Fake/回放回归与历史会话兼容已复核。” | 写序列对比、关闭流程、checksum、回放、独立窗口测试 | 待填写 |
| Route B host | “protocol 4.0 的可替换字节传输在 Fake 环境中覆盖分帧、partial write、重启和 unknown 对账。” | 双向协议测试、无 COM 打开记录 | 待填写 |
| Route B firmware | “共享 portable core 与 Virtual DS-BVM 通过离线测试；安全默认镜像 locked、节点未分配、真实端口 unavailable。” | C 测试、GNU/Keil 产物、权限审计、哈希 | 待填写 |
| Route C host | “gm1_route_v1 在 Mock 环境中覆盖握手、字序、计数、命令对账和 fail-closed 场景。” | Mock 测试与无网络端点证据 | 待填写 |
| Route C ESI | “对本地归档 ESI 的只读审计报告已列出 identity、PDO、DC、InitCmd 写与未闭合项；未发生网络或 EtherCAT I/O。” | 来源说明、输入原件 SHA-256、`ROUTE_C_ESI_AUDIT.json`、Markdown、SHA256SUMS、回归测试 | 待填写 |
| 会话兼容 | “历史 16 列 samples.csv 与控制记录仍可读；新 route 元数据为可选字段。” | 完整性、回放、schema 兼容测试 | 待填写 |
| GUI 门禁 | “V9 GUI 默认 Offline/SIL，真实连接、预备控制和运行入口受禁用。” | GUI 测试/截图、端点 I/O 监测 | 待填写 |
| 默认桌面候选 | “默认桌面可在源码目录外启动，无原始数据夹带、无固定用户路径和无自动 I/O。” | 文件/依赖清单、三 EXE hash、默认启动记录 | 待填写 |
| Direct 回退入口 | “候选含单独命名的 Route A Direct EXE；它未随默认桌面启动，启动窗口本身不自动连接。” | Direct EXE/hash、README_DIRECT_HARDWARE_V9.txt、源码外观察记录 | 待填写 |
| 可选 transport 依赖 | “候选包含 pyserial/pymodbus；Route B/C 仍以运行时 lazy/fail-closed 策略阻止未授权端点 I/O。” | import/bundle 清单、配置/静态审查、默认启动观察记录 | 待填写 |

## 可以引用的历史事实及其边界

| 历史事实 | 允许的准确措辞 | 不允许的扩展 |
|---|---|---|
| Route A 历史记录 | “Windows-PCAN→DS-BVM 的固定单轴路线已有 PASS_WITH_LIMITATIONS 基线。” | 不能写成 Route B/C、三轴、硬实时、STM32 或安全通过 |
| 既有 V8 固件 | “V8 具有离线固件候选和历史验证资料。” | 不能写成已烧录、已连 FDCAN 或已控制 DS-BVM |
| 旧闭环职责档案 | “SCHEME_A/B/C 历史档案仍可读。” | 不能把它们重新命名为物理 Route A/B/C |

## 明确禁止的声明

以下声明在 V9 离线阶段均为禁止项，即使 Mock/SIL/构建通过也不能使用：

- STM32 已上板、已烧录验证或已连接真实串口/FDCAN；
- CANopen、NMT、SDO 下载、CiA 402、PDO/SYNC/DC 已在现场通过；
- GM1 已联机、Modbus 已连接或 EtherCAT 已扫描/进入 OP；
- 本地 ESI 文件 hash 已证明当前 GM1 工程、从站 identity、PDO/DC 实配或任何现场状态；
- 三轴、实时性、精度、反馈、STO/E-stop、制动或机械安全已通过；
- V9 已经允许任意 Route A 新动作、任意轨迹或多轴现场控制。

## 历史结论门模板（已由当前矩阵替代）

| 结论 | 证据门 | 可写文本 |
|---|---|---|
| 实施中 | 任一新增模块或证据尚未完成 | “待离线验证完成；不得进入现场结论。” |
| 离线候选完成 | 全部软件声明有实际证据、全量零失败、发布验证完成、没有真实端点 I/O | “READY_FOR_LAB_TEST” |
| 现场阶段 | 需要另外的逐动作授权和父工作区安全放行 | 本矩阵不提供现场通过措辞 |

## 历史填写栏（已由当前矩阵替代）

| 字段 | 待填写内容 |
|---|---|
| 测试证据编号 | [待填] |
| 构建/发布证据编号 | [待填] |
| 数据 hash 对账编号 | [待填] |
| 无真实 I/O 证明 | [待填] |
| 缺陷表版本与未关闭项 | [待填] |
| 最终裁决日期/执行人 | [待填] |
