# 阶段 A 受控资料清单

> 审计日期：2026-07-13  
> 审计方式：只读静态审计；未运行测试、脚本、数据分析程序或硬件操作。  
> 根路径：`D:\清华\清华-科研横向\光栅尺运动控制调研`

## 1. 证据标签

| 标签 | 本轮含义 |
|---|---|
| E1 | 当前台架原始数据、对应 manifest、实验记录和现场状态记录 |
| E2-C | 当前 `motion_acquisition` 源码或配置中实际存在的实现 |
| E2-T | 当前测试代码覆盖的行为；本轮没有运行测试 |
| E2-R | 既有发布说明或测试运行报告中记录的历史结果；本轮没有复跑 |
| H | 研究状态声明、说明性文档、示例、推断或尚未确认事项；不能单独作为台架事实 |

## 2. 当前入口与实验记录

| 来源 ID | 标签 | 完整文件路径 | 日期/版本 | 定位 | 用途 | 限制 |
|---|---|---|---|---|---|---|
| SRC-A001 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\README.md` | 2026-07-13 工作模式 | 第 5-11、15-43 行 | 当前离线研究入口与目录职责 | 是工作约束，不证明台架性能 |
| SRC-A002 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\README.md` | 2026-07-13 | 第 5-43、62-92 行 | 当前研究边界、代理平台定位 | 不证明自研光栅尺或三轴能力 |
| SRC-A003 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\research-state.yaml` | 2026-07-13 | `verified_proxy_basis`、`evidence_boundary` | 机器可读研究状态 | 内容来自已有记录的汇总，不是独立证据 |
| SRC-A004 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\00_总览.md` | 更新至 2026-07-13 追加事实 | 第 3、38、92-97 行 | 当前执行事实入口与最新结果摘要 | 摘要必须回查实验记录和数据文件 |
| SRC-A005 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\00_索引.md` | 2026-07-12 | 第 1-47 行 | 确认实验记录职责和时间顺序 | 索引不替代实验正文 |
| SRC-A006 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-18_光栅尺单反馈闭环_Scope数据采集_10Hz限制确认.md` | 2026-06-18，6 月 23 日补记 | 第 5、22、55-64 行 | 当前光栅尺单反馈接法和 Scope 约 10 Hz 边界 | 10 Hz 数据只适合趋势与通断判断 |
| SRC-A007 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-22_CANopen链路_USB-CAN实测_scope-can622数据分析.md` | 2026-06-22，6 月 23 日补记 | 第 42-50、60-89 行 | USB-CAN 背景、Scope 约 10 Hz 和旧 node 622 语境 | 旧 `622` 不是 7 月确认的 CANopen Node-ID |
| SRC-A008 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 2026-07-12，2026-07-13 追加第 14 节 | 第 1-203 行 | USB/COM 与 USB-CAN 分工、50/100/200 Hz、方向差异、参数敏感性和暂定换算 | 指标是记录中的既有分析；本轮未重新计算 |
| SRC-A009 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\README.md` | 2026-06-23 | 第 8-48 行 | 两份 Scope CSV 的列定义和用途边界 | 文档中的峰值和采样率未在本轮复算 |
| SRC-A010 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\scope_usb_2026-06-18.csv` | 2026-06-18 | 表头、首行、末行 | USB 直连 Scope 原始趋势数据 | 本轮只核对结构和对应关系 |
| SRC-A011 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\scope_can622_2026-06-22.csv` | 2026-06-22 | 表头、首行、末行 | USB-CAN 下 Servo Studio Scope 原始趋势数据 | 文件名保留历史 `622`，不代表当前 Node-ID |

## 3. `motion_acquisition` 说明、架构、配置与既有报告

| 来源 ID | 标签 | 完整文件路径 | 日期/版本 | 定位 | 用途 | 限制 |
|---|---|---|---|---|---|---|
| SRC-A020 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\README.md` | v0.2.0 | 第 5-21、148-199、201-212 行 | 软件定位、CSV/manifest 格式和安全声明 | 自述需由源码、测试和 E1 交叉确认 |
| SRC-A021 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\architecture.md` | 2026-07-09 | 第 23-50 行 | 模块职责和设计边界 | 是设计说明，不等于运行证据 |
| SRC-A022 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\release_v0.2.0_readonly_station_foundation.md` | 2026-07-10，v0.2.0 | 第 14-35、68-127 行 | 既有 `31 passed`、Playwright 和静态检查报告 | 早于 7 月 12 日台架验证；本轮未复跑 |
| SRC-A023 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\roadmap.md` | 2026-07-09 | 第 3-49 行 | 区分已实现版本与未来 TPDO/应用路线 | Roadmap 项不是当前实现 |
| SRC-A024 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\examples\mock_manifest_v0.2.0_readonly_station_foundation.json` | 2026-07-10，schema 1.0 | 全文件，尤其 `notes` | manifest 示例字段 | 明确为 mock，不是台架数据 |
| SRC-A030 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\configs\canopen_readonly.example.yaml` | 2026-07-08 | 第 1-27 行 | 示例配置字段和默认单对象读取 | 示例值不能证明现场配置 |
| SRC-A031 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\configs\canopen_readonly.local.yaml` | 2026-07-12 | 第 1-24 行 | 现场使用的 PCAN、1 Mbps、Node 1 配置 | 配置存在不等于设备响应；需 E1 佐证 |
| SRC-A090 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\output\playwright\desktop_screenshot_log.txt` | 2026-07-08 | 全文件 | 历史桌面截图命令记录 | 只记录截图动作，不含布局断言结果 |
| SRC-A091 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\output\playwright\mobile_screenshot_log.txt` | 2026-07-08 | 全文件 | 历史移动端截图命令记录 | 只记录截图动作，不含布局断言结果 |

## 4. 当前源码（E2-C）

以下均位于 `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\`。`.egg-info`、`__pycache__` 和 `.pyc` 为生成物，未作为源码证据。

| 来源 ID | 文件 | 日期 | 主要定位 | 审计用途 | 限制 |
|---|---|---|---|---|---|
| SRC-A040 | `__init__.py` | 2026-07-10 | 第 1-3 行 | 软件版本 `0.2.0` | 只证明版本常量 |
| SRC-A041 | `acquisition.py` | 2026-07-09 | `read_once()` 第 17-47 行；`poll()` 第 50-87 行 | 轮询、每轮统一时间戳、读失败隔离和调度 | 不证明实际周期精度 |
| SRC-A042 | `canopen_client.py` | 2026-07-08 | `CanopenSdoClient` 第 20-111 行 | 唯一真实 CANopen 客户端和 `sdo.upload()` | 静态源码不能替代总线抓包 |
| SRC-A043 | `canopen_monitor.py` | 2026-07-09 | 第 6-34 行 | 被动分类 boot-up、heartbeat、EMCY | 当前 CLI 未形成真实 monitor 运行证据 |
| SRC-A044 | `commands.py` | 2026-07-09 | 第 7-67 行 | dry-run 命令数据模型 | 无执行后端 |
| SRC-A045 | `config.py` | 2026-07-08 | 第 14-230 行 | 配置加载、Node-ID 1..127、周期和对象校验 | 配置校验不证明现场参数正确 |
| SRC-A046 | `connection_state.py` | 2026-07-09 | 第 7-24 行 | 连接状态模型 | 未与真实 CAN 客户端集成成台架状态机 |
| SRC-A047 | `csv_logger.py` | 2026-07-08 | 第 11-81 行 | 长表字段及逐对象写行 | 不负责指标分析 |
| SRC-A048 | `diagnostics.py` | 2026-07-09 | 第 7-116 行 | 错误寄存器、SDO abort/timeout 文本分类 | 解析规则不证明对应错误已在台架出现 |
| SRC-A049 | `dry_run.py` | 2026-07-09 | 第 10-52 行 | `would_write` 预览中的 `0x6040/0x607A` | 只是描述对象，不发送报文 |
| SRC-A050 | `errors.py` | 2026-07-08 | 全文件 | 软件异常类型 | 无硬件结论 |
| SRC-A051 | `logging_config.py` | 2026-07-08 | 全文件 | 日志级别配置 | 无采集指标结论 |
| SRC-A052 | `main.py` | 2026-07-09 | 第 21-64、97-174 行 | CLI 仅含 list/read/log/ui；真实读取入口 | 没有 PDO 或控制命令 CLI |
| SRC-A053 | `manifest.py` | 2026-07-09 | 第 15-87 行 | 生成软件、配置、统计和安全字段 | 安全字段为固定布尔声明，不是独立测量 |
| SRC-A054 | `models.py` | 2026-07-08 | 全文件 | 对象、读取结果和采集记录模型 | 模型本身不证明数据正确 |
| SRC-A055 | `object_dictionary.py` | 2026-07-09 | 第 9-129 行 | 10 个只读对象及解码 | 对象可被软件请求，不证明所有固件均支持 |
| SRC-A056 | `profiles.py` | 2026-07-09 | 全文件 | dry-run 设备/轴元数据与软限位模型 | 未连接真实轴 |
| SRC-A057 | `safety_state.py` | 2026-07-09 | 全文件 | dry-run 权限判断 | 软件骨架，不是现场安全放行证据 |
| SRC-A058 | `sampling_stats.py` | 2026-07-09 | 第 6-53 行 | 工作段耗时、失败读取、overrun 和 `estimated_rate_hz` | rate/deadline 字段名称可能被误读，需单列解释 |
| SRC-A059 | `session.py` | 2026-07-09 | 全文件 | 会话与运行元数据模型 | 未形成真实会话数据库 |
| SRC-A060 | `simulated_backend.py` | 2026-07-09 | 全文件 | 模拟只读后端 | 不能作为台架证据 |
| SRC-A061 | `statusword.py` | 2026-07-09 | 第 7-93 行 | Statusword 位解析和 CiA 402 状态标签 | 解析实现不等于完成 CiA 402 控制 |
| SRC-A062 | `ui_server.py` | 2026-07-08 | 全文件 | 静态本地 HTTP 服务 | 无硬件 API |
| SRC-A063 | `ui/app.js` | 2026-07-09 | 第 263-274 行及文件导入/渲染逻辑 | CSV/manifest 展示和 dry-run 文本 | 无 fetch/POST/WebSocket/硬件发送路径 |
| SRC-A064 | `ui/index.html` | 2026-07-09 | 第 20、67、103-104、137 行 | UI 入口、文件导入和 DRY-RUN 标识 | 按钮仅配置/复制/导入，不证明运行安全 |
| SRC-A065 | `ui/styles.css` | 2026-07-09 | 全文件 | UI 样式 | 无功能证据 |
| SRC-A066 | `utils/timebase.py` | 2026-07-08 | 第 7-25 行 | wall/monotonic/elapsed 及 sleep_until | 每个对象没有独立时间戳 |

## 5. 当前测试代码（E2-T，本轮未运行）

以下均位于 `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\`。

| 来源 ID | 文件 | 覆盖定位 | 证明范围 | 不证明范围 |
|---|---|---|---|---|
| SRC-A070 | `test_acquisition.py` | 第 22-32 行 | `read_once` 对象级错误隔离 | 未覆盖 `poll()` 周期和真实 CAN |
| SRC-A071 | `test_canopen_monitor.py` | 第 6-14 行 | boot-up/heartbeat/EMCY 分类 | 未证明真实帧已采集 |
| SRC-A072 | `test_config.py` | 第 11-38 行 | 示例配置、Node-ID校验和覆盖 | 未证明本地设备参数 |
| SRC-A073 | `test_connection_state.py` | 第 6-9 行 | 健康状态标签 | 未连接真实客户端 |
| SRC-A074 | `test_csv_logger.py` | 第 10-39 行 | CSV 长表字段写入 | 未验证大文件或真实时序 |
| SRC-A075 | `test_diagnostics.py` | 第 6-28 行 | 错误位、错误码和文本分类 | 未验证 DS-BVM 特定含义 |
| SRC-A076 | `test_dry_run.py` | 第 9-31 行 | 预览列出 `0x6040/0x607A` 且标记 dry-run | 不证明真实控制存在 |
| SRC-A077 | `test_manifest.py` | 第 10-33 行 | manifest 包含固定安全字段和统计 | 不证明字段来自独立总线观测 |
| SRC-A078 | `test_no_write_api.py` | 第 16-54 行 | Python源码禁用词扫描、模块隔离、UI禁用标签 | 不是完整协议级静态分析或抓包 |
| SRC-A079 | `test_object_dictionary.py` | 第 9-45 行 | 必需对象表与 int32 解码 | 不证明驱动固件支持所有对象 |
| SRC-A080 | `test_profiles.py` | 第 6-11 行 | 软限位模型 | 不证明真实限位 |
| SRC-A081 | `test_safety_state.py` | 第 8-22 行 | dry-run 安全阻断逻辑 | 不证明硬件 E-stop/STO |
| SRC-A082 | `test_sampling_stats.py` | 第 6-18 行 | 当前 rate/deadline 公式语义 | 未覆盖 sleep 后实际轮次周期 |
| SRC-A083 | `test_session.py` | 第 6-12 行 | 会话记录模型 | 未形成持久化运行证据 |
| SRC-A084 | `test_simulated_backend.py` | 第 7-13 行 | 模拟位置步进 | 不是台架采集 |
| SRC-A085 | `test_statusword.py` | 第 6-24 行 | Statusword 状态分类 | 不证明状态机控制实现 |

## 6. 被最新实验记录引用的 SDO 数据组（E1）

每一行包含一个 CSV 与同名 `.manifest.json`。CSV 本轮只核对存在性、表头、首行和末行；所有峰值、RMS和频率沿用 SRC-A008 的既有记录，不重新计算。

| 来源 ID | CSV 与 manifest 完整路径 | 条件/用途 | manifest 定位 | 限制 |
|---|---|---|---|---|
| SRC-A100 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv.manifest.json` | 7 对象、20 ms、30 s、1000 counts 往返 | `command`、`acquisition`、`statistics`、`safety_boundary` | manifest rate 不是实际轮次率 |
| SRC-A101 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv.manifest.json` | 7 对象、20 ms、100 RPM、5 组往返 | 同上 | 方向差异成立于当前参数 |
| SRC-A102 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv.manifest.json` | 7 对象、20 ms、50 RPM、3 组往返 | 同上 | 速度单位/缩放未由手册核验 |
| SRC-A103 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv.manifest.json` | Servo Off、7 对象、10 ms 请求 | 同上 | 只支持静止采集上限判断 |
| SRC-A104 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv.manifest.json` | Servo Off、7 对象、5 ms 请求 | 同上 | 实际仍约 100 Hz；overrun 全轮 |
| SRC-A105 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv.manifest.json` | Servo Off、3 核心对象、5 ms 请求 | 同上 | 约 200 Hz 不等于实时控制周期 |
| SRC-A106 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv.manifest.json` | 5000 counts、50 RPM、300 参数、3 对象 | 同上 | 单次轨迹不能证明重复定位性能 |
| SRC-A107 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv.manifest.json` | 5000 counts、50 RPM、150 参数，前两组完整 | 同上 | 原窗口未覆盖第三组返回 |
| SRC-A108 | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv.manifest.json` | 5000 counts、50 RPM、150 参数，60 s 补采 | 同上 | 仍只代表当前平台与参数 |

## 7. 明确排除

- `90_原始资料/` 中的师兄论文、源码和 DS-BVM 手册；
- `motion_acquisition/scripts/`；
- `.venv`、`.pytest_cache`、`.playwright-cli`、`__pycache__`、`.pyc`、`.egg-info`；
- 未被 SRC-A006、SRC-A007、SRC-A008 明确引用的日志；
- 任何算法路线、候选反馈拓扑和未来控制模式判断。

