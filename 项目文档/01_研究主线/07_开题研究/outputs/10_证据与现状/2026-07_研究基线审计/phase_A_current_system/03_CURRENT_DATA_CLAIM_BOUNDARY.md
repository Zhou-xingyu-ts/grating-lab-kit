# 当前数据主张边界

> 本文只审计既有实验记录、manifest 与原始文件的对应关系。CSV 仅核对存在性、表头和首尾结构；本轮没有重新计算采样率、RMS、峰值、残差或其他统计量。

## 1. 分组证据与可用主张

| 数据 ID | 实验条件与既有结论 | 标签 | 完整文件路径 | 定位信息 | 支持什么结论 | 禁止外推什么结论 |
|---|---|---|---|---|---|---|
| DATA-A001 | Servo Studio Scope 的 USB 数据与 CAN背景数据均约 10 Hz，只用于趋势和链路背景。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-18_光栅尺单反馈闭环_Scope数据采集_10Hz限制确认.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-06-22_CANopen链路_USB-CAN实测_scope-can622数据分析.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\README.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\scope_usb_2026-06-18.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\62_Scope双总线数据采集\scope_can622_2026-06-22.csv` | 06-18 记录第 55-68 行；06-22 记录第 60-101 行；数据目录 README 第 8-48 行；两份 CSV 表头与首尾行 | 支持“Position Actual 随 Demand 变化”和两份 Scope 数据均为约 10 Hz 趋势证据。 | 不支持严肃量化动态峰值、振荡频率、控制带宽、高频噪声或 CANopen SDO 的实际轮询能力。 |
| DATA-A002 | `195923`：运动中，7 对象，20 ms 请求，1500 轮；既有记录给出约 49.999 Hz、对象级读取失败为 0。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_195923.csv.manifest.json` | 实验记录第 30-68 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持当前 Node 1 / 1 Mbps / 7对象配置在运动中完成约 50 Hz 的只读轮询记录。 | 不支持 50 Hz 为驱动内环频率、控制带宽、硬实时周期或所有工况上限。 |
| DATA-A003 | `201548`：Servo Off 静止，7 对象，10 ms 请求；既有记录给出约 98.62 Hz。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201548.csv.manifest.json` | 实验记录第 127-137 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持当前静止条件下 7 对象的实际轮次率接近 100 Hz。 | 不支持运动中 7 对象稳定达到同样轮次率，也不支持确定性实时性。 |
| DATA-A004 | `201745`：Servo Off 静止，7 对象，5 ms 请求；既有记录给出约 101.62 Hz。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201745.csv.manifest.json` | 实验记录第 127-137 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持将请求周期从 10 ms 降到 5 ms 时，7 对象轮询没有相应提升到 200 Hz，当前瓶颈约在 100 Hz 量级。 | 不支持把瓶颈唯一归因于 Windows、USB-CAN、DS-BVM、SDO协议或某一代码段。 |
| DATA-A005 | `201831`：Servo Off 静止，3 对象，5 ms 请求；对象为 position demand、position actual、following error；既有记录给出约 199.90 Hz。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201831.csv.manifest.json` | 实验记录第 127-137 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持减少到 3 对象后，当前静止轮询可接近 200 Hz；支持对象数影响可达轮次率。 | 不支持运动中 200 Hz、7 对象 200 Hz、逐对象同步采样或 200 Hz 外位置闭环。 |
| DATA-A006 | `200718`：1000 counts、100 RPM、Accel/Decel/Jerk 300，五次往返；既有记录显示返回方向动态峰值绝对值及部分残差大于正向。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_200718.csv.manifest.json` | 实验记录第 83-102 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持当前轨迹和参数下存在重复出现的方向相关差异。 | 不支持把差异归因于摩擦、重力、间隙、刹车、线缆牵引、反馈噪声或某个控制参数。 |
| DATA-A007 | `201224`：1000 counts、50 RPM、Accel/Decel/Jerk 300，三次往返；既有记录显示相较 100 RPM，返回峰值和残差范围改善。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_201224.csv.manifest.json` | 实验记录第 106-123 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持误差现象对速度设置敏感，降低速度后部分既有指标改善。 | 不支持速度是唯一因果变量，也不支持把 0x606C 的工程单位或比例视为已由手册确认。 |
| DATA-A008 | `202323`：5000 counts、50 RPM、Accel/Decel/Jerk 300，3 对象，单次往返；既有记录给出实际约 199.325 Hz及正/返稳态 RMS。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202323.csv.manifest.json` | 实验记录第 141-163 行；CSV 表头与首尾行；manifest 的 `command/acquisition/statistics` | 支持在较长行程和该参数下采到一组可与 150 参数组比较的单次基线。 | 单次基线不支持重复性结论、普适方向规律或最优参数结论。 |
| DATA-A009 | `202749`、`203414`：5000 counts、50 RPM、Accel/Decel/Jerk 150，共三次往返；既有记录显示正向稳态 RMS 相较单次 300 参数基线改善。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_202749.csv.manifest.json`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv`；`D:\清华\清华-科研横向\光栅尺运动控制调研\motion_acquisition\logs\motion_sdo_20260712_203414.csv.manifest.json` | 实验记录第 165-180 行；两份 CSV 表头与首尾行；两份 manifest 的 `command/acquisition/statistics` | 支持当前负载/轨迹下误差指标对 Accel/Decel/Jerk 设置敏感，降低这些设置后部分指标改善。 | 不支持 150 为最优值；300 组仅一次，不能完成严格统计比较或因果识别。 |
| DATA-A010 | `10000 counts≈5 mm` 与 `1 count≈0.5 μm` 是用户提供并在记录中明确标为暂定的工程换算。 | E1 | `D:\清华\清华-科研横向\光栅尺运动控制调研\02_实验与现场\01_当前执行\60_实验室推进记录\2026-07-12_USB控制_CANopen只读_50Hz动态采集.md` | 第 182-203 行 | 支持在内部讨论中暂时把 counts 转成约 mm/μm 的量级。 | 没有外部位移基准，不能写成正式标定、绝对定位精度、重复定位精度或计量溯源结果。 |

## 2. 当前数据能够支持的研究结论

1. 当前单轴代理台架上已形成“Servo Studio 经 USB/COM 发运动命令 + USB-CAN/CANopen SDO upload 只读采集”的真实证据链。
2. 当前顺序 SDO 轮询的实际轮次率对读取对象数敏感：7 对象约 100 Hz 上限、3 对象静止时可接近 200 Hz；运动中 7 对象 20 ms 记录约 50 Hz。
3. 在既有轨迹和参数下，往返方向存在可重复观察到的误差差异。
4. 降低速度，或降低 Accel/Decel/Jerk 后，既有记录中的部分误差指标改善，支持“参数敏感性”这一现象级主张。
5. Scope CSV 可作为单反馈跟随和约 10 Hz 趋势证据，不能承担高频动态分析。

## 3. 当前数据不能支持的研究结论

1. 不能把方向差异归因为摩擦、重力、间隙、刹车、线缆牵引、反馈噪声或某个控制参数；当前没有因果隔离实验。
2. 不能宣称绝对定位精度、重复定位精度、分辨率、线性度或计量溯源；`0.5 μm/count` 不是正式标定。
3. 不能宣称自研光栅尺性能、三轴平台性能或三轴同步能力；这些对象尚未进入本轮证据范围。
4. 不能把 SDO 只读轮询率等同于控制频率、驱动内环频率、控制带宽、PDO周期或实时外位置环能力。
5. 不能从 manifest 的 `estimated_rate_hz` 宣称实际轮次率，也不能从 `deadline_miss_count` 宣称完整调度抖动已得到验证。
6. 不能把 3 对象、Servo Off、静止约 200 Hz 外推为 7 对象运动 200 Hz。
7. 不能从一组 300 参数基线与三组 150 参数结果推出普适最优参数或算法创新。

## 4. 审计处理说明

- 本轮未重新计算 RMS、峰值、采样率或统计置信区间；所有数值来自既有实验记录或 manifest，并按其原始口径转录。
- CSV 的用途仅限于确认文件存在、字段结构、Node-ID、对象行结构和首尾时间结构与记录相符。
- 后续若要把现象升级为论文结论，应先关闭计数标定、采样时序定义和方向差异因果识别缺口。
