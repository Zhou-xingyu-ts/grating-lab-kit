# motion_acquisition 软件证据审计

> 审计对象为当前工作区内的只读采集底座。源码实现（E2-C）、测试代码覆盖（E2-T）、既有报告（E2-R）和真实台架证据（E1）严格分层；本轮没有运行测试、脚本、数据分析或硬件操作。

## 1. E2-C：源码实际实现

| 软件事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| SW-A001 | 真实 CANopen 客户端被设计为 upload-only；硬件读取落点是 `self.node.sdo.upload(index, subindex)`。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\canopen_client.py` | 第 15-24、63-95 行，尤其第 73 行 | 当前客户端源码存在 SDO upload、解码和读取重试路径。 | 静态源码不是总线抓包；不能据此排除第三方库建立连接时的协议帧，也不能证明台架运行成功。 |
| SW-A002 | CLI 仅注册 `list-objects`、`read-once`、`log-sdo` 和 `ui`，没有写对象、Servo On、Jog、Move、Home、PDO配置或闭环控制命令。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\main.py` | 第 35-64、97-174 行 | 当前公开入口是对象查看、单次读取、日志采集和静态界面。 | 不证明其他工作区、Servo Studio 或第三方程序不存在控制能力。 |
| SW-A003 | 对象字典定义 10 个可读对象：0x1000、0x1001、0x1018:01、0x603F、0x6062、0x6064、0x60F4、0x606C、0x6078、0x6041。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\object_dictionary.py` | 第 9-95 行 | 软件层明确了对象索引、子索引、名称和解析类型。 | 不证明 DS-BVM 当前固件支持全部对象，也不证明速度、电流等工程单位和比例准确。 |
| SW-A004 | 一轮采集先生成一次 wall/monotonic/elapsed 时间戳，再依次读取多个对象，同轮对象共用该时间戳。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\utils\timebase.py` | acquisition 第 25-46 行；timebase 第 15-19 行 | CSV 可以按 `sample_seq` 将同轮对象对齐。 | 同轮对象不是严格同步读取；CSV 没有逐对象真实采样时刻。 |
| SW-A005 | 调度在每轮读取后记录工作段耗时，再调用 `sleep_until()` 等待下一目标时刻。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\utils\timebase.py` | acquisition 第 72-84 行；timebase 第 22-34 行 | 源码实现了目标周期驱动的顺序轮询。 | 不是硬实时调度器；没有给出 Windows 唤醒抖动、起点相位误差或最坏响应保证。 |
| SW-A006 | `estimated_rate_hz` 被定义为 `1/mean_cycle_s`，而 `mean_cycle_s` 来自读取与记录工作段，不包含周期等待。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py` | sampling_stats 第 13-52 行；acquisition 第 72-84 行 | 该字段表示平均工作段吞吐率的倒数口径。 | 不能把 manifest 的 `estimated_rate_hz` 直接称为实际采样轮次率。 |
| SW-A007 | `deadline_miss_count` 在单轮工作段耗时大于 `target_period_s` 时加一。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py` | sampling_stats 第 25-27 行；acquisition 第 73-84 行 | 字段可以计数读取工作段 overrun。 | 不等价于完整调度 deadline miss，不覆盖起点迟到、周期相位漂移和逐对象时延。 |
| SW-A008 | `failed_reads` 按对象读取结果累计；CSV 同时保留状态、错误文本和尝试次数。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\csv_logger.py` | acquisition 第 47-80 行；sampling_stats 第 16-19 行；csv_logger 第 10-52 行 | 读取失败、错误和重试信息有记录字段，失败数的单位是“对象结果”。 | 零失败不代表零重试、零延迟或总线没有未捕获异常。 |
| SW-A009 | manifest 写入命令、配置、对象和统计摘要；安全字段由生成器固定写为只读声明。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\manifest.py` | 第 31-93 行，尤其第 77-84 行 | 每次采集的配置和软件声明可随数据留档。 | 硬编码安全字段不是独立协议抓包，不能单独证明物理总线上绝无写报文。 |
| SW-A010 | `WouldWrite` 中的 0x6040/0x607A 仅用于 dry-run 预览；命令对象固定 `dry_run_only=True`。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\dry_run.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\commands.py` | dry_run 第 10-52 行；commands 第 7-67 行 | 软件可以展示“若未来写入将涉及什么”，但不会从该模块连接硬件。 | 不能把预览当作 CiA 402 控制实现、PDO实现或运动安全验证。 |
| SW-A011 | UI 的运动命令仅调用 dry-run 预览，并明确显示没有 execute-to-hardware backend。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\ui\app.js` | 第 263-274 行 | 当前浏览器界面没有硬件执行后端。 | UI 文案本身不能约束外部程序或证明总线安全。 |
| SW-A012 | 当前源码未形成 PDO配置/收发、CiA 402 控制字写入或实时外位置环的数据通路。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\canopen_client.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\main.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\dry_run.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\commands.py` | canopen_client 第 15-95 行；main 第 35-174 行；dry_run 第 10-52 行；commands 第 7-67 行 | 在当前入口和真实客户端中，硬件功能止于 SDO upload 读取。 | 这是对本包当前源码的结论，不是对 DS-BVM 能力或未来方案的否定。 |
| SW-A013 | 诊断模块可解析 Error Register 位、归类部分 SDO abort/timeout 文本并形成诊断摘要。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\diagnostics.py` | 第 7-116 行 | 当前软件存在对已读取错误值和异常文本的解释层。 | 分类规则存在不证明对应故障已在当前台架发生，也不证明其含义已按 DS-BVM 手册逐项确认。 |

## 2. E2-T：测试代码覆盖

| 软件事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| SW-A020 | `test_no_write_api.py` 检查指定源码目录中的禁用调用/词项、dry-run 模块导入边界和 UI 只读标签。 | E2-T | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_no_write_api.py` | 第 16-54 行 | 测试设计中存在针对“无写 API”的静态守护。 | 本轮未运行；关键字扫描不是完整语义分析，也不是总线抓包。 |
| SW-A021 | 采集错误隔离、配置、CSV、统计、manifest 和对象字典有对应测试文件。 | E2-T | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_config.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_csv_logger.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_sampling_stats.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_manifest.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_object_dictionary.py` | 各文件中的 `test_*` 用例；具体覆盖见 `00_SOURCE_INVENTORY.md` 的 SRC-A070、A072、A074、A077、A079、A082 | 这些关键纯软件逻辑存在单元测试入口。 | 没有专门的 `test_canopen_client.py` 或 `test_timebase.py`；测试代码存在也不等于当前环境通过或真实驱动器验证。 |
| SW-A022 | CAN帧分类、连接状态、诊断、dry-run、安全状态、会话、模拟后端和 Statusword 等辅助模块有测试文件。 | E2-T | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_canopen_monitor.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_connection_state.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_diagnostics.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_dry_run.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_no_write_api.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_profiles.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_safety_state.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_session.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_simulated_backend.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_statusword.py` | 各文件中的 `test_*` 用例；具体覆盖见 `00_SOURCE_INVENTORY.md` 的 SRC-A071、A073、A075、A076、A078、A080、A081、A083-A085 | 当前测试树中的其余辅助模型和只读边界存在测试入口。 | 没有专门的 UI、CLI主命令、真实 SDO客户端、真实 poll 时序或硬件集成测试文件；本轮也没有运行这些测试。 |

## 3. E2-R：既有运行报告

| 软件事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| SW-A030 | 2026-07-10 发布说明记录当时 `31 passed in 0.47s`，并记录静态只读检查与 Playwright 页面检查。 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\release_v0.2.0_readonly_station_foundation.md` | 第 83-112 行 | 7 月 10 日曾形成一份软件测试/检查结果报告。 | 本轮未复跑，不能把历史通过写成当前再次通过；报告也不是台架控制验证。 |
| SW-A031 | 两份 Playwright 输出日志只保留页面截图命令/结果文本，详细断言和边界说明仍需回到发布说明。 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\output\playwright\desktop_screenshot_log.txt`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\output\playwright\mobile_screenshot_log.txt`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\release_v0.2.0_readonly_station_foundation.md` | 两份日志全文；发布说明第 83-112 行 | 页面曾被自动化浏览器打开并留下报告性痕迹。 | 单独日志不能证明所有交互断言、布局边界或真实硬件能力。 |
| SW-A032 | 同一发布说明在 7 月 10 日明确写明尚未完成真实硬件验证；7 月 12 日实验记录随后提供了只读台架 E1。 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\release_v0.2.0_readonly_station_foundation.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 发布说明第 114-127 行；实验记录第 15-68 行 | 说明历史软件报告与后续真实台架证据的时间关系。 | 后续 E1 只验证记录中实际执行的 upload-only 采集，不使全部软件测试自动升级为台架验证。 |

## 4. E1：真实台架与软件的交叉证据

| 软件事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| SW-A040 | 7 月 12 日记录显示 Node 1、1 Mbps 下完成 7 对象运动采集；CSV/manifest 与该次记录对应。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv.manifest.json` | 实验记录第 15-68 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 当前包的日志链路确实在实际 DS-BVM 台架上生成过只读采集证据。 | 不证明 PDO、写控制、Windows硬实时或外位置环。 |
| SW-A041 | 7 月 13 日追加实验记录了 7 对象约 100 Hz 上限与 3 对象约 200 Hz 上限，且条件为 Servo Off 静止。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv` | 实验记录第 127-137 行；各 CSV 表头与首尾行 | 当前 USB-CAN + SDO顺序轮询对对象数敏感。 | 不能将 3 对象静止读取外推为 7 对象运动 200 Hz 或控制环频率。 |

## 5. 两项必须固定的诊断语义

### 5.1 `estimated_rate_hz` 不是实际轮次率

| 数据文件 | manifest 的 `estimated_rate_hz` | 实验记录/CSV时间戳口径 | 审计判断 |
|---|---:|---:|---|
| `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv.manifest.json` | 约 95.53 Hz | 约 49.999 Hz | 前者是读取工作段吞吐口径，后者才是本次 20 ms 调度的实际轮次率口径。 |
| `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv.manifest.json` | 约 232.86 Hz | 约 199.90 Hz | 不能用前者宣称 232.86 Hz 实际采样。 |

上述数值仅转录既有记录与 manifest，没有在本轮重新统计 CSV。

### 5.2 `deadline_miss_count` 不是完整调度抖动

- 当前实现只比较“该轮读取和记录工作段耗时”与目标周期。
- 它没有比较计划起点与真实起点，没有计算周期相位误差，也没有逐对象时间戳。
- 因此它只能回答“工作段是否超期”，不能回答 Windows 调度是否确定、每轮是否准时开始或各对象是否同步。

## 6. 软件层审计结论

1. 当前真实 CAN 路径止于 SDO upload 只读采集；没有 PDO、CiA 402 写控制或实时外环实现。
2. dry-run 的 0x6040/0x607A 是预览模型，不是控制实现。
3. 源码实现、测试代码、历史报告和 7 月 12 日台架证据分别成立，不能相互替代。
4. manifest 的安全字段必须与 upload-only 源码、测试覆盖和实验记录交叉佐证；它本身不是独立总线安全证据。
5. 当前诊断足以描述工作段耗时、对象级失败和日志元数据，但不足以描述完整调度抖动和逐对象采样时序。
