# Motion Acquisition V7.0 总体架构复审

## 结论

V7.0 继续以方案 A 作为第一个可执行方向：

```text
Windows 上位机
  负责配置、任务意图、监视、记录、回放与人工决策
        │ protocol 4.0：能力、状态、命令、确认、事件
        ▼
STM32H743 实时控制器
  负责任务生命周期、三轴时序、同步、通信监督与故障收口
        │ CANopen：节点生命周期、状态/命令接口、未来 PDO/SYNC
        ▼
三个驱动器
  当前优先承担单轴位置闭环；具体对象与模式必须由现场配置证明
        │
        ▼
每轴一套权威位置反馈
```

方案 A 不是上位机协议或控制核心的永久假设。闭环所有者、反馈来源和
驱动命令模式均通过 `responsibility` 能力描述：

- 方案 A：`loop_owner=drive`、`feedback_source=drive`、
  `drive_command_mode=position`；
- 方案 B 示例：`loop_owner=realtime_controller`、
  `feedback_source=controller_direct`、命令可为 velocity/effort；
- 方案 C：仍复用相同能力、状态、命令确认和数据记录语义，只替换
  STM32 下方的反馈/驱动/总线端口与责任配置。

V7 没有建设通用插件市场。可替换点只有已有的 typed ports、责任配置和
未来经验证的 drive profile。

## V3～V6 设计复审

| 既有设计 | 结论 | V7 处理 |
|---|---|---|
| Windows 与控制器协议分离 | 保留 | protocol 4.0 不变，补齐嵌入式响应编码与部分写入 |
| 状态接口与命令接口分离 | 保留 | 能力协商继续分别列出 state/command interfaces |
| raw counts 权威数据 | 保留 | 不引入未经标定的物理单位，不改 16 列采集 CSV |
| semantic `DrivePort` | 保留 | 核心只提交 enable/stop/setpoint 等意图，不依赖 CANopen 对象 |
| 三轴固定容量核心 | 保留 | X/Y/Z 为产品轴语义；实际 Node-ID 仍为运行配置 |
| 绝对时限裸机调度 | 保留 | 增加生命周期、liveness 与看门狗诊断 |
| 模拟控制器与真实接口一致 | 加强 | Python SIL、native C、embedded virtual byte stream 共用协议 4.0 |
| CANopen 三节点“健康外壳” | 不足 | V6 只能 observe；V7 增加真实标准帧状态机和虚拟主站发送 |
| H743 全部硬件端口 unavailable | 部分保留 | 生产目标仍关闭；虚拟环境增加显式授权发送，不再是零发送空壳 |
| V6 protocol dispatcher | 不足 | 原来只生成 C response struct；V7 已能编码、排队、处理 partial write |
| 模拟能力写死方案 A | 风险 | 责任配置变为运行时数据，B/C 能通过同一协议编码 |
| 安全状态 | 语义保留、物理延期 | protective stop 与 E-stop/STO 分离；后两者继续 unknown |

## 职责边界

### Windows

- 允许：实验配置、任务/程序意图、能力协商、状态和事件显示、命令确认、
  采集/回放/分析、人工停止请求、证据记录。
- 不负责：确定性插补、每周期控制、CAN ISR、硬件看门狗、安全认证动作。
- 不能把通信断开、窗口关闭或 protective-stop 请求描述为 E-stop/STO。

### STM32

- 负责：控制器连接/任务生命周期、绝对时限周期、三轴一致状态、
  CANopen 节点监督、命令去重/确认/对账、故障收口、诊断发布。
- 未来方案 A：轨迹、同步和状态管理，驱动器承担单轴位置闭环。
- 未来方案 B：可在 `DrivePort` 上方增加控制律，在 `FeedbackPort` 读取
  单一反馈源；PC 协议和记录格式不变。
- 不负责：在缺少接线/极性/安全分析时声称安全停机；未经验证自动写入
  驱动器对象。

### 驱动器

- 当前首选职责：电流/速度/位置内部环与功率输出。
- V7 只掌握系列手册层面的 CiA 301/402 语义；实际型号、固件、0x6502、
  PDO 映射和模式必须现场读回确认。
- 工作区没有 DS-BVM CANopen EDS。归档 `DS301_profile.eds` 属于旧
  CANopenNode 控制器示例，不是驱动器 EDS。

### 反馈与安全

- 每轴只定义一个权威反馈源；方案变化可以改变读取位置，但不自动形成
  双反馈融合。
- raw counts 是软件证据边界；scale/offset 仍是可选未验证显示。
- E-stop、STO、制动器、限位、看门狗输出必须由物理硬件完成并单独验收。
  V7 的 safety port 与软件 liveness 不是安全认证功能。

## 运行时选择

V7 保持裸机：

- 当前 H743 基线无 RTOS，V6 已有有界单写者主循环和绝对时限；
- 本地唯一 FreeRTOS 示例是 H750VBTx，使用动态任务/newlib 和不同 HSE；
- 引入它不能解决当前 H743 板卡、FDCAN、PC 传输或安全接线缺失。

未来若板级需求需要 FreeRTOS，只应让任务调度调用相同 runtime/ports；
不得改变 protocol 4.0、`DrivePort` 或会话记录语义。

## 从成熟项目吸收的最小经验

- [ros2_control](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html)：
  生命周期化硬件资源，state/command interface 分离，控制周期明确
  read/update/write。V7 采用语义，不采用 ROS/URDF/pluginlib。
- [LinuxCNC HAL](https://linuxcnc.org/docs/html/man/man3/intro.3hal.html)：
  实时和非实时职责分离，接口信号可观察。V7 对应 Windows/STM32 分界
  和 typed ports，不复制 HAL。
- [CANopenNode](https://github.com/CANopenNode/CANopenNode)：
  快速 CAN 接收、固定周期实时处理、mainline 耗时处理和目标驱动分离。
  V7 保留三类工作和非阻塞有界队列。
- [ros2_canopen master](https://ros-industrial.github.io/ros2_canopen/manual/humble/user-guide/master.html)：
  主站事件循环与 NMT/SDO 服务分离。V7 将 read SDO 与 NMT/control
  发送权限拆成两个门。

## V7 仍未证明

- H743 板卡型号、时钟、缓存、FDCAN 实例/引脚/收发器/终端；
- PC 到 STM32 的 UART/USB/Ethernet 物理传输；
- 三个真实 Node-ID、驱动器 identity/固件、EDS、PDO/SYNC 和模式；
- 反馈接法、单位、方向、延时和时间同步；
- E-stop/STO/制动/限位/硬件看门狗的接线与失效行为；
- 实际周期抖动、同步、运动、精度、鲁棒性和第二台 Windows 部署。

