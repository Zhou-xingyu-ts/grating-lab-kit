# 当前系统研究事实表

> 仅使用阶段 A 标签。每条事实分别保留 E1、E2-C、E2-T、E2-R 或 H，不把不同证据层合并成“已验证”。

## 1. 当前台架与通信分工

| 事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| FACT-A001 | 当前实际位置反馈是 Ri0040 经绿板 DB15 接入 DS-BVM 数字编码器口，并在该实验配置中作为唯一位置反馈源。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-18_光栅尺单反馈闭环_Scope数据采集_10Hz限制确认.md` | 第 5、22、55 行 | 当前台架确实运行过光栅尺单反馈位置闭环。 | 不证明 DS-BVM 能同时接收电机编码器与光栅反馈，也不判断未来双反馈拓扑。 |
| FACT-A002 | 当前 6 月 Scope 数据中 Position Actual 能跟随 Position Demand，记录据此判定光栅尺参与位置闭环。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-18_光栅尺单反馈闭环_Scope数据采集_10Hz限制确认.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-22_CANopen链路_USB-CAN实测_scope-can622数据分析.md` | 06-18 第 55-64 行；06-22 第 60-88 行 | 单反馈闭环链路具备运动跟随现象。 | 10 Hz Scope 不能严肃量化峰值、振荡频率或高频动态。 |
| FACT-A003 | 7 月 12 日实验中，运动命令由 Servo Studio 经 USB/COM 下发，USB-CAN/Python 同时只做 CANopen SDO upload。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 4-5、15-26、56-60 行 | 两条物理连接可并行承担“运动控制”和“只读采集”两个角色。 | 不证明 Windows/CANopen 已形成实时控制环，也不证明 CANopen 可下发运动。 |
| FACT-A004 | 当前台架 CANopen 通信使用 `PCAN_USBBUS1`、1 Mbps、Node-ID 1。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\configs\canopen_readonly.local.yaml` | 实验记录第 15-25 行；配置第 1-12 行 | 7 月 12 日现场扫描、SDO返回和配置一致支持 Node 1 / 1 Mbps。 | 不证明其他驱动器、其他固件或未来台架仍使用相同参数。 |
| FACT-A005 | 6 月记录中的 `node 622` 是当时 Servo Studio/文件命名语境，7 月记录明确否定其可作为 `1..127` 的 CANopen Node-ID；当前值取 1。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-22_CANopen链路_USB-CAN实测_scope-can622数据分析.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 06-22 第 5、97 行；07-12 第 15-17 行 | 解释了历史文件名与当前 Node-ID 的时间顺序，不构成两个并列当前值。 | 不解释 Servo Studio 当时显示 `622` 的内部编码含义。 |

## 2. CANopen 只读边界的分层证据

| 事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| FACT-A006 | 7 月现场实验明确限制 CANopen 为 SDO upload，不写对象、不发 NMT Operational、不配置 PDO。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 4-5、63-68 行 | 本次实验的操作边界和记录结论是只读采集。 | 记录声明不是独立总线抓包，不能单独证明总线绝无其他帧。 |
| FACT-A007 | 当前真实 CANopen 客户端只暴露读取接口，并在 `read()` 中调用 `self.node.sdo.upload()`。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\canopen_client.py` | 第 15-24、63-95 行，尤其第 73 行 | 当前 Python 源码的真实硬件路径实现了 SDO upload 和读取重试。 | 静态源码不能替代运行时抓包，也不能排除第三方库连接阶段的协议帧。 |
| FACT-A008 | 当前 CLI 只有 `list-objects`、`read-once`、`log-sdo` 和静态 UI，没有真实 PDO、CiA 402 写控制或实时闭环命令。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\main.py` | 第 35-64、97-174 行 | 当前入口没有 Servo On、Jog、Move、Home、PDO配置或控制字写入命令。 | 不证明整个电脑环境或其他项目不存在写控制工具。 |
| FACT-A009 | `0x6040` 和 `0x607A` 只出现在 `WouldWrite` dry-run 预览中，预览对象没有连接真实 CAN 客户端。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\dry_run.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\commands.py` | dry_run 第 10-52 行；commands 第 7-67 行 | 运动词汇和地址是界面/模型预览，不是硬件写入实现。 | 不能据此宣称未来运动控制已实现或已通过安全验证。 |
| FACT-A010 | 测试代码扫描 Python 源码中的若干写调用/标签，并检查 dry-run 模块不导入真实客户端。 | E2-T | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\tests\test_no_write_api.py` | 第 16-54 行 | 当前测试设计覆盖指定禁用词、模块隔离和 UI 标签。 | 本轮没有运行；该测试也不是完整协议级静态分析或总线抓包。 |
| FACT-A011 | 7 月 10 日发布说明记录历史结果为 `31 passed in 0.47s`，并报告静态检查未发现指定写控制词。 | E2-R | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\docs\release_v0.2.0_readonly_station_foundation.md` | 第 83-127 行 | 该日期曾形成软件测试与静态检查报告。 | 本轮未复跑；报告早于 7 月 12 日真实台架采集，不能替代 E1。 |

## 3. 读取对象与采样能力

| 事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| FACT-A012 | 当前软件对象表包含 10 个可读对象：0x1000、0x1001、0x1018:01、0x603F、0x6062、0x6064、0x60F4、0x606C、0x6078、0x6041。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\object_dictionary.py` | 第 9-95 行 | 软件知道这些对象的索引和解码类型。 | 不证明当前 DS-BVM 固件支持全部对象，也不证明其单位/缩放正确。 |
| FACT-A013 | 7 对象 20 ms 运动采集使用 position demand/actual、following error、velocity、current、statusword、error code；记录实际约 49.999 Hz、零失败。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv.manifest.json` | 实验记录第 30-61 行；manifest `command/acquisition/statistics` | 当前台架在该对象集和参数下完成了约 50 Hz 只读采集。 | 不证明 50 Hz 是驱动内部周期、确定性实时周期或控制带宽。 |
| FACT-A014 | Servo Off 静止上限实验记录：7 对象 10 ms 请求约 98.62 Hz，7 对象 5 ms 请求约 101.62 Hz，3 对象 5 ms 请求约 199.90 Hz。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv.manifest.json`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv.manifest.json`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv.manifest.json` | 实验记录第 127-137 行；各 CSV 表头与首尾行；各 manifest 的 `command/acquisition/statistics` | 对象数显著影响当前 SDO 轮询可达轮次率。 | 静止读取上限不等于运动中所有对象的 200 Hz，也不等于实时闭环能力。 |
| FACT-A015 | manifest 的 `estimated_rate_hz` 计算为 `1/mean_cycle_s`，其中 cycle 只覆盖读取和写CSV的工作段，不包含 `sleep_until()` 等待。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py` | acquisition 第 72-84 行；sampling_stats 第 13-52 行 | 解释了为何 manifest 的约 95/225 Hz 与实验记录的约 50/200 Hz 不同。 | manifest `estimated_rate_hz` 不能直接作为实际轮次频率。 |
| FACT-A016 | `deadline_miss_count` 实际统计“读取工作段耗时大于 target_period”的轮数。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py` | sampling_stats 第 25-27 行；acquisition 第 73-84 行 | 它可作为工作段 overrun 指标。 | 它不包含完整调度相位误差、Windows唤醒抖动或每轮起点偏差。 |
| FACT-A017 | 每轮在读取第一个对象前只生成一次 wall/monotonic/elapsed 时间戳，随后同一时间戳写给该轮所有对象。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\utils\timebase.py` | acquisition 第 25-46 行；timebase 第 15-19 行 | CSV 能按轮次对齐对象。 | 不存在逐对象真实读取时刻，不能把一轮对象视为严格同步采样。 |
| FACT-A018 | `failed_reads` 按对象读取结果累计，而不是按采样轮累计。 | E2-C | `D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\acquisition.py`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\src\motion_acquisition\sampling_stats.py` | acquisition 第 74-80 行；sampling_stats 第 16-19 行 | manifest 的零失败表示记录层未累计到对象级失败结果。 | 不等于总线不存在重试、时延或未被记录的报文异常。 |

## 4. 位移换算、方向差异和参数敏感性

| 事实 ID | 结论 | 标签 | 完整文件路径 | 定位信息 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| FACT-A019 | 记录中存在用户给出的 `5 mm≈10000 counts`，并据此暂取 `1 count≈0.5 μm`。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 182-190 行 | 证明项目采用了一个明确标注的暂定工程换算。 | 未经千分表、位移计或其他外部基准，不是正式标定或计量溯源。 |
| FACT-A020 | “当前真实比例精确等于 0.5 μm/count”仍属未确认主张。 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 190、203 行 | 固定该换算的证据边界。 | 不能用暂定比例宣称绝对定位精度、重复定位精度或光栅计量精度。 |
| FACT-A021 | 1000 counts、100 RPM 的五组数据在当前参数下显示返回方向动态峰值和部分残差大于正向。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv.manifest.json` | 实验记录第 83-102 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 方向相关差异在重复往返中出现，不只是单次截图现象。 | 不支持把差异归因于摩擦、重力、间隙、刹车、反馈噪声或某一参数。 |
| FACT-A022 | 将速度从 100 RPM 降到 50 RPM 后，记录中的返回峰值和残差范围改善，说明现象具有速度相关性。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv.manifest.json` | 实验记录第 106-123 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 当前台架在这两个设置之间存在可重复的参数敏感性。 | 不证明速度是唯一原因；0x606C 单位/缩放尚未由本轮手册审计确认。 |
| FACT-A023 | 5000 counts、50 RPM 下把 Accel/Decel/Jerk 从 300 降到 150 后，三组记录中的正向稳态 RMS 相对单次 300 参数结果改善。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv.manifest.json`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv.manifest.json`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv.manifest.json` | 实验记录第 141-180 行；各 CSV 表头与首尾行；各 manifest 的 `command/acquisition/statistics` | 当前负载和当前轨迹下存在加减速/jerk参数敏感性。 | 300参数只有一次5000-count基线，不能推出普适最优参数或因果模型。 |
| FACT-A024 | 摩擦、重力、机械间隙、线缆牵引、反馈噪声或控制参数分别导致方向不对称，当前均未确认。 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 102、123、163、180 行 | 明确当前只有现象和参数相关性证据。 | 不能在阶段 A 选择单一物理原因或控制算法。 |

## 5. 当前研究状态边界

- 当前 E1 足以支持“代理单轴已有真实采集和参数敏感性现象”，不足以支持自研光栅尺、三轴平台或新控制算法结论。
- E2-C、E2-T、E2-R 共同支持“当前软件定位为只读采集底座”，但只有 E1 能证明 7 月 12 日真实 DS-BVM 台架上发生过 SDO upload 采集。
- 本表不判断未来电机侧反馈加光栅外位置环，也不判断任何候选控制算法。
