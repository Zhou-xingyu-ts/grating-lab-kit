# Motion Acquisition V2.0 本机离线构建与验收记录

验收日期：2026-07-24

## 结论

V2.0 通过当前构建电脑上的离线验收。平台已在 V1.0 项目—采集—回放—分析—交付闭环上增加显式时间/数据质量、标定证据、成组实验、可复现离线算法和无设备写能力的影子模式。

本记录没有访问 CAN、USB 或运动硬件，也没有第二台 Windows 电脑、外部长度基准或设备同步时钟证据。

## 产品能力

- 新会话显式记录主机轮次起点时间语义、时钟来源、同步/硬实时否定和限制；
- 流式、内存有界、可取消的数据质量评价，输出 JSON/CSV/Markdown；
- 线性仿射标定、参考点/证据/残差和五类独立精度 claim；
- 只有每个点都有可追溯外部基准时才是 `externally_referenced`，近似 10000-count 关系保持 `draft`；
- 2～20 个相对路径、SHA-256 固定的重复/方向/速度/加速度/稳定性实验组；
- 跟随误差、观测位移、方向闭合、末段稳定性和漂移的描述性评价；
- 移动平均、线性漂移、轨迹运动学、误差补偿表、仿射误差模型和 PID 影子六种算法；
- 原子、可取消、带输入/配置/代码指纹和校验账本的算法运行目录；
- 实时影子逐条保存 `shadow_outputs.jsonl`，GUI 只显示限频副本；
- 一个增量“研究与算法”页，研究任务在独立可取消 QThread 中运行；
- V1.0 项目、预设、事件、采集、浏览、回放、分析、比较、恢复、诊断和交付全部保留。

## 数据和兼容性

`samples.csv` 仍为原有 16 列 raw 数据。V2.0 只向 schema-1.0 manifest 增加可选 `time_semantics`、`calibration`、`experiment_group`、`shadow` 和对应文件引用。

持久旧数据审计覆盖：

- 会话：23
- 记录：266,056
- outcome：14 legacy、7 completed、2 stopped
- 扫描/回放/普通分析/质量评估兼容：23/23
- CSV SHA-256 改变：0
- 旧会话显式 V2 时间语义：0/23，均按旧主机时间格式保守解释
- 质量状态：6 good、17 warning；warning 包含旧时间语义或既有数据质量提示，不被改写

## 自动测试

- 未修改 V1.0 基线：`86 passed in 13.24s`
- V2 核心服务合并回归：`102 passed in 19.95s`
- GUI 接线修复后定向：`25 passed in 6.26s`
- V2 GUI/安全集成子集：`10 passed in 3.43s`
- 文档/版本/最终源码回归：`109 passed in 15.44s`
- 最终无写/可移植/V2 专项：`28 passed in 4.41s`
- `compileall`：通过
- `pip check`：`No broken requirements found.`

测试覆盖显式/旧时间语义、质量异常与取消、12,050 轮有界分位数缓存、草稿/外部证据标定、精度 claim 分离、便携组引用与哈希锁定、六种算法及不可辨识拒绝、重现指纹、取消清理、影子全量记录、GUI 研究工作流、16 列 CSV、旧会话兼容和静态无写边界。

## 独立 Demo 冒烟

产物位于 `build\V2_Acceptance_Smoke_20260724`，明确是合成 Demo/工程假设，不是现场证据。

主要两会话：

| 会话 | outcome | 轮次 | 记录 | 实际轮次率 | 失败 | deadline miss | 事件 | checksum |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| v2-smoke-completed | completed | 9 | 27 | 145.42 Hz | 0 | 0 | 1 | verified |
| v2-smoke-active-stopped-2 | stopped | 9 | 27 | 157.25 Hz | 0 | 0 | 1 | verified |

两者均为 16 列 CSV，使用明确标为草稿的合成标定，产生逐条影子输出。完成了两会话方向组评价、质量三格式导出和 moving-average 可复现算法运行。活动停止会话包含 9 行影子输出。

另外保留两个合法的“开始阶段立即停止、0 轮”会话，用于证明零样本 stopped 收尾。冒烟脚本最初等待 `recorder.is_open`，发现该状态早于初始 manifest 完成；最终改为等待 `AcquisitionState.ACQUIRING` 后停止，没有修改生产代码。

## 发布包

- 路径：`release\MotionAcquisition`
- 类型：Windows x64 PyInstaller `onedir --windowed`
- 构建时间：125.9 s
- 文件数：381
- 总大小：170,231,880 bytes（162.35 MiB）
- `MotionAcquisition.exe` SHA-256：`0A0A8F16E92A875914F41244116F8E44CF3D80CA891FE615A486D04D7786B6BE`
- V2 必需文档缺失：0
- 固定用户名/构建盘符/源码路径命中：0
- 内嵌会话文件：0
- V0/V1 旧发布文档：0
- 旧网页资源：0

成品 EXE 已直接启动，Windows 可访问性树确认 V2.0 标题、工作台和八个功能页。桌面自动化输入层连续返回 `call get_window_state before issuing coordinate input`，因此按工具恢复规则停止继续注入；未把没有执行的点击写成成品交互证据。完整 GUI 交互由同一构建源码的 Qt 离屏端到端测试证明。

## 安全和延期项

- 源码静态扫描未发现 SDO download、NMT/PDO、raw send 或运动控件路径。
- manifest、运行结果、诊断和完整性检查都声明 device/raw-CAN/shadow write capability 为 false；相反声明会被判为错误。
- 未来只读源协议只有 `open/read_batch/close`，V2.0 未实现 PDO、EtherCAT 或外部传感器。
- 待现场验证：PCAN_USBBUS1、1 Mbps、Node-ID 1、静止/运动只读采集、断线和 Servo Studio 配合。
- 待外部标定：可追溯基准、测量/重复/绝对定位精度。
- 待时间验证：设备时间戳、外部同步和高频采集。
- 待第二台 Windows：完整目录复制、启动、用户隔离、Demo、回放和研究工作流。
