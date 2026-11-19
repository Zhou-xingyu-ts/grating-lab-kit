# Motion Acquisition V9.0.0 离线用户指南

## 适用范围

本指南只适用于 V9 离线候选的源代码、SIL/Mock、回放和离线发布验证。执行任何测试、构建、打包或脚本前，仍须遵循工程 AGENTS.md：先说明命令、输入、预期输出与风险，并取得用户对该批命令的明确确认。

默认禁止打开或枚举 CAN、串口、USB CDC、ST-LINK VCP、网络端点、GM1、STM32、EtherCAT、驱动器或电机。本指南中的“连接”“预检”“运行”均仅指 Fake/Mock/SIL，除非另有现场书面授权。

## 1. 进入 Route Manager

启动桌面程序后，进入“路线管理”页面。页面应显示三张卡片：

| 卡片 | 应显示的定位 | 离线可执行动作 |
|---|---|---|
| Route A | 现场固定基线 | Fake/回放预检、历史会话兼容检查 |
| Route B | 主推、待上板 | 协议 Fake、Virtual DS-BVM、配置校验 |
| Route C | GM1/EtherCAT 候选、待工程验证 | Mock GM1、寄存器契约与 ESI 审计 |

初始模式必须为 Offline/SIL。真实连接、预备控制和运行按钮在未获得单独授权时保持不可用；不可用状态要显示原因，而非尝试探测硬件。

## 2. 路线选择与状态

1. 确认当前没有活动路线会话。
2. 选择一张路线卡，读取拓扑、主站所有者、配置完整性、证据门和能力快照。
3. 仅执行该路线的 Offline/SIL 预检。
4. 预检通过后，如需留存本次软件证据，显式点击“保存离线预检记录”。它只会在本机 `route_preflight_records/` 下写入一份空样本、16 列兼容的 manifest 记录；不会读取、连接或写入任何设备。
5. 若预检失败、端点配置缺失、命令对账不完整、关闭状态未知或资源锁被占用，则保持 unavailable、fault 或 unknown，不可切换或发命令。
6. 完成后使用路线专属 shutdown；关闭结果必须记入控制记录或验证证据。

“请求保护停机”只表示 PC 的普通停止请求，不等同物理急停或 STO。物理 E-stop/STO 只能显示来自控制器的已证实状态；没有来源时必须显示 unknown。

## 3. 三路线离线工作流

### Route A：固定直控基线的兼容回归

- 使用 Fake SDO、回放和正式历史会话的完整性检查。
- 检查固定 +5000/-5000 counts 计划、七对象采集、bit10/bit12 判据、共享 client 与 Halt→Disable 收尾语义。
- 确认独立直控窗口和 motion-acq-direct 入口仍独立存在。
- 不得在 V9 GUI 中因为“回归”而打开 PCAN 或连接 Node 1。

真实 Windows-PCAN 基线的重新运行属于后续现场工作，须使用 [Route A 回归检查卡](ROUTE_A_REGRESSION_CHECK_CARD.md) 并取得逐动作授权。

### Route B：STM32 CANopen 候选

- 先读取配置；若 COM 口、波特率、板端外设或权限级别不完整，路线应为 unavailable。
- 在 In-memory transport 上验证 protocol 4.0 分帧、partial write、ACK、boot ID、重启、断线和 unknown 处理。
- 在 Virtual DS-BVM 上验证只读/超时/abort/heartbeat/EMCY/PPM/PDO/SYNC 场景。
- 使用 locked 安全默认镜像；它不分配节点、不打开真实端口，也不允许写入。

首次烧录、串口、FDCAN、只读 CANopen 和后续控制都不属于本指南的执行范围，见 [Route B 首次实验入口](ROUTE_B_STM32_LAB_ENTRY.md)。

### Route C：GM1/EtherCAT 候选

- 使用 Mock GM1 验证 gm1_route_v1 的握手、字序、负数 raw counts、心跳、命令 sequence、ACK 和停止对账。
- 对 ESI/XML 原件运行只读审计，并检查报告是否列出 InitCmd、PDO、DC 与未闭合项。
- 缺少当前 GM1 工程、契约版本、build ID 或证实寄存器映射时，真实路线保持 unavailable。

不得通过 Windows 安装、启用或测试 EtherCAT master。真实 GM1/网络/EtherCAT 工作属于后续入口。

## 4. 会话和证据记录

离线控制记录或 manifest 的新增 route 元数据至少应保存：

- route ID、主机传输、fieldbus、master owner；
- adapter/capability、时间语义、权限状态与预检结果；
- real_hardware_access=false、device_write_capability=false；
- 路线专属关闭结果及 unknown/fault 原因；
- 若存在独立采集会话，其 SHA-256 关联关系。

不得修改 16 列 samples.csv、旧 manifest 的必填解释、历史控制记录或 checksum。Route A 保持原有控制—采集合并会话；B/C 只通过控制记录与独立采集会话哈希关联。

Route Manager 的“保存离线预检记录”是显式、用户触发的本机文件写入；路线列举、卡片创建、配置检查与预检本身均不写文件。该预检记录无采样值，不能用于位移、同步、控制或设备通过结论。

## 5. 配置检查

使用 [三路线配置说明](V9_0_ROUTE_CONFIGURATION.md) 做无 I/O 校验。下列任何一项缺失都应 fail-closed：

| 路线 | 必须保持或具备的条件 |
|---|---|
| A | 离线模式；真实 PCAN 入口禁用；固定基线仅作为证据描述 |
| B | 显式 transport 配置；安全默认 locked；未确认端口为 unavailable |
| C | 可验证的 magic/version/build ID/心跳；GM1 工程与寄存器契约缺失时 unavailable |

配置中出现历史 IP、历史寄存器、Node ID、bitrate 或引脚信息时，必须标记为“实验模板/待证实”，不能把它们当成自动生效的现场默认值。

## 6. 常见离线状态

| 状态 | 含义 | 用户应做什么 |
|---|---|---|
| unavailable | 配置、设备契约或授权不完整 | 补齐离线配置或证据，不连接设备 |
| disconnected | 路线可被创建但未连接 | 只可选择 Fake/Mock 端点 |
| connected_readonly | 只读或仿真链已连接 | 不把它解释为设备写权限 |
| preflight_passed | 对当前离线配置的检查通过 | 记录结果，仍不等于现场放行 |
| armed/running | 仅在明确授权的相应环境才可出现 | 离线阶段通常不应对应真实硬件 |
| stopping/closed | 路线专属关闭正在进行/已确认 | 记录关闭证据后才允许切换 |
| fault/unknown | 状态或命令结果不可确认 | 停止后续动作，不自动重试运动命令 |

## 7. 离线完成前的退出条件

关闭 V9 离线工作前，确认：

- 没有任何真实端口被打开、枚举或扫描；
- 没有烧录、复位、NMT、SDO 下载、PDO、SYNC、Modbus 写或运动；
- 原始数据和历史发布物无字节变化；
- 测试/构建/打包结果已填入离线验证报告；
- 缺陷、未知状态和现场证据缺口已记入缺陷表；
- 最终文字只使用 READY_FOR_LAB_TEST，且附带全部现场待确认项。
