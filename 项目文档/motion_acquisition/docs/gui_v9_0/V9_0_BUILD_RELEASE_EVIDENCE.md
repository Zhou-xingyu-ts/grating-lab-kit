# Motion Acquisition V9.0.0 构建与发布证据记录（当前离线证据；候选外部启动已通过）

## 适用范围

本文件记录 V9 离线候选的实际测试、构建、依赖、源文件、数据和发布物证据。它不提供任何命令的预授权；每批测试、构建、打包、安装或脚本执行前，仍须按 AGENTS.md 取得用户明确确认。

本轮记录的硬件边界必须为：

~~~text
real_hardware_access=false
device_write_capability=false
未打开、枚举或连接真实 CAN、串口、USB、网络、STM32、GM1、EtherCAT、驱动器或电机
~~~

该声明描述本表记录的测试、构建、ESI 审计和打包**执行批次**。最终候选会包含根目录
`MotionAcquisitionV9Direct.exe` 这一可选 Route A 回退入口，但本轮不启动它；它的存在不把
`device_write_capability=false` 改写成“任何 EXE 都不可能在获授权后写设备”。默认
`MotionAcquisitionV9.exe` 的启动边界仍是零 I/O。

## 当前离线执行证据（2026-08-09）

| 范围 | 证据位置/结果 | 状态 |
|---|---|---|
| 全量 Python | **257 passed / 0 failed / 0 skipped，40.00 s**；最终记录见 `build/v9_offline_completion_20260809_23/FINAL_PYTEST_EXECUTION.md` | 通过 |
| 定向静态 + ESI | **31 passed，1.26 s** | 通过 |
| Route B 三 lane | `build/v9_offline_completion_20260809_15/`：Host CTest 非空且零失败；Arm GNU target 与 Keil target 构建成功 | 通过；未烧录、未连接设备 |
| Route C 官方 ESI | `build/v9_offline_completion_20260809_22/`：输入 SHA-256 `92733290d5adc3fad608b78b174043123cfae44c2c9d834a574c8b10010cb69d`；JSON、Markdown、SHA256SUMS 齐全 | 通过；本地只读、无网络/EtherCAT I/O |
| 最终离线候选外部启动 | `build/v9_offline_completion_20260809_27/FINAL_CANDIDATE_LAUNCH_VERIFICATION.md`：`MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 的默认 GUI、未连接 Direct 窗口和 SIL localhost protocol 4.0 hello/connect/poll/disconnect 均 PASS；415 条清单 SHA-256 通过，无固定路径、原始 data、logs 或 CSV | 通过；仅离线软件候选 |

上述结果均未访问真实硬件。离线软件证据现可支持 `READY_FOR_LAB_TEST`；该结论不表示 STM32、GM1、DS-BVM、CANopen、EtherCAT 或安全链的现场待确认项已通过。

## 历史命令接口与待填写模板（非当前状态）

| 目的 | 入口 | 必要确认与输出边界 |
|---|---|---|
| GNU/Keil 源码同源静态报告 | `packaging/write_v9_source_contract_report.ps1` | 需 `-AcknowledgeOfflineReport`；只写全新报告目录，不编译、不连接硬件 |
| GNU/CTest + Keil portable 合同构建 | `packaging/build_v9_portable_contract.ps1` | 需 `-AcknowledgeOfflineBuild` 和显式 CMake/CTest/Ninja/GNU/nm/size/Keil/fromelf 路径；只写新的 `build/` 子目录，报告的是 portable artifact section，不是 H743 镜像内存或上板证据 |
| Route C 官方 ESI 本地审计 | `packaging/write_v9_route_c_esi_audit.ps1` | 需 `-AcknowledgeLocalReadOnlyAudit`、本地原件、来源说明和新 `build/` 子目录；产生 JSON/Markdown/SHA256，不联网、不扫描、不进 SafeOp/OP |
| V9 PyInstaller 候选 | `packaging/package_v9_release.ps1` | 需 `-AcknowledgeOfflinePackaging`；拒绝复用 output/work 目录，产生三个根目录 EXE、依赖/源/发布物 hash，并可只读附加已有测试、Route B 和 ESI 证据，不启动真实端点 |

本次新增的 Route C ESI 入口、三 EXE 候选打包入口及其证据附加均未执行。先前的
source-contract/双工具链尝试及其 GNU configure 失败事实见
[离线验证报告](V9_0_OFFLINE_VERIFICATION_REPORT.md)；不得把旧输出移作本次新源码的通过证据。
任何新执行前仍须把精确命令、输入、预期输出、写入目录和无硬件 I/O 边界提交给用户确认。
Route C 原件及产物接口详见 [Route C ESI 审计入口](V9_0_ROUTE_C_ESI_AUDIT_ENTRY.md)。

## 1. 历史源码与配置清单模板

| 项目 | 路径/生成方式 | SHA-256 | 复核结果 |
|---|---|---|---|
| V9 源文件清单 | [待填] | [待填] | [待填] |
| 路线配置模板清单 | [待填] | [待填] | [待填] |
| V9 文档清单 | [待填] | [待填] | [待填] |
| 受保护的 V8/历史文件基线 | [待填] | [待填] | [待填] |

清单应明确排除 data、logs、release 和其他受保护历史物的任何改写。

## 2. 历史测试证据模板

| 范围 | 事前获批命令 | 输入 | 预期输出 | 实际输出/退出码 | 通过数/失败数 | 证据路径 |
|---|---|---|---|---|---|---|
| Route model | [待填] | [待填] | 零 I/O、状态机/锁通过 | [待填] | [待填] | [待填] |
| Route A | [待填] | Fake/回放/历史会话 | 兼容且不改数据 | [待填] | [待填] | [待填] |
| Route B host | [待填] | Fake transport | protocol 4.0 对账 | [待填] | [待填] | [待填] |
| Route B firmware | [待填] | portable/virtual fixture | 离线安全默认 | [待填] | [待填] | [待填] |
| Route C | [待填] | Mock/ESI fixture | fail-closed/只读审计 | [待填] | [待填] | [待填] |
| GUI/会话 | [待填] | Offline/SIL | 门禁与元数据 | [待填] | [待填] | [待填] |
| 全量回归 | [待填] | 工程测试集 | 零失败 | [待填] | [待填] | [待填] |

若全量回归有失败，必须关联缺陷表条目；不得用 skip、删除测试或改黄金数据伪造零失败。

## 3. 历史控制器候选构建模板

| 工具链 | 源码基线 | 事前获批命令 | 产物/符号/内存 | SHA-256 | 写权限审计 | 结果 |
|---|---|---|---|---|---|---|
| GNU | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |
| Keil | [待填] | [待填] | [待填] | [待填] | [待填] | [待填] |

两个工具链必须引用相同业务源码。构建成功不等于已烧录、上板、可通讯或可控制。

### 3.1 Route B 构建证据附加接口

经批准的 `build_v9_portable_contract.ps1 -OutputDir <fresh-root>` 成功后，发布入口只接受该
`<fresh-root>` 的固定接口，而不依赖中间测试目录名称：

- `V9_PORTABLE_DUAL_TOOLCHAIN_BUILD_REPORT.json`；
- `SHA256SUMS.txt` 与 `SOURCE_SHA256SUMS.txt`；
- `source-contract/SOURCE_SHA256SUMS.txt` 与
  `source-contract/V9_GNU_KEIL_SOURCE_CONTRACT_REPORT.json`；
- 顶层 `candidate_version=9.0.0`，以及 `hardware_boundary` 的
  `real_hardware_access`、`device_write_capability`、`endpoint_enumeration`、
  `flash_or_debug_action` 均为实际布尔 `false`。

打包时把目录作为 `-FirmwareEvidenceDir` 传入。脚本先逐项核验**原目录**的完整 hash
账本；原目录、CMake/Keil 中间物、原始日志和含本机路径的 build report 均保留在 `build/`，不
复制进候选。候选只写 `evidence/firmware/FIRMWARE_EVIDENCE_RECEIPT.json`、两份路径安全的
source-hash ledger、路径安全的 source-contract report，以及候选内新生成的 `SHA256SUMS.txt`。
receipt 记录原完整账本和原 build report 的文件名/哈希及已核验条目数；它不重新编译 Route B。

### 3.2 Route C ESI 审计证据附加接口

经批准的 ESI 审计成功后，发布入口只接受其新目录中的：

- `ROUTE_C_ESI_AUDIT.json`（`candidate_version=9.0.0`、本地只读结果、输入原件 hash）；
- `ROUTE_C_ESI_AUDIT.md`；
- `SHA256SUMS.txt`（生成 JSON/Markdown 的 hash）。

将目录作为 `-EsiAuditEvidenceDir` 传入后，打包脚本先在原目录核验账本和 JSON 的来源说明、输入
SHA-256、`source_write_performed=false` 以及六项 false 边界（含 `network_accessed`、
`ethercat_master_started` 与 `ethercat_state_transition`）。候选仅附加路径安全的 JSON、Markdown、
`ROUTE_C_ESI_EVIDENCE_RECEIPT.json` 和候选内新生成的 `SHA256SUMS.txt`；原目录不会递归复制，
脚本也不替 ESI 审计器读取 XML，更不连接 GM1/EtherCAT。

## 4. 历史数据完整性模板

| 对象 | 检查 | 结果 | 证据/哈希 |
|---|---|---|---|
| 正式 Route A 会话 | checksums.sha256 对账 | [待填] | [待填] |
| 16 列 samples.csv | 列数和原始字节/解释兼容 | [待填] | [待填] |
| 历史控制记录 | 可读与旧 profile/route 映射兼容 | [待填] | [待填] |
| 原始 data/logs | 无本轮字节修改 | [待填] | [待填] |

## 5. 历史桌面端发布候选模板

| 检查 | 实际证据 | 结果 |
|---|---|---|
| 事前获批打包命令、输入与输出路径 | [待填] | [待填] |
| 源码目录外启动 | [待填] | [待填] |
| 无固定用户名或盘符 | [待填] | [待填] |
| 无原始 data/logs 夹带 | [待填] | [待填] |
| 不自动打开 CAN/串口/网络或启动 helper | [待填] | [待填] |
| 三个根目录 EXE（Desktop/SIL/Direct）均存在且 hash 已登记 | [待填] | [待填] |
| 默认 Desktop 不启动 Direct EXE，Direct EXE 本身不自动连接 | [待填] | [待填] |
| 包含 pyserial、pymodbus 且 import/bundle 不自动打开端点 | [待填] | [待填] |
| 已附加的测试/Route B/Route C ESI 证据目录 hash 二次核验 | [待填] | [待填] |
| 依赖清单及版本 | [待填] | [待填] |
| 发布文件清单及总大小 | [待填] | [待填] |
| 发布物 SHA-256 | [待填] | [待填] |

release 目录不得手工编辑。若需要候选目录，必须使用工程既定打包流程产生，并保留输入、输出和哈希。

### 5.1 经批准后的完整候选命令接口

只有已经分别产生测试日志、Route B 新鲜构建目录和 Route C 新鲜 ESI 审计目录时，才使用
严格附加模式。以下是待批准的命令形状，不是本轮已运行命令：

```powershell
.\packaging\package_v9_release.ps1 `
  -PythonExe .\.venv\Scripts\python.exe `
  -TestEvidencePath .\build\v9_python_full_YYYYMMDD_01.log `
  -FirmwareEvidenceDir .\build\v9_portable_dual_toolchain_build_YYYYMMDD_01 `
  -EsiAuditEvidenceDir .\build\v9_route_c_esi_audit_YYYYMMDD_01 `
  -RequireExecutionEvidence `
  -OutputDir .\release\MotionAcquisitionV9_0_0_OfflineCandidate_YYYYMMDD_01 `
  -WorkDir .\build\v9_release_packaging_YYYYMMDD_01 `
  -AcknowledgeOfflinePackaging
```

输入目录或输出目录只要已经存在，脚本就会拒绝继续。成功候选的根目录应至少包含
`MotionAcquisitionV9.exe`、`MotionAcquisitionV9SilController.exe`、
`MotionAcquisitionV9Direct.exe`、`README_FIRST.txt`、`README_DIRECT_HARDWARE_V9.txt`、
`OPTIONAL_HARDWARE_TRANSPORTS.txt`、`SOURCE_SHA256SUMS.txt`、`SHA256SUMS.txt`、
`V9_RELEASE_MANIFEST.json`、`evidence/tests/`，以及经核验后附加的
`evidence/firmware/` 和 `evidence/route_c_esi/`。这些输出仍不代表任何 EXE 已启动、
任何端点已打开或 READY_FOR_LAB_TEST。

## 6. 历史最终核对模板

- [ ] 所有测试、构建和打包命令都有事前授权记录；
- [ ] 所有结果都有实际输出、退出码和哈希；
- [ ] 全量回归零失败；
- [ ] 原始会话和历史发布物没有字节变化；
- [ ] 本轮没有真实硬件 I/O；
- [ ] Route C ESI 原件来源说明、输入 SHA-256、JSON、Markdown 与账本齐全；
- [ ] 若附加 Route B/ESI/测试执行证据，原完整账本已先核验，候选仅含路径安全子集、receipt 和候选内账本；
- [ ] 默认 Desktop 的零 I/O 启动与 Direct EXE 的未启动/不自动连接边界分开记录；
- [ ] 缺陷表中不含阻止 READY_FOR_LAB_TEST 的 OPEN 项；
- [ ] 现场待确认项已明确保留，未被写成通过。

完成后，将本文件的证据编号回填至离线验证报告、声明—证据矩阵和候选发布说明。
