# Motion Acquisition V6.0 架构复审与 H743 固件基础

## 本版结论

V6.0 将 V5 的 portable C 三轴核心真正编入 `STM32H743IIT6` ARM
镜像，并补齐嵌入式调度、协议入口、三节点健康、反馈、安全和硬件端口边界。
它是可以交叉构建、可供调试器检查、面向未来上板的固件基础，不是已经可用于
三轴运动的板级固件。

当前镜像中的所有具体硬件端口均返回 `unavailable`：

```text
real_hardware_access = false
device_write_capability = false
board_mapping_available = false
```

因此，生成的 ELF/HEX/BIN 不得在证据门槛关闭前烧录，也不能证明 CANopen、
同步、安全、运动或精度。

## 本地证据重新校对

| 本地材料 | 可以复用的证据 | 不能升级为当前事实的部分 |
|---|---|---|
| `fxxkSTM32` H743 Cube 工程 | H743IIT6、CubeH7 1.12.1、官方 startup/linker 结构 | 只有时钟/GPIO，无 FDCAN、PC 通信、RTOS、watchdog 或控制器 |
| `fdcan_demo` | H743 FDCAN、定时器、RTT、CANopenNode 组件存在 | FDCAN 是 external loopback；含 SDO download；节点 24 不是当前驱动节点 |
| `h750_baremetal` | normal-mode FDCAN 和定时器组织可供比较 | MCU、内存和板卡均不是 H743IIT6 |
| `h750_demo` | CAN 接收、恒周期处理、普通 CANopen task 分层 | H750、相对延时、节点 0x21；不是三驱主站或 V5 协议实现 |
| 当前执行文档 | Node 1、1 Mbps、PC SDO upload 只读 | STM32 主站、三节点、PDO/SYNC、写控制和安全链均未放行 |
| 驱动器手册 | 对象族和 PPM/PVM/CSP/CSV/CST 概念存在 | 不能证明当前三台驱动的固件、模式、PDO、单位和写入策略 |
| 现场照片 | H743 芯片和疑似 CAN 相关器件可见 | 不能证明板卡型号、引脚、收发器型号、终端、电气参考或时钟 |

V6 没有复制毕业归档中的应用、对象字典或 SDO helper。官方 CMSIS 最小集合
来自 STM32CubeH7 v1.12.1 和其 H7 device 子模块的精确提交，并保留许可证。

## V3～V5 中继续保留的设计

- 上位机、控制服务和控制器 adapter 分离；
- capability negotiation，而不是由 GUI 猜测硬件能力；
- state interface 与 command interface 分离；
- ACK 与 `accepted/running/completed/failed/cancelled/unknown` 生命周期分离；
- boot ID、connection ID、command ID 和有界重复命令对账；
- raw counts 和现有数据/记录 schema 不因控制器接入改变；
- 设备细节只进入语义端口以下，不进入实验、记录、回放和分析层；
- PC 保护性停止始终标记为非安全功能。

## V5 中仍隐含的方案 A 假设及本版处理

V5 的 `DrivePort.read_state()` 同时返回驱动状态和位置/速度，名称上接近
“反馈来自驱动器”的方案 A。V6 没有破坏这个稳定核心接口，而是在运行时增加
独立 `FeedbackPort`：

- 方案 A：DrivePort 从驱动器状态/PDO 形成逻辑轴状态，FeedbackPort 可只做
  独立交叉诊断；
- 方案 B：板级 adapter 将 STM32 直读反馈与驱动器状态合成为同一个逻辑
  DrivePort state；
- 方案 C：替换板级反馈/驱动 adapter，核心和上位机协议不变。

这里采用的是小型 facade，不是庞大插件系统。真正需要按方案变化的代码被限制在：

```text
板级时钟与调度
PC byte transport
CANopen stack port
驱动器 profile / logical DrivePort facade
FeedbackPort
SafetyPort
```

## 职责边界

| 层级 | 当前和未来职责 | 不应承担的职责 |
|---|---|---|
| Windows 上位机 | 实验配置、能力协商、任务下发、状态展示、记录、回放、分析 | 硬实时闭环、物理急停/STO、逐周期总线调度 |
| STM32 实时控制器 | 三轴任务状态、轨迹/同步策略、命令幂等、周期调度、节点健康、硬件 watchdog 接口 | GUI、历史数据分析、伪造传感器有效性 |
| CANopen/驱动 profile | 节点身份、CiA 402/mode/PDO 映射、语义 intent 到已审核总线行为 | 向 PC 暴露任意 SDO/NMT/CAN 写入口 |
| 驱动器 | 方案 A 下的单轴内环、电机功率级、驱动器保护和反馈读入 | 三轴实验流程、Windows 数据管理 |
| 反馈装置 | 产生带来源、时间和有效性的测量 | 在无接法/标定证据时自动成为正式物理量 |
| 独立安全链 | E-stop、STO、制动和硬件 watchdog 的最终安全动作 | 由 Windows “停止”按钮代替 |

## 运行时结构

```text
SysTick 1 ms bring-up time base（仅软件配置，不是实测性能）
  ├─ fast receive hook：未来 CAN ISR/driver drain
  ├─ absolute-deadline realtime hook：未来 SYNC/PDO + mc_core_tick
  ├─ bounded mainline hook：heartbeat/NMT 状态、SDO discovery、PC framing
  └─ health hook：feedback/safety validity、诊断计数
```

每次 `step()` 最多运行每类到期任务一次；落后周期转换为 deadline-miss
计数，不执行无界追赶循环。帧流使用固定 8192 字节上限，支持分片、多帧、
零长度拒绝和超长帧有界丢弃。

## 协议与真实/模拟一致性

同一份嵌入式 decoder 处理 protocol 4.0：

- `hello` / capability negotiation；
- `poll` connect/status；
- `command` 与 command ID/sequence/expected connection；
- `status_request`；
- `event_request`；
- `disconnect`。

它直接调度同一 `mc_core_t`，因此重复命令、ACK、status history 和连接语义
不依赖 Windows reference process。当前 H743 没有被释放的 UART/USB/Ethernet，
所以 byte transport 和响应 JSON egress 均保持 unavailable；不能为完成度虚构
某个引脚或接口。

## 三节点 CANopen 管理

固件包含三个轴位的节点 inventory 和状态：

```text
unassigned → unavailable → observed → operational
                         ↘ degraded / fault
```

H743 目标的三个 Node-ID 当前均为 0（unassigned）。`1/2/3` 只在主机测试 fixture
中验证 manager 行为，不是现场三节点配置。CANopen stack port 只有 fast、RT、
mainline 和被动 node observation hook，不包含 raw transmit、SDO download、
NMT write、PDO configuration、controlword 或 mode write API。

## 从成熟项目吸收的经验

- [ros2_control](https://control.ros.org/master/doc/ros2_control/doc/index.html)
  将硬件组件的生命周期、state interfaces 和 command interfaces分离；V6
  采用能力描述和显式端口，但不引入 ROS/pluginlib。
- [LinuxCNC 架构说明](https://linuxcnc.org/docs/html/code/code-notes.html)
  将非实时 task、实时 motion、离散 IO 和 GUI 分层；V6 同样不让 Windows GUI
  进入实时循环，也不让普通通信状态冒充安全链。
- [CANopenNode](https://github.com/CANopenNode/CANopenNode)
  区分快速 CAN 接收、恒周期 SYNC/PDO 和普通 SDO/NMT/heartbeat；V6 保留这三类
  context，但未接入缺少板级证据的 FDCAN port。
- [ros2_canopen lifecycle manager](https://ros-industrial.github.io/ros2_canopen/api/iron/classros2__canopen_1_1LifecycleManager.html)
  说明 master 与 device lifecycle 需要配置、激活和恢复；V6 用显式节点状态和
  capability gate，避免“一连上 CAN 就认为可运动”。

这些项目是架构经验来源，不是本项目依赖，也不会被整套引入。

## 切换方案时的改动范围

| 变化 | 需要替换 | 不需要重写 |
|---|---|---|
| 驱动器位置闭环 → STM32 位置闭环 | trajectory/loop module、DrivePort facade、capability profile | GUI、协议 envelope、命令 ID/status、记录/回放 |
| 驱动反馈 → STM32 直读反馈 | FeedbackPort 和逻辑轴 state aggregator | task/session、实验信息、原始数据系统 |
| position → velocity/torque | drive profile、命令 interface capability、控制器算法 | transport、节点健康、上位机通用 interface UI |
| UART → USB/Ethernet | ByteStreamPort 与部署配置 | framing、decoder、core、CANopen/drive profile |
| CANopen → 其他总线 | bus/drive adapter | 上位机协议、task/core、数据和研究平台 |

## V6 已完成与未完成

已完成：

- 真实 H743 startup/linker/CMSIS 交叉构建；
- portable core 实际进入 ELF；
- 有界无 heap 的 runtime、framing 和 protocol decoder；
- 三节点健康 manager；
- PC/CANopen/feedback/safety 明确端口；
- debugger 可读持久诊断区和 fault handler；
- 主机 C 测试、Python 全量回归和跨语言协议验证；
- 一键固件构建、符号/segment/无写入审计和构建报告。

尚未完成且未声称：

- 板卡启动、时钟、cache/MPU、SysTick 精度和 watchdog；
- FDCAN、收发器、终端、电气参考和真实 CANopen；
- PC 物理传输和响应 egress；
- 三台驱动的身份、对象字典、模式、PDO 和任何写权限；
- 反馈接法、单位、方向、标定和有效性；
- E-stop/STO/制动/限位安全逻辑；
- 三轴同步、轨迹、运动、精度和鲁棒性；
- 第二台 Windows 电脑和真实硬件验证。
