# Motion Acquisition V9.0.0 桌面修订与历史候选发布说明

<!-- last_reviewed: 2026-10-23 -->

## 当前桌面修订：20261023-ux2

本轮用于自用实验和开题展示，基础版本仍为 9.0.0；界面与包显示构建修订号，防止误开旧 EXE。固定单轴运动、原始 counts、16 列 CSV、单主站及默认不连接设备的行为继续沿用。

- 直控窗口显示中文阶段、使能状态、采样质量和数据新鲜度；读取失败不会把旧值伪装成当前读数。
- 直控位置叠加图和跟随误差图复用同一采集流，以有界缓存、限频刷新显示，不新增 CAN 连接。
- Direct 记住上次保存目录和实验名，首次继承主窗口数据根；独立偏好不会覆盖主窗口配置或恢复连接/使能。
- “回放 / 保存结果图”打开本次会话，非默认保存目录也可索引并自动生成基础摘要；旧分析忙时排队处理最新实验，避免选中 B 却展示 A 的结果。
- 回放提供给定/实际位置、跟随误差双图及完整会话 PNG 导出，含名称、相对时间与 counts；缺少信号明确显示未记录。支持回到开头和轮次/时间显示。
- 实时、回放及比较图缺测断线；比较图抽点保留断点，回放新鲜度独立维护且跳转后正确重建。
- 主窗口提供常用/全部功能切换，路线入口保留；基础统计与离线分段指标明确区分。
- 窗口按屏幕可用空间启动，复杂页面可滚动；直控运动按钮固定底部，展开详情或日志时仍可操作。
- 桌面打包临时优先解析 Windows 系统 DLL，避免构建环境中的第三方同名 ICU 库造成 QtCore 启动失败；不会修改系统环境。
- 控制核心在预采集、驻留和最终采集阶段检查驱动异常，失败走既有 Halt→Disable；受管直控运行前刷新预检，起点、配置记录与返回偏差使用同一新快照。
- 包含此前 7 项修复及 10-23 复查的 6 项修复，逐项见 `V9_0_DEFECT_LEDGER.md`。GUI 仍展示基础摘要，分段超调等由独立离线分析管线计算。包内说明来自 `packaging/README_DESKTOP.txt`；构建仅使用 `packaging/build_desktop.ps1`，不构建固件或覆盖历史发布物。

最终测试结果、包路径与启动验证以工程 `CLAUDE.md` 的当前记录为准。本轮不增加硬件验证结论，不更改自研光栅尺尚未接入的事实；历史分析报告保持原样。

## 2026-08-09 历史候选：发布类型与边界

以下为当时发布快照，不代表当前桌面修订的测试、固件或打包记录。

版本目标：V9.0.0  
发布类型：离线候选，止于线下实验开始前  
当前状态：最终离线候选 `MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 已通过源码目录外启动核验  
当前结论：`READY_FOR_LAB_TEST`，仅限离线软件就绪，不表示真实硬件通过

本轮已有离线 Python（257 passed / 0 failed / 0 skipped，40.00 s）、定向静态 + ESI（31 passed，1.26 s）、Route B 三 lane 构建和官方 ESI 审计证据。Route B 构建记录在 `build/v9_offline_completion_20260809_15/`：Host CTest 非空且零失败，Arm GNU 与 Keil target 构建成功。官方 ESI 审计记录在 `build/v9_offline_completion_20260809_22/`，输入 SHA-256 为 `92733290d5adc3fad608b78b174043123cfae44c2c9d834a574c8b10010cb69d`，JSON、Markdown 与 SHA256SUMS 齐全。上述工作均未访问真实硬件。

外部工作目录验证见 `build/v9_offline_completion_20260809_27/FINAL_CANDIDATE_LAUNCH_VERIFICATION.md`：最终离线候选 `MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 的默认 GUI 启动/关闭、未点击连接的 Direct 窗口启动/关闭，以及 SIL localhost protocol 4.0 hello/connect/poll/disconnect 均 PASS；候选清单 415 条 SHA-256 通过，且没有固定路径、原始 data、logs 或 CSV。验证未访问真实端点。逐项构建、审计与候选证据见[构建与发布证据记录](V9_0_BUILD_RELEASE_EVIDENCE.md)。

## V9 目标内容

- 统一 Route Manager：三条物理路线的描述、配置校验、证据快照、单主站资源锁、状态机和路线专属关闭；
- Route A 的兼容包装：保留现有 DirectMotionAcquisitionService、固定直控基线和独立 motion-acq-direct 回退入口；候选根目录还必须有单独命名的 `MotionAcquisitionV9Direct.exe`；
- Route B 的离线候选：protocol 4.0 可替换字节传输、串口惰性实现、Virtual DS-BVM、portable STM32 候选和安全默认配置；
- Route C 的离线候选：GM1 Modbus 契约、Mock GM1、ESI/XML 只读审计产物入口，且 Windows 不成为 EtherCAT 主站；
- GUI 路线卡、Offline/SIL 门禁，以及用户显式触发的离线预检证据记录；
- 可选 route 会话元数据、配置模板、验证报告、缺陷表、声明—证据矩阵和三条线下测试入口。
- 最终候选要求包含 pyserial 与 pymodbus，但它们只为未来获授权的 transport 提供惰性依赖；包内存在不等于有端点、连接权限或自动 I/O。

以上条目在实际发布前均须以验证报告的证据为准，不能仅凭本说明写成已完成。

## 保持不变

- 16 列 samples.csv、raw counts、旧会话字节、checksums.sha256 和历史控制记录；
- 旧 SCHEME_A/B/C 闭环职责档案与 profile ID 的可读性；
- Route A 的既有固定单轴 CiA 402/PPM 行为、bit10/bit12 判据、共享 CANopen client 和 Halt→Disable 收尾；
- 独立直控窗口与 motion-acq-direct 入口；
- V8 固件证据、历史 release 和原始资料，不覆盖、不重命名、不手改。

## 默认执行不做的事

- 默认 `MotionAcquisitionV9.exe` 启动和打包流程不执行真实端口连接、扫描、烧录、复位、NMT、SDO 下载、PDO、SYNC、Modbus 写、EtherCAT 状态切换或运动；
- 已验证的 STM32 板型、串口、FDCAN、收发器、CAN 总线电气、GM1 工程或 EtherCAT 设备；
- 已验证的设备 identity、PDO/SYNC/DC、三轴、实时性、精度、反馈或安全链；
- Windows EtherCAT master 依赖。

`MotionAcquisitionV9Direct.exe` 是保留的 Route A 直控回退入口，而不是默认执行路径。启动窗口
本身不自动连接；但在获逐项授权后，窗口中的预检或控制动作可能使用既有真实 Route A 写路径。
该入口的能力、未启动记录和现场授权必须与默认桌面的零 I/O 启动证据分开保存。

## 兼容性说明

| 项目 | V9 处理 |
|---|---|
| Route ID | 新增固定物理 Route ID；历史 control_route=windows_pcan_ds_bvm 继续识别为 Route A |
| SCHEME_A/B/C | 保留为闭环职责档案，不能作为新物理路线名称 |
| 会话数据 | route 仅为可选元数据；旧 manifest/control record 和 16 列 CSV 不改 |
| 真实写入口 | Python .sdo.download 仍只允许位于 dsbvm_direct.py |
| V8 | 原有 V8 固件候选与证据保持原样 |
| 候选 EXE | 根目录固定为 MotionAcquisitionV9.exe、MotionAcquisitionV9SilController.exe、MotionAcquisitionV9Direct.exe；第三者为可选回退，不随默认桌面启动 |
| 可选 transport | pyserial/pymodbus 随候选包含；Route B/C 未经完整配置、证据与授权仍 fail-closed |

## 历史发布前门（当前状态以本页开头和构建证据记录为准）

1. V9 定向测试和全量回归都有实际零失败结果；
2. GNU 与 Keil 候选的实际构建、符号/内存、哈希和写权限审计已归档；
3. Route C 的本地官方 ESI 审计已实际生成来源说明、输入 hash、JSON、Markdown 与 SHA256SUMS，且未发生网络/EtherCAT I/O；
4. Route A 正式历史会话的 checksum、完整性、回放和分析已复核；
5. 发布候选已在源码目录外启动，且无原始数据夹带、固定盘符/用户名、自动端点打开或 helper 自动启动；
6. 三个根目录 EXE、pyserial/pymodbus、默认桌面零 I/O 与 Direct EXE 的未启动/不自动连接边界均已实际核验；
7. 依赖清单、源文件清单、发布物 hash、已附加测试/firmware/ESI 证据 hash 和缺陷表均已填写；
8. 验证过程没有访问任何真实硬件。

若任一门未满足，版本只能称为“V9.0.0 实施中/待离线验证”，不能称为 READY_FOR_LAB_TEST。

## 历史发布证据填写栏（当前状态以本页开头为准）

| 项目 | 待填写内容 |
|---|---|
| 桌面端候选路径 | [待填] |
| 控制器候选路径 | [待填] |
| 源文件清单及 SHA-256 | [待填] |
| 依赖清单及 SHA-256 | [待填] |
| 发布物清单及 SHA-256 | [待填] |
| 测试和构建证据 | [待填] |
| Route C ESI 输入/产物证据 | [待填] |
| 三 EXE 与 optional transport 证据 | [待填] |
| 已知缺陷/现场待确认项 | [待填] |
| 最终离线裁决 | [待填：仅可为 READY_FOR_LAB_TEST 或待离线验证] |
