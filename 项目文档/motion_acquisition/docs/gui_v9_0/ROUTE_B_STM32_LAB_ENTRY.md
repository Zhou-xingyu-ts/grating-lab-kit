# Route B 首次实验入口：Windows → STM32H743 → CANopen → DS-BVM

## 本入口不是授权

本文件定义 V9 离线候选完成后，Route B 未来进入现场的分段门。它不授权连接、烧录、复位、打开串口、CAN 通讯、NMT、SDO、PDO、SYNC、CiA 402、Servo On 或运动。

Route B 的控制责任固定为：Windows 发送 protocol 4.0 高层任务，STM32H743 是唯一 CANopen 主站。PC 不得绕过 STM32 直接成为该路线的 CANopen 写主站。共享底座同一时刻只允许一个运动主站。

## 已落地代码（2026-08-17，纯离线、未上电）

本轮已在源码层落地「Route B 静止只读采集」链路代码，未做任何硬件动作：

- 固件：`MotionControllerRouteBReadonlyFirmwareH743`（`main_routeb_real.c` + `usart1_port`/`fdcan_port` + `route_b_readonly_sampler` + 只读镜像），CANopen Node 1 主站，仅 SDO upload 读 `0x1000/0x1018/0x6041/0x6061/0x6064`，~5 Hz，Servo 保持 Off，不写、不 NMT、不 PDO/SYNC、不动。
- 上位机：`RouteBReadOnlySource`（首个 `TimestampedReadOnlySource`）+ Route Manager「Route B 静止只读采集」按钮，收 50 个样本 → 150 行 16 列 `samples.csv`。
- 离线验证：`test_route_b_v9.py`/`test_route_manager_workspace_v9.py`/`test_no_write_api.py` 34 passed；只读目标一次 Arm GNU 构建成功（`-Werror` 零告警）。证据 `build/v9_readonly_armgnu_20260817/`。

**未闭合**：未烧录、未开 COM、未接 CAN 收发器/总线、未上电、未真实采集。Gate 0～4 的现场证据与逐项授权仍未开始；下方各 Gate 表格仍为空，不得以 Mock/SIL 输出代填。

**2026-09-18 板级提醒（公开资料核实）**：阿波罗 H743 底板**自带 TJA1050 CAN 收发器**，而 PA11/PA12 在板上是 USB/CAN 复用、**出厂跳线默认给 USB**。因此：

- 要走板载收发器，必须先把底板「CAN/USB 选择接口」的跳线帽改到 `CRX`/`CTX` 侧。**跳线不动，FDCAN 的发送不达任何收发器**，现场会表现为完全无报文，容易误判成固件或波特率问题。
- 板载 TJA1050 与外接 VP230 模块（现场已见，带 `121` 终端电阻）**只能二选一**。两者同时接在 PA11/PA12 上会互相驱动。
- 板载收发器的 120 Ω 终端在板上；外接模块的终端在模块上。无论走哪条，断电测 CANH-CANL 仍按父工作区 `57` §7.3：约 60 Ω = 两端都在，约 120 Ω = 只有一端。
- Gate 3 表格需补记「跳线位置」与「走板载还是外接」两项后再上电。

## Gate 0：离线候选冻结

在任何板卡或线缆操作前，必须取得：

- [ ] V9 离线验证报告的实际测试、构建、哈希和发布候选证据；
- [ ] 结论为 READY_FOR_LAB_TEST，且不包含虚假的硬件通过措辞；
- [ ] safe image 配置证明：authority=locked、Node ID 未分配、bitrate 未分配、真实 ByteStreamPort/FDCANPort/FeedbackPort/SafetyPort 均 unavailable；
- [ ] GNU 与 Keil 使用同一业务源码的实际构建、符号/内存、哈希和写权限审计；
- [ ] protocol 4.0、Virtual DS-BVM、断线/重启/重复/陈旧 ACK 与未知状态处理的离线证据；
- [ ] 父工作区上电前放行、验收和未闭合项的适用审阅记录。

缺任一项时，不进入 Gate 1。

## Gate 1：精确板型和首次烧录门

目标仅是验证“安全默认镜像可被受控下载和观察”，不启用任何现场端口。

必须先记录：

| 项目 | 现场待确认内容 |
|---|---|
| 板卡 | STM32H743 板/底板精确型号、修订、供电、BOOT/RESET、晶振、存储与调试电压 |
| 调试 | ST-LINK/SWD/JTAG 连接方式、参考电压、独立供电策略、复位策略 |
| 固件 | 选定的已哈希 safe image、工具链、下载日志和读回/标识策略 |
| 隔离 | 电机、驱动、CAN、EtherCAT 和不必要外设处于断开或经独立安全确认的状态 |
| 授权 | 对本次烧录/复位操作的明确单独批准 |

Gate 1 的允许观察项只能是 MCU 身份、复位、build ID、boot counter、时基、故障记录和不可用端口状态。不得将“可下载/可调试”称为 FDCAN、CANopen、驱动器或运动通过。

停止条件：电源、板型、调试电压、镜像哈希、端口不可用状态或复位行为任一不明，即停止并记录。

## Gate 2：串口/PC transport 门

目标是经批准验证 Windows—STM32 的单一字节传输边界，不接入驱动器控制链。

前置：

- [ ] 明确选定 USB CDC、ST-LINK VCP 或 UART，且 COM 口、波特率、电平、隔离、线序和板端外设均有证据；
- [ ] 每次打开端口已获单独授权；禁止程序自动扫描或猜测端口；
- [ ] protocol 4.0 的 HELLO、capability、boot ID、sequence、ACK、最终状态与断线恢复预期已写入测试计划；
- [ ] 不确认的命令、重启后的命令和超时命令都保持 unknown，禁止自动重发。

允许的现场问题仅限“受控 transport 是否能按契约传递和对账”。它不能证明 CANopen、FDCAN、驱动器或安全链。

停止条件：身份/boot ID 不符、帧对账失败、意外可写权限、端口自动打开或未知状态。

## Gate 3：FDCAN 电气和端口门

目标是确认 STM32 与 CAN 物理层的前置事实，而不是开始驱动控制。

必须先确认：

- [ ] FDCAN 外设实例、引脚复用、时钟、收发器型号/供电/使能、CANH/CANL/GND、终端和总线电气；
- [ ] MCU FDCAN 逻辑脚绝不直接连接 CANH/CANL；
- [ ] 节点、设备 identity、总线拓扑、其他主站排他和分析/测量方法；
- [ ] 对每一项连接与通信操作的单独授权。

在上述证据完成前，FDCANPort 必须保持 unavailable。仅观察到引脚或收发器也不等于 CANopen 已工作。

停止条件：任何线序、终端、电压、收发器、主站排他或身份事实不明。

## Gate 4：只读 CANopen 门

只有 Gate 0～3 全部通过且获得新的明确授权后，才可以规划只读验证。此门的范围应严格限制为：

- 不发送 NMT、SDO 下载、PDO 配置、SYNC、控制字、模式写入、Servo On 或运动；
- 只读取经确认节点的身份与允许的 SDO upload 对象；
- 当前可作为待证实读取候选的对象包括 0x6041、0x6061、0x6064，实际对象、Node ID 和 bitrate 以现场证据为准；
- 对 heartbeat、abort、EMCY、超时和身份不匹配保持记录，不把无响应解释为成功。

读取成功只能证明该一次受控只读观测，不自动放行控制。

## Gate 5：后续控制门

本门不属于 V9 离线候选。只有所有前置证据、共享底座安全放行、设备 identity、对象能力、PDO/SYNC/DC、反馈、STO/E-stop、制动、限位、机械约束、人员和逐动作风险评审都完整后，才可另行提出控制方案。

后续每个动作必须单独授权。写权限从 locked 到 read_only 或 control_armed 不能由软件、重连或 UI 自动升级。

## 现场证据记录模板

| Gate | 日期/执行人 | 授权编号 | 输入配置/哈希 | 观察结果 | 异常与停止 | 结论 |
|---|---|---|---|---|---|---|
| 0 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| 1 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| 2 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| 3 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| 4 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| 5 | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |

离线候选阶段这张表应保持未填写的现场结果；不得以 Mock/SIL 输出代填。
