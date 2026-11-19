# Motion Acquisition V5.0 架构复审与控制器基础

## 1. 本版结论

V5.0 不宣称“STM32 + CANopen 三轴控制已完成”。本版把 V4 已建立的上位机中立契约第一次落到独立实现的控制器侧 C 核心和原生进程边界，从而把未来固件接入需要重写的范围压缩到：

- STM32H743 的板级时间、RTOS/HAL 和通信端口；
- CANopen master 与三驱动器状态监督；
- 经现场确认的驱动器模式、对象字典/PDO 和反馈配置；
- 独立的物理安全链。

GUI、`ControllerService`、能力协商、控制记录、采集、回放和分析不需要因方案 A/B/C 改写。

## 2. 真实证据对范围的约束

当前现场主口径仍只有 Node 1、1 Mbps、PC 侧 SDO upload 只读链路。STM32 master、三节点、PDO/SYNC、驱动模式、反馈接法、对象映射和安全放行均未验证。

本地归档的 `h750_demo` 与 `fdcan_demo` 不能作为当前固件基线：

- MCU 是 H750，而当前候选为 H743；
- 示例 node ID 分别为 `0x21` 和 24，与当前驱动 Node 1 口径不同；
- 一个为 normal mode，一个为 external loopback；
- 没有三驱动器、CiA 402 策略、轨迹队列、V4 协议和安全监督；
- `fdcan_demo` 还含 SDO download helper。

官方 DINGS 手册证明产品系列描述 PPM/PVM/CSP/CSV/CST 等模式及常见 CiA 402 对象，但不能证明已安装固件、反馈配置和 PDO map。特别是 0x6064 在传感器未配置时可能为 0，因此数值 0 不能自动解释为“轴静止且反馈有效”。

## 3. V3/V4 审校：保留、修正和未来替换面

| 现有设计 | V5 决定 | 原因 |
|---|---|---|
| GUI → worker → service → adapter | 保留 | 监督层与实时层边界正确 |
| capability/responsibility/interface | 保留 | A/B/C 通过数据表达，不在 GUI 分支 |
| 4-byte length + protocol 4.0 JSON | 保留 | 已有兼容记录和适配器；V5 无需破坏性协议升级 |
| ACK 与 command status 分离 | 保留并在 C 核心实现 | 真实通信必须区分“收到”和“完成” |
| boot/connection ID、reconciliation | 保留并实测跨语言 | 支撑断线、重启和重复命令 |
| 16 列原始 CSV 和既有 manifest | 完全保留 | 控制器演进不能污染采集权威数据 |
| Python 模拟器 | 保留为回归 oracle | 仍适合故障注入和 A/B/C 行为对照 |
| GUI 见 position 就发 move_absolute | 修正 | 这是隐藏的方案 A/legacy verb 耦合；现优先使用控制器声明的 generic setpoint |
| Python-only controller process | 新增 C 原生边界 | 旧进程没有验证非 Python 固件侧状态/协议实现 |
| 固定 mm/500 Hz SIL 参数 | 原生 fixture 改为 count/100 Hz 并明确 synthetic | 未标定单位和硬件周期不能伪装成事实 |
| PC “急停”语义 | 继续禁止 | PC 只能请求非安全保护停；E-stop/STO 只读且可为 unknown |

## 4. 四层职责

### Windows 上位机

- 连接、能力协商、任务/监督意图提交；
- 人机交互、采集、回放、分析和证据记录；
- 显示状态来源、有效性、命令终态和安全未知项；
- 不承担确定性循环，不直接暴露 SDO/PDO/NMT/CAN。

### STM32 实时控制器（未来）

- 绝对周期调度、三轴任务、轨迹、同步和状态管理；
- 驱动器/反馈健康监督和能力发布；
- 通过 DrivePort 将语义意图映射到已审定的驱动 profile；
- 保存命令生命周期与重连协调状态；
- 不把 Windows 当作物理安全控制器。

### 驱动器

- 在方案 A 中闭合单轴位置/速度/电流内部环；
- 公开经验证的状态、反馈和故障；
- 接收什么模式/对象/PDO 必须由真实硬件发现与评审决定。

### 反馈与安全装置

- 反馈来源可为 drive/controller_direct/external/fused，由 capability 声明；
- `valid/stale/unknown/invalid` 与数值分离；
- 物理 E-stop、STO、制动和 watchdog 独立于 Windows protective stop。

## 5. 新控制器基础

```text
ControllerService / RemoteSilControllerAdapter
        │ protocol 4.0 framed JSON
        ▼
Windows native reference process
        │ fixed-shape protocol mapping
        ▼
portable C controller core
        │ semantic DrivePort
        ▼
virtual drive fixture
```

纯 C 核心只使用定长结构：

- 三轴 snapshot；
- 32 条命令协调历史；
- 64 条有界事件队列；
- monotonic tick、cycle、deadline miss、failed read；
- `accepted/running/completed/cancelled/failed/rejected/unknown`；
- duplicate command ID 幂等协调；
- connection ID 防止旧连接命令生效。

Windows 端口只绑定 `127.0.0.1`，实现分片接收、4 MiB 帧上限、JSON/版本错误响应和独立 100 Hz fixture tick。它是未来固件边界的主机可测参考，不是固件或硬件仿真精度证明。

## 6. 开源经验的落地方式

- CANopenNode 将硬件 driver 作为外部 port，并把 time-critical SYNC/PDO 与普通 SDO/NMT/heartbeat 处理分开；V5 对应为纯 core、DrivePort、未来实时/普通任务分层。[CANopenNode driver](https://canopennode.github.io/CANopenNode/group__CO__driver.html)、[CANopen processing](https://canopennode.github.io/CANopenNode/group__CO__CANopen.html)
- CANopenNode SDO client 同时提供 upload/download，因此本项目不把原始 SDO client 暴露给 PC；未来 download 只能存在于经评审的 drive profile。[SDO client](https://canopennode.github.io/CANopenNode/group__CO__SDOclient.html)
- STM32CubeH7 采用芯片/板级项目和版本化组件，H750 工程不能通过改名变成 H743 port。[STM32CubeH7](https://github.com/STMicroelectronics/STM32CubeH7)
- FreeRTOS 的 `vTaskDelayUntil` 提供绝对周期语义，适合未来周期任务；V5 只实现 deadline 计数，暂不选择 RTOS tick。[vTaskDelayUntil](https://www.freertos.org/Documentation/02-Kernel/04-API-references/02-Task-control/02-vTaskDelayUntil)

这些经验用于约束边界，不引入 ROS、LinuxCNC 或大型插件系统。

## 7. 方案切换成本

| 变化 | 应替换 | 应保持 |
|---|---|---|
| A → STM32 位置环 | DrivePort、capability profile、controller algorithm | GUI、service、protocol、record |
| drive feedback → STM32 direct | feedback adapter、source/validity capability | state interface/render/record |
| position → velocity/effort | drive profile、command interface descriptor | generic setpoint、program、status |
| CANopen → 其他总线 | bus/drive port | controller core and PC semantic contract |
| 新方案 C | bounded profile + port implementation | acquisition/replay/research/history |

V5 不为未知方案构造通用插件市场；稳定点只有能力、状态、命令、DrivePort 和记录语义。

## 8. 未关闭项

- H743 板卡、RTOS/HAL、FDCAN、收发器、引脚和定时器；
- 三个实际 Node-ID、身份、heartbeat/NMT、0x6502/0x6061；
- CiA 402 状态转换、对象/PDO、SYNC 周期；
- 反馈接法、工程单位、限位、回零和标定；
- E-stop/STO/制动/watchdog 的物理方案；
- 真实传输、运动、同步、精度、时序和第二台 Windows。

以上项目不能由 V5 SIL 结果替代。

