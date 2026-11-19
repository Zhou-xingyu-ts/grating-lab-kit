# Motion Acquisition V4.0 总体架构审校与决定

日期：2026-07-26  
证据范围：现有 V0.3～V3.0 源码、文档、测试和本机软件在环；未访问 CAN、STM32、驱动器、电机或安全硬件。

## 1. 审校结论

V3.0 的分层方向可以继续使用，但“未来真实适配器无需改 GUI/服务”的结论只成立了一半：

- `GUI -> Qt worker -> ControllerService -> ControllerAdapter` 是正确且足够小的稳定边界；
- 采集、16 列 raw CSV、会话、回放、分析、研究输出与控制会话分离，必须原样保留；
- V3 的具体数据契约仍固定在 mm 位置命令、位置/速度反馈、主机软限位和进程内模拟器；
- V3 文档把位置环、轨迹和同步同时分给“实时控制器”，隐含选择了方案 B，而 V3 模拟器的具体行为又更像方案 A 与理想机构的混合体；
- V3 `accepted` ACK 没有表达命令终态，通信丢失还把速度改写成 0；
- PC `emergency_stop/reset_emergency_stop` 命名夸大了 Windows 软件的安全权限。

V4.0 因此保留分层，升级契约，不重新设计采集和实验数据平台。

## 2. 保留、修改与重写风险

| V3 设计 | V4 决定 | 若不修改的后果 |
|---|---|---|
| GUI/worker/service/adapter 分层 | 保留 | 无 |
| 进程内 `SimulatedControllerAdapter` | 保留为快速单元 SIL；新增同契约跨进程 SIL | 无法验证 framing、握手、进程死亡和重连边界 |
| 16 列 `samples.csv` 与 V2 会话链 | 完全保留 | 修改会破坏历史权威数据 |
| 独立控制会话和 V2 SHA-256 关联 | 保留并增加可选 V4 证据 | 无 |
| 固定 `CommandKind` 高层动词 | 旧动词保留解码；增加接口化 setpoint/program | 速度/转矩方案会重写服务和 GUI |
| `TaskStep.targets` 位置字典 | 保留旧任务；新增 `axis + interface + value` 程序段 | 方案 C 无法复用任务协议 |
| 固定 `AxisSnapshot.position/velocity` | 保留兼容字段；新增带来源/单位/有效性的接口值 | 反馈改接 STM32/外部传感器时必须改 schema |
| `AxisConfig` 同时表示显示单位和最终限制 | V4 校验优先使用协商接口描述，旧配置只作额外主机防护 | raw counts、mm、rad 和控制器限制混在一起 |
| simulator-only 限制写进 `ControllerCapabilities` | 移到独立 `ControllerEndpointPolicy` | 中立契约无法描述未来端点 |
| 精确等于协议 3.0 | V4 默认 4.0，仍解码 3.0 | 无法协商与演进 |
| ACK 等于执行结果 | 增加独立命令生命周期和查询 | 超时、重连后无法判断命令是否执行 |
| 通信丢失时速度置零 | 保留最后值并标记 stale | 可能把未知运动错误显示为已停止 |
| PC “急停/复位急停” | 不再公布或在 GUI 使用；改为非安全保护性停止请求 | 会把普通网络命令误解为安全功能 |

## 3. 从成熟项目吸收的经验

V4 只吸收边界与语义，不引入 ROS、LinuxCNC 或其运行时。

### ros2_control

- [Hardware interface types](https://control.ros.org/kilted/doc/ros2_control/hardware_interface/doc/hardware_interface_types_userdoc.html) 把只读 state interface 与可声明占用的 command interface 分开，并允许 position/velocity/effort 之外的接口名。
- [Hardware component lifecycle](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html) 区分“已配置、可读但不可命令”和“已激活、命令可用”，并显式处理 read/write 错误。
- [Mock components](https://control.ros.org/master/doc/ros2_control/hardware_interface/doc/mock_components_userdoc.html) 在同一硬件边界替换真实部件，而不是让业务逻辑调用模拟器私有方法。

对应 V4 决定：状态/命令接口分离；能力驱动 GUI；连接、生命周期、轴状态分开；进程内和跨进程 SIL 共用适配器契约。

### LinuxCNC

- [HAL introduction](https://www.linuxcnc.org/docs/html/hal/intro.html) 使用带类型的 pin/signal 连接可替换组件，并把机器连线放在配置中。
- [Motion component](https://linuxcnc.org/docs/stable/html/hal/components.html) 将实时运动执行和上层命令/状态监督分开。

对应 V4 决定：Windows 只发监督意图和记录结果；确定性循环永不移到 PC；方案差异由接口与职责档案表达，不在 GUI 中写 `if scheme_a`。

### CANopenNode

- [NMT and heartbeat](https://canopennode.github.io/CANopenNode/group__CO__NMT__Heartbeat.html) 区分 initializing、pre-operational、operational、stopped。
- [Heartbeat consumer](https://canopennode.github.io/CANopenNode/group__CO__HBconsumer.html) 区分 unconfigured、尚未收到首帧的 unknown、active 与 timeout，并可识别远端重启。
- [PDO](https://canopennode.github.io/CANopenNode/group__CO__PDO.html) 和 SYNC/EMCY 具有各自的时序和错误语义。

对应 V4 决定：未来 STM32 报告中不得把 NMT、heartbeat、SYNC、总线、驱动故障和轴任务状态压成一个布尔量；V4 协议已保留独立诊断扩展位置和 controller boot ID。

### ros2_canopen

- [Master driver](https://ros-industrial.github.io/ros2_canopen/manual/humble/user-guide/master.html) 把 CAN 接口和事件循环归属 master。
- [Managed service interface](https://ros-industrial.github.io/ros2_canopen/manual/iron/user-guide/operation/managed-service-interface.html) 要求 master 先配置/激活，再按序激活驱动。
- [Operation modes](https://ros-industrial.github.io/ros2_canopen/manual/rolling/user-guide/operation.html) 区分简单监督、可恢复生命周期管理和低延迟控制用途。

对应 V4 决定：CANopen master、NMT、PDO/SDO 和驱动启动顺序属于 STM32 内部实现；Windows 只看聚合能力、就绪状态和诊断，不把对象字典读写暴露成通用 GUI 命令。

## 4. 冻结的职责边界

### Windows 上位机

- 项目、实验、会话、任务编辑、能力展示和人工确认；
- 生成带 command ID/sequence/connection ID 的监督命令；
- 按协商能力做格式、范围和用户策略预检查；
- 查询命令生命周期，显示状态/诊断/数据有效性；
- 保存命令、ACK、状态、事件、能力快照及与 V1～V3 数据的哈希关联；
- 不承担确定性轨迹插补、轴闭环、总线周期、物理互锁或安全功能。

### STM32 实时控制器

- 协议握手、能力声明、controller boot ID 和命令去重/对账；
- 三轴任务队列、轨迹、同步、状态聚合和确定性调度；
- CANopen master、节点生命周期、heartbeat/SYNC/EMCY/驱动诊断；
- 控制器最终限位、命令门禁、保护性停止处理和故障收尾；
- 方案 A 中向驱动器提交位置轨迹；方案 B 中可读取反馈并闭合位置环；方案 C 中由声明的接口决定，不改变 PC 协议。

### 驱动器

- 功率级、电机换相、驱动器状态和自身保护；
- 方案 A 中完成单轴位置闭环；
- 方案 B/C 中按最终能力选择位置、速度或转矩/电流模式；
- 实际 CiA 402 模式、对象字典和 PDO 映射尚未确定，V4 不做假设性实现。

### 反馈装置

- 来源可以是驱动器、STM32 直接输入、外部传感器或融合值；
- 每个状态值必须报告 source、unit、validity 和控制器侧采样时间；
- 上位机不得把旧值、推算值或换算值冒充新鲜的设备反馈。

### 安全系统

- 急停、STO、制动和硬限位的物理权限属于独立硬件/驱动器/实时控制器安全链；
- Windows `request_protective_stop` 只是普通通信请求；
- 物理急停/STO 状态未知时显示 unknown，不推断；
- V4 软件和 SIL 不能作为安全认证或现场放行证据。

## 5. V4 逻辑契约

### 能力握手

控制器返回：

- 支持的协议范围、controller/adapter/boot ID、transport kind；
- X/Y/Z 拓扑；
- 每轴 state/command interface 的名称、单位、范围和 rate；
- loop owner、feedback source、drive command mode；
- 名义状态率、控制器周期配置、命令对账支持和限制；
- simulated、real hardware access、device write capability 的事实声明。

V4 发布策略仅允许：

```text
simulated = true
real_hardware_access = false
device_write_capability = false
transport = in_process | loopback_tcp
```

### 命令生命周期

```text
submitted -> accepted/queued/running
          -> completed | cancelled | failed
submitted -> rejected
通信或重启后无法判定 -> unknown
```

ACK 只回答是否接收。运动是否完成由独立 status/snapshot/event 决定。

### 状态有效性

```text
valid   当前有效
stale   最后已知值，已过期
unknown 没有足够证据
invalid 控制器明确判无效
```

通信丢失时保留最后数值并改为 stale；不得把速度写成 0。

### 传输

本机跨进程 SIL 使用：

- 4-byte big-endian 长度 + UTF-8 JSON；
- 最大帧 4 MiB；
- HELLO/CAPABILITIES；
- COMMAND/ACK/STATUS；
- POLL/SNAPSHOT；
- EVENT_REQUEST/EVENT_BATCH；
- DISCONNECT/PROTOCOL_ERROR。

V4 没有真实 STM32 网络端点。`loopback_tcp` 只接受数字回环地址，helper 进程不包含 CAN 或硬件访问。

## 6. 方案切换影响

| 变化 | 不需要改 | 需要改 |
|---|---|---|
| A：驱动器位置环 | GUI、服务、记录、协议 | STM32/驱动映射和现场配置 |
| B：STM32 位置环、直接反馈 | GUI、服务、记录、协议 | STM32 控制算法、反馈驱动、驱动命令模式 |
| C：速度命令 | 协商、通用 setpoint/program、记录 | 对应任务编辑器呈现、STM32/驱动实现 |
| C：转矩/自定义接口 | 协议和记录结构 | 新接口描述、有限的专用参数编辑器、真实实现 |
| CANopen 改其他总线 | Windows 控制语义 | STM32 总线适配、能力和诊断映射 |
| 反馈来源变化 | Windows 状态容器 | STM32 采集/融合与 provenance |

V4 不建设通用插件市场。只有实际出现新接口且用户需要编辑其专有参数时，才增量增加小型编辑器。

## 7. 尚不能冻结的现场事项

- STM32 与 PC 的最终物理传输、认证、重放保护和网络部署；
- STM32 实际控制周期、遥测率、缓冲和时钟同步方法；
- 三个驱动器的对象字典、CiA 402 模式和 PDO 映射；
- 电机/反馈实际接法、单位、分辨率和方向；
- 软/硬限位、制动、STO、急停、安全继电器和故障复位权限；
- 方案 A 的驱动位置模式是否满足同步与轨迹要求；
- 真实命令幂等、断连后动作和控制器重启恢复策略。

这些事项需要真实硬件和安全方案证据；V4 不伪造答案。
