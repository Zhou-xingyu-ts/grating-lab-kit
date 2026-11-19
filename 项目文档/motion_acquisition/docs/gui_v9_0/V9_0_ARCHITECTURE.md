# Motion Acquisition V9.0.0 架构与证据边界

## 1. 目标

V9 在保留既有采集、回放、分析、SIL 与 Route A 独立直控入口的前提下，增加轻量 Route Manager。它统一描述三条**物理**路线，但不重写既有 CANopen 直控核心，也不把 PC 变为 EtherCAT 主站。

~~~text
Route A  Motion Acquisition → Windows/PCAN → DS-BVM
Route B  Motion Acquisition → Windows/serial transport → STM32H743 → CANopen → DS-BVM
Route C  Motion Acquisition → Windows/Modbus TCP → Panasonic GM1 → EtherCAT → DS-BVM
~~~

V9 的软件目标是无真实设备 I/O 的候选实现。默认 Offline/SIL，所有真实端口、写权限和运动入口均应保持关闭或 unavailable。

## 2. 术语与兼容层

### 2.1 物理路线

物理路线 ID 固定为：

| ID | 名称 | 主站所有者 |
|---|---|---|
| windows_pcan_direct | Route A：Windows-PCAN 直连 | Windows/PCAN |
| stm32_canopen_bridge | Route B：STM32 CANopen bridge | STM32H743 |
| gm1_ethercat_controller | Route C：GM1 EtherCAT controller | Panasonic GM1 |

### 2.2 闭环职责档案

旧 SCHEME_A、SCHEME_B、SCHEME_C 及其 profile ID 是历史“闭环职责档案”：它们表达位置环/速度接口等控制责任划分，**不是** Route A/B/C 的物理别名。V9 必须继续读取旧记录和常量，但 GUI、manifest 和新文档统一称其为“闭环职责档案”。

## 3. 统一路线模型

### 3.1 公共对象

| 对象 | 责任 | 不得承担的责任 |
|---|---|---|
| RouteDescriptor | 拓扑、主站、支持轴、能力、证据等级、资源锁和停止策略的静态描述 | 建立连接、硬件探测或写命令 |
| RouteState | unavailable、disconnected、connected_readonly、preflight_passed、armed、running、stopping、closed、fault、unknown 的可审计状态 | 掩盖未确认状态 |
| RouteFactory | 根据配置与策略创建 ActiveRoute | 在路线枚举时打开端点 |
| ActiveRoute | 可选暴露 ControllerAdapter、TimestampedReadOnlySource、preflight、shutdown 和能力/时间/证据快照 | 绕过端点策略或隐式扩大权限 |
| RouteManager | 注册、配置校验、会话排他、路线切换、状态展示、预检快照与显式证据记录交接 | 实现另一套电机控制器 |

ControllerAdapter 和 TimestampedReadOnlySource 继续沿用既有工程契约。路线若不具备某能力，应显式返回 unavailable/None，并说明原因；不得用伪造的“已连接”代替。

### 3.2 状态和资源锁

~~~text
unavailable ──(配置与策略满足)──> disconnected
disconnected ──(受控连接)──────> connected_readonly
connected_readonly ──(预检通过)─> preflight_passed
preflight_passed ──(明确授权)───> armed ──> running
running ──(路线专属关闭)────────> stopping ──> closed

任何未确认的超时、断线、重启、异常收尾或安全状态
  └──────────────────────────────────────> unknown / fault
~~~

- 同一时刻 Route Manager 只能持有一个共享底座运动主站锁。
- 切换前必须调用当前路线专属 shutdown 并确认已关闭。
- 对 Route A，既有 Halt→Disable 语义不可被通用“请求保护停机”替代。
- 无法确认使能、命令结果或关闭状态时，进入 unknown；此时禁止切换、重连后自动重发或继续命令。
- 仅查询路线列表、配置完整性或证据等级时不得进行 I/O、端口枚举或网络探测。

## 4. 各路线的责任边界

### 4.1 Route A：Windows-PCAN 直连基线

现场 Route A 仍只由既有 DirectMotionAcquisitionService 与 DsBvmDirectController 执行，不重新实现其 CiA 402 流程。V9 Route Manager 的 Route A facade 不实例化或调用该直控服务；既有独立窗口和 motion-acq-direct 入口保持不变。

V9 的 Route A 包装层只能：

- 在 Offline/SIL 中提供 Fake/回放预检；
- 将控制与采集共用同一个 CANopen client、固定 profile、七对象采集、bit10/bit12 判据和 Halt→Disable 收尾保留为既有现场基线约束；
- 不把离线关闭伪称为驱动器 Disabled 已确认；真实入口的使能状态无法确认时仍必须标记“使能状态未知”。

Python 真实 SDO 下载入口仍仅允许存在于 dsbvm_direct.py。Route Manager 或其他新模块不得直接导入 python-can/canopen 写接口。

### 4.2 Route B：STM32H743 CANopen 主推候选

PC 只下发 protocol 4.0 的高层任务；STM32 是 CANopen 主站。主机端的帧关联、命令 ID、sequence、ACK、最终状态、事件、boot ID 和断线处理必须与既有 protocol 4.0 兼容，socket 与字节传输实现分离。

- FramedByteTransport 可由 In-memory Fake、串口、USB CDC、ST-LINK VCP 或 UART 承载，但串口模块必须惰性导入，缺少显式端口/波特率即为 unavailable。
- 超时或 boot ID 改变后的未确认命令为 unknown，绝不自动重发运动命令。
- 固件安全默认镜像：权限 locked、节点未分配、真实端口 unavailable。
- Node ID、bitrate、对象能力、PDO 映射和周期都来自配置；Node 1 与 1 Mbps 只能作为实验模板，不能成为默认硬件事实。
- 板型、引脚、时钟和收发器未证实前，STM32 FDCAN 端口不得变为可发送端口。MCU FDCAN 逻辑脚绝不能直连 CANH/CANL。

Virtual DS-BVM 用于帧级 boot-up、heartbeat、EMCY、SDO、abort、状态机、PPM、PDO/SYNC 与故障场景的离线验证。虚拟帧不能视作真实总线证据。

### 4.3 Route C：GM1/EtherCAT 候选

Route C 的 PC 端适配器只与 GM1 的 Modbus TCP 契约交互。GM1 负责 EtherCAT 主站工作，Windows 不安装或启用 EtherCAT master。

gm1_route_v1 契约至少包括：

- magic、契约版本、GM1 工程 build ID、boot counter；
- 心跳、控制器状态、安全状态和逐轴状态；
- 高字在前的 32 位有符号 raw counts；
- command sequence、kind、参数、提交标志；
- ACK sequence、执行阶段、错误码和完成状态。

magic、版本、build ID、心跳或序列不匹配时，只能显示诊断，必须 fail-closed。历史 GM1 IP、寄存器、线圈地址及位置偏移算法都是历史线索，不能复制为 V9 当前配置。

ESI/XML 审计器只读取原件并生成新报告。其输出必须分别列出设备 identity、PDO、DC、对象映射、启动命令和 InitCmd 写操作；特别是 SafeOp→OP 的 InitCmd 不能被描述为天然只读。

## 5. 时间、能力与证据快照

每个 ActiveRoute 应输出下列不变快照，并被写入可选 route 元数据：

| 类别 | 最小字段 |
|---|---|
| 路线 | route_id、topology、master_owner、adapter、能力集合 |
| 权限 | real_hardware_access、device_write_capability、authority_level、预检结论 |
| 时间 | host 时间源、设备时间语义、同步是否已证实、时间戳边界 |
| 证据 | 证据等级、来源、已知限制、缺失证据、配置摘要 |
| 会话 | 控制记录关联、独立采集会话 SHA-256、关闭结果 |

离线/SIL 会话必须明确 real_hardware_access=false 且 device_write_capability=false。B/C 的控制反馈属于既有控制会话流，并以 SHA-256 关联独立采集会话；不得强行写入历史 CANopen SDO CSV。Route A 保持其原有控制—采集合并会话。

Route Manager 本身在列举、创建或预检时不落盘。只有用户在预检通过后按“保存离线预检记录”时，桌面端才会在 `route_preflight_records/` 下写入一份明确标注为“无样本、无设备 I/O”的 16 列兼容记录及 route metadata。该记录是预检证据，不是采集数据，也不替代 B/C 后续的控制记录—采集会话 SHA-256 关联。

## 6. 不变的数据和发布边界

- 16 列 samples.csv、raw counts、旧解释和每会话 checksum 不变。
- manifest/control record 只允许增加可选 route 元数据；旧 control_route=windows_pcan_ds_bvm 必须继续被识别为 Route A。
- 完整性判定按路线白名单、精确权限与完整控制日志执行，不能因 B/C 引入而全局放宽。
- release 不可手工修改；V9 发布候选必须可独立清点、可哈希、可在源码目录外验证。

## 7. 可声明和不可声明的结论

V9 完成离线验证时可以写 READY_FOR_LAB_TEST，并必须同时列出待现场确认的板型、收发器、总线电气、GM1 工程、设备 identity、PDO/SYNC/DC、反馈与安全项。

在任何离线阶段都不得写成“STM32 已上板”“CANopen/PDO/SYNC 已通过”“EtherCAT 已通过”“GM1 已联机”“三轴已通过”“安全功能已验证”或等义表述。
