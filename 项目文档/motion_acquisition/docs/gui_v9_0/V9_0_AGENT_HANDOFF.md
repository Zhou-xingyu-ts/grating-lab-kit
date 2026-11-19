# 给下一个 Agent 的交接：Motion Acquisition 当前真实状态

本文件不是给用户写的过程记录，**是给下一个接手的 agent 的事实交接**。目的只有一个：避免重做一遍已经做过的调查，避免基于过时文档得出错误结论。

- 调查日期：2026-09-17
- 调查方式：只读（文件读取、哈希、`pytest`）。**未接任何硬件**，未启动 `MotionAcquisitionV9Direct.exe`，未修改任何源码、数据或发布物。
- 唯一写入：本文件，以及 `src/motion_acquisition/desktop/__pycache__/__main__.cpython-312.pyc`（跑测试产生的字节码缓存）。

---

## 0. 最重要的一条：现有文档已经过时

`V9_0_DEFECT_LEDGER.md` 的 `V9-OPEN-001` 写着 **257 passed / 0 failed**，最后更新时间 **2026-08-09**。

**当前实测不是这个状态。** 实测（2026-09-17）：

```
258 passed, 1 failed in 27.66s        （259 collected）
```

全库搜索 `docs/gui_v9_0/`：`259` 出现 0 次、`OneDrive` 0 次、`autocrlf` 0 次、`git_commit` 0 次、`KEIL_BUILD_REPORT` 0 次。

**结论：不要相信 `docs/gui_v9_0/`、`AGENTS.md`、`CLAUDE.md`、`task_plan.md`、`progress.md`、`findings.md` 里任何关于"当前测试状态 / 当前代码状态"的陈述。它们停在 2026-08-09。它们记录的硬件边界和安全规则仍然有效且必须遵守。**

---

## 1. 当前红灯（唯一失败项）

```
FAILED tests/test_v8_keil_project.py::test_native_build_report_closes_offline_acceptance
```

### 机理

`tests/test_v8_keil_project.py:252-268` 取 `controller_firmware/firmware_sources.json` 的 15 个共享 C 源，**对活文件重新算 SHA256**，再与 `build/V8_Keil_Native_20260801/KEIL_BUILD_REPORT.json` 里冻结的聚合值比较。

### 实测漂移（逐个文件比对得出）

| 文件 | 记录值(08-01) | 当前值 |
|---|---|---|
| `controller_firmware/src/core.c` | `77BA5F55…8B36AB` | `70E8522C…793C8F` |
| `controller_firmware/embedded/embedded_protocol_encode.c` | `86053393…B860610` | `BEBF4C01…DA8632A` |

两份文件 `LastWriteTime` = `CreationTime` = **2026-08-17 00:23**，即被 08-17 的 V9 固件工作**重新创建**。报告生成于 2026-08-01。

### 关键判断

**不要重跑 Keil 构建。** 重建等于用今天的源码覆盖 08-01 的 V8 证据，违反 `AGENTS.md` 的"证据原样保留"。

**正确修法**：新增 `controller_firmware/v8_frozen_sources.json`（固化 08-01 报告的 15×`{path,sha256}`），把测试改为对**快照**重算聚合值。这样 V8 证据一字节不动，且断言从"恒真"变成"真的在读文件"——现在的写法是报告与它自己的 `files` 列表比较，等于没测。

### 顺带发现（不是失败项）

`controller_firmware/firmware_sources.json` 的 `shared_contract` **不含任何哈希字段**（只有 `c_sources` 路径列表和编译参数），所以哈希只有 `KEIL_BUILD_REPORT.json` 一处来源。

---

## 2. 版本控制现状（与 `AGENTS.md:62` 冲突）

- `motion_acquisition/` 自身不是 git 仓库（`git rev-parse` → `fatal: not a git repository`）。
- 父工作区 `光栅尺运动控制调研\.git` 存在，但**是一个 0 条目的空目录**，不是可用仓库。`git -C <父目录> log` 同样报 `not a git repository`。
- `AGENTS.md:62` 写着「不得初始化 Git」。**但用户 2026-09-17 已明确同意使用 Git**，所以这条规则需要先被修改，而不是被绕过。

### 建仓时必须同时做的三件事（否则会污染校验和）

```
本机 git 全局配置：core.autocrlf = true      ← 危险
项目内 .gitattributes：不存在
```

`autocrlf=true` 会在检出时改写换行符，而 `data/*/checksums.sha256` 和 `release/*/SHA256SUMS.txt` 是这个项目的权威校验。一旦被改写，校验和失配，且**看起来像数据被篡改**。

必须：① 仓库级 `git config core.autocrlf false`；② 写 `.gitattributes` 标记 `*.csv *.hex *.elf *.bin *.pdf` 等为 `binary`；③ 先改 `AGENTS.md:62`。

### 建仓的实际规模（已实测，不是 2.9 GB）

`.gitignore` 已排除 `build/ release/ data/ logs/ output/ .venv/`。**实际会跟踪 380 个文件、6.62 MB。** 最大的是 `controller_firmware/third_party/.../stm32h743xx.h`（2.03 MB）。

### Git 补不回的东西

那 17 个已漂移文件（见第 3 节）在建仓前就已改变，**改了什么、为什么，Git 永远不知道**。不要承诺能回溯。

本机 git 身份：`user.name = Zhou-xingyu-ts`、`user.email = 3314087390@qq.com`（推送到远程会进历史）。

---

## 3. 量化漂移（对照 08-09 发布物的源码账本）

对照 `release/MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03/SOURCE_SHA256SUMS.txt`（201 条）：

**184 未变 / 17 已变 / 0 缺失**

变更的 17 个：

```
controller_firmware/v9/route_b_canopen_drive.c
controller_firmware/v9/route_b_canopen_drive.h
controller_firmware/v9/route_b_port_contract.c
controller_firmware/v9/route_b_port_contract.h
docs/gui_v9_0/ROUTE_B_STM32_LAB_ENTRY.md
docs/gui_v9_0/V9_0_BUILD_RELEASE_EVIDENCE.md
docs/gui_v9_0/V9_0_CLAIM_EVIDENCE_MATRIX.md
docs/gui_v9_0/V9_0_DEFECT_LEDGER.md
docs/gui_v9_0/V9_0_OFFLINE_VERIFICATION_REPORT.md
docs/gui_v9_0/V9_0_RELEASE_NOTES.md
src/motion_acquisition/control/models.py
src/motion_acquisition/desktop/main_window.py
src/motion_acquisition/desktop/route_manager_workspace.py
src/motion_acquisition/object_dictionary.py
src/motion_acquisition/routes/route_b.py
tests/test_route_b_v9.py
tests/test_route_manager_workspace_v9.py
```

另有 `build/v9_readonly_armgnu_20260817/`（2026-08-17 00:55）包含新的 `.elf/.hex/.bin/.map`，**从未被打包进任何 release**。

注意第 1 节的两个漂移文件**不在**这份 201 条账本里——说明该账本是固件专属子集，不能当作全量源码账本使用。

---

## 4. 数据现状（原文与派生）

### 论文数据：20 个真实硬件会话，完整、有哈希、未被污染

`build/Joint_Experiment_Analysis_20260801/analysis_manifest.json` 的 `raw_source_sha256` 列全 20 个会话，`all_assertions_pass: true`。

| 队列 | 数量 | 位置 | 采集方式 |
|---|---|---|---|
| V0.2 | 14 | `logs/motion_sdo_20260712_*.csv` + `.manifest.json` | CLI，2026-07-12 |
| V7 | 6 | `data/20260801/2026/08/*/samples.csv` | `motion-acq-desktop real`，V7.0.0 冻结版 |

V7 六个会话的 manifest 全部记录 `command.line = "motion-acq-desktop real"`、`software_version = 7.0.0`、`python_version = 3.12.2`。

`build/Field_Verification_20260801/BASELINE_RELEASE_VERIFICATION.md` 证明该采集路径对应**冻结的 V7 源码**（77 条目 0 missing / 0 changed）。

**实验条件记录在 `configs/joint_analysis_20260801.json`**：每个会话有 `role`（`valid_dynamic` / `stationary_baseline` / `no_motion_observed` / …）、描述、以及每段的 `speed_rpm / accel / round_trip`。

### 论文数据不包含任何模拟数据

`data/20260801` 的 `position_actual` 是 `-10632` 这类真实编码器计数。

### 两个模拟器会话不在论文里

`OneDrive\文档\MotionAcquisitionData\2026\08\{20260809_224737,20260809_224756}`：`command_line = "motion-acq-desktop demo"`，样本是 `2, 5, 2, 5, 0` 这类模拟器值。含真实事件记录（2 条 / 1 条）。**它们不在 `configs/joint_analysis_20260801.json` 里**，不属于论文数据。

### 常见误判（已排除）

`data/20260801` 的 10 个会话 `events.jsonl` 全是 0 字节。这**不是缺陷**——它们是 V7.0.0 产物，而 V7 会话本来就没人点过人工事件按钮。事件功能在 9.0.0 实测可用（见上段）。`session_integrity.py:233-236` 只在事件文件**缺失**时告警，所以 0 字节文件合法通过。

### 分析运行环境

`THESIS_ANALYSIS_REPORT.md` 记录复现命令用 `D:\Python\python.exe`（已实测存在，Python 3.12.2 + pandas 2.2.3 + numpy 2.5.1 + matplotlib 3.8.4），**不是项目 `.venv`**。两者都是 3.12.2，但依赖是分开装的。

---

## 5. OneDrive 的真实影响（结论：很小）

`data_root` 默认来自 `QStandardPaths.DocumentsLocation`（`user_settings.py:40-42`），而 Windows 已把"文档"重定向到 `C:\Users\alberic2024\OneDrive\文档`（注册表 `User Shell Folders\Personal` 实测确认）。OneDrive 客户端在运行（PID 22180）。

**并且**存在持久化设置覆盖默认值：

```
C:\Users\alberic2024\AppData\Local\MotionAcquisition\Motion Acquisition\settings.json
  "data_root": "C:\\Users\\alberic2024\\OneDrive\\文档\\MotionAcquisitionData"
```

**所以只改 `default_data_root()` 在这台机器上完全无效。**

源码中**没有任何** OneDrive / 云同步路径检测（全库搜索无匹配）。

**但影响很小，不要夸大**：论文数据 V0.2 在 `logs/`、V7 在 `data/`，**都在项目内**。OneDrive 里只有那 2 个模拟器会话。**不需要搬任何数据。** 要做的只是让**以后的**采集不默认落云盘。

另一个潜在问题：`load_user_settings()` 在解析失败时 `return defaults`（`user_settings.py:77-78`），即 `settings.json` 损坏会让 `data_root` **静默切到另一个目录**。

---

## 6. 已被核实为「正常」的项（不要再当缺陷调查）

| 项 | 核实结论 | 依据 |
|---|---|---|
| 人工事件记录链路 | **正常**。缓冲创建并 `start()`，采集中启用按钮，服务排空并写入 | `main_window.py:1829-1830`、`1915`；`acquisition_service.py:137-139` |
| `_record_event` 的 `except RuntimeError` | 写入 `diagnostics` 面板，**不是静默丢弃** | `main_window.py:1729-1731` |
| `current_git_commit()` | 工作正常，只是**永远返回 `None`**（此处没有 git 仓库） | `manifest.py:169-179` |
| 测试套件速度 | **27.66 秒**。曾因 `pytest -q` 不实时输出 + 600 s 超时被误读为"死锁"，不是死锁 | 实测 |
| OneDrive 会话的 `events.jsonl` | 非空（453 B / 214 B），事件功能正常 | 实测 |
| 数据完整性 | 20 个源文件 SHA256 全部匹配，无缺失 | `analysis_manifest.json` |

---

## 7. 用户背景（决定优先级，不要忽略）

- 这是一位研究生**一个人的课题工具**，用于采集测数据。不是产品，没有团队，没有部署。
- **开发基本由 Codex 完成**，用户对细节掌握较弱，明确说过"掌控程度较弱""很多细节不清楚"。
- 用户**不要给人看的长文档**（原话：写了也不会看）。
- **论文数据由 Codex 用 computer use 自动采集**，用户只负责接线。
- 因此：**"再增加工程严格性"不是价值，"减少理解负担"才是价值。** 任何行动前先问：这会让用户更理解自己的数据和工具，还是只是让仓库更"正规"？

### 已知的过度工程化证据

源码 26,950 行（`src/` 88 个 `.py`）、测试 54 个文件 / 259 条、`main_window.py` 单文件 **2899 行**、文档约 **400 KB**（`findings.md` 165.7 KB + `task_plan.md` 112.5 KB + `progress.md` 96.6 KB）+ `docs/` 73 个文件 / 330 KB。

对一个单人课题工具，这个体量与"用户不理解自己的软件"直接相关。

### 论文真正依赖的部分（改动时务必区分）

**改动以下内容会直接影响论文数据，必须最高谨慎：**

```
configs/joint_analysis_20260801.json         论文实验条件台账
scripts/joint_experiment_analysis.py         分析脚本（41.5 KB）
logs/motion_sdo_20260712_*.csv (+.manifest)  V0.2 源数据，14 组
data/20260801/2026/08/*/                     V7 源数据，6 组 + checksums
build/Joint_Experiment_Analysis_20260801/    论文图表与指标产物
release/MotionAcquisition/                   V7 冻结发布物（采集时用的 EXE）
```

**其余大部分（`control/`、`routes/`、`controller_firmware/`、`packaging/`、`release/MotionAcquisitionV9_*`、`build/v9_*`、`build/v8_*`）是未被论文使用的工程体系。**

---

## 8. 已知缺口（尚未解决，不是"已核实正常"）

1. **实验条件来源未记录**。运动条件（1000 counts、100 RPM、A/D/J=300）是**事后标在 `configs/joint_analysis_20260801.json` 里的**，采集时的 manifest 不记录伺服参数，也不记录谁下的运动指令。`v7-sysid-01/02` 的配置里明确写着 `nominal speed per segment not independently recorded`。这站得住（报告已声明 observational / not causal），但和"采集时记录"不是一回事。
2. **测试基线未重设**。259 条是否仍是应然数量、哪些测试已随代码演进失去意义，未评估。
3. **`build/` 有 36 个子目录**，其中 `v9_offline_completion_20260809_01..27` 是 27 个序号目录，绝大多数是中间产物。精简方案未定。
4. **发布物未与源码对齐**。见第 3 节，且 08-17 的新固件构建从未打包。

---

## 9. 覆盖边界（2026-09-18 更新：缺口已补齐）

**2026-09-18 已补**：下表标"未读"的 `control/`、`routes/`、`controller_firmware/`、`packaging/`、`tests/`、`build/v9_offline_completion_*` 已做**结构与依赖层面审查**（含实测 import 图、模块规模、死代码定位、写 API 收口核验、发布物对源码的漂移核对）。结论落在 **`V9_0_CODE_MAP.md`，该文件现在是代码结构的权威入口**。下表仅作历史记录保留，不再作为覆盖声明。

**原调查（2026-09-17）当时的状态：**

**已详细读过**：`manifest.py`、`session_store.py`（前 393 行）、`events.py`、`session_integrity.py`（全 873 行）、`session_checksums.py`（部分）、`user_settings.py`（全 90 行）、`acquisition_service.py`、`main_window.py`（约 500/2899 行，关键路径）、`test_v8_keil_project.py`、`test_v1_integrity_delivery.py`、`test_manifest.py`、`test_acquisition_service.py`、`configs/joint_analysis_20260801.json`、`build/Joint_Experiment_Analysis_20260801/THESIS_ANALYSIS_REPORT.md` + `analysis_manifest.json` + `SESSION_INVENTORY.csv`、`build/Field_Verification_20260801/BASELINE_RELEASE_VERIFICATION.md`、`docs/gui_v9_0/V9_0_DEFECT_LEDGER.md`、`AGENTS.md`、`data/README.md`。

**只做了文件级/哈希级清点、未读内容**：

| 区域 | 体量 | 状态 |
|---|---|---|
| `src/motion_acquisition/control/` | 约 5000 行（`simulator.py` 1283、`models.py` 1156、`remote_adapter.py` 753） | 未读 |
| `src/motion_acquisition/routes/route_b.py` | 1426 行 | 未读 |
| `controller_firmware/` | 110 文件 / 4.4 MB | 未读 |
| `packaging/` | 21 文件 / 155 KB | 未读 |
| `docs/` | 73 文件 / 330 KB | 未读（除缺陷账本） |
| `tests/` | 54 文件 / 276 KB | 只读了 4 个 |
| `build/v9_offline_completion_20260809_01..27` | 27 个目录 | 未读 |
| `release/MotionAcquisitionV9_0_0_OfflineCandidate_20260809_01/02` | 各约 225 MB | 未读 |

**按文件数估算覆盖约 1/3**，但**论文依赖的部分已被完整覆盖**，且未覆盖部分不属于论文临界路径（见第 7 节）。**这是一个声明，不是保证。** 后续 agent 若要动第 7 节列出的论文临界文件，必须自己重新核实。

---

## 10. 未决事项（需要用户决定，不要自行假设）

| ID | 事项 | 状态（2026-09-18 更新，详见本节末补充） |
|---|---|---|
| D1 | 是否初始化 Git | **已执行**（`core.autocrlf false` + `.gitattributes` + `AGENTS.md` 已改） |
| D2 | OneDrive 处理范围 | **只做了一半**：哈希账本已停止生成，但 `data_root` 仍指向 OneDrive（代码默认值 + 持久化设置都没改），新采集仍落 OneDrive |
| D3 | 实验条件台账 | 未决；在 `docs/` 增一页说明每个会话运动条件的来源（伺服面板 / 人工记录 / 事后判读） |
| D4 | `build/` 精简 | 未决；`AGENTS.md:64` 要求保留被规则引用的构建目录 |
| D5 | 第 1 节红灯的取舍 | **已定**（不回滚、不重跑 Keil；改为固化 V8 源码快照，证据见本节末补充） |
| D6 | Direct EXE 的位置 | 未决；实测为 8 份 / 4 个不同构建，来源是 08-09 的四轮打包，不是意外产物。是否合并需用户点头 |

### 2026-09-18 补充：D1/D5 已由证据闭合；D2 只做了一半；D3/D4 与新的 D6 仍在

本节由 2026-09-18 的只读取证补充（数据来源：Codex rollout 历史

`~/.codex/sessions/**`、`release/MotionControllerFirmwareV8/source/` 冻结副本、`release/*03/` 发布说明）。

**D5（第 1 节红灯）——不需要在「承认推进」和「回滚两文件」之间做取舍，因为回滚会拆掉已落地的只读采集。**

用 `release/MotionControllerFirmwareV8/source/controller_firmware/` 这份**冻结副本**逐字节比对，08-17 的改动是 Route B 只读采集的功能增量，不是误改：

| 文件 | 08-17 实际改动 |
|---|---|
| `src/core.c` | `mc_core_tick()` 内新增 6 行，把 CANopen 采样字段拷进 axis 快照：`statusword` / `mode_display` / `identity_vendor_id` / `identity_product_code` / `identity_revision` / `identity_serial` |
| `embedded/embedded_protocol_encode.c` | 新增 `write_axes()`；capabilities 改为按 `axis_mask` 输出 `axes`、按运行时诊断输出 `real_hardware_access` / `board_mapping_available`、`nominal_state_rate_hz` 改为动态；axis 快照新增 `statusword` / `mode_display` / `identity{...}` 三个字段 |

两处都服务于「暴露 Z 轴、真实声明 capability、输出 0x6041/0x6061/0x6064 与 identity」这一既定目标。V8 的冻结 HEX（`build/V8_Keil_Native_20260801/`，08-01 21:42）本体未被触碰，仍在原处。

**结论**：V8 固件证据 = 08-01 二进制 + 报告，已冻结且完整；活的共享源码确实前进了一步。所以正确修法仍是第 1 节那条：新增 `controller_firmware/v8_frozen_sources.json` 固化 08-01 报告的 15×`{path,sha256}`，测试改为对快照重算聚合值。**禁止重跑 Keil、禁止回滚那 6 行/那些字段。**

**D2（OneDrive）——09-17 那轮只做了一半，写入路径根本没改。本节 2026-09-18 先行记录时曾误写为"已执行"，现更正。**

09-17 那轮实际做的是**移除哈希账本**（新会话不再生成 `checksums.sha256`、manifest 不再有 `integrity` 段；11 个历史账本独立复核仍为 `ledger=verified / integrity=verified`）。它解决的是"少写文件"，不是"写到哪"。

2026-09-18 只读核实：

- 代码层无任何云路径防护 —— `src/motion_acquisition/user_settings.py:40-42` 的 `default_data_root()` 仍是 `QStandardPaths.DocumentsLocation / "MotionAcquisitionData"`，没有 OneDrive / 云同步判断。
- 本机 `DocumentsLocation` 已被重定向到 OneDrive，且**已有持久化覆盖**：`%LOCALAPPDATA%\MotionAcquisition\Motion Acquisition\settings.json` 里 `"data_root"` = `C:\Users\alberic2024\OneDrive\文档\MotionAcquisitionData`。

**结论：新采集会话今天仍会写进 OneDrive。** 修复必须同时改代码默认值和那条持久化设置（只改代码在这台机器上无效，09-16 已查明），且落点需用户先定（`%LOCALAPPDATA%` 还是 D 盘）。

**D1（Git）**——已完成：仓库级 `core.autocrlf=false`、`.gitattributes`、`AGENTS.md` 的 Git 段已改；跟踪 379 文件，`data/`、`logs/`、`release/`、`build/` 零入库；工作区干净。

**D6（新，尚未决定）——`MotionAcquisitionV9Direct.exe` 的实际分布与 09-16 审阅的表述不符，需要重新表述。**

实测全树共 **8 份**，不是 15 份；且 4 份在 `build/` 的打包中间目录里，只有 3 份在 `release/*/` 根目录，另有 1 份在 `build/..._18/pyinstaller-dist/MotionAcquisitionV9/` 子目录。哈希显示这是 **4 个不同构建**（`42D09035…`、`0A6D69D9…`、`3F48F5B8…`、`5C2F2E79…`），分别是 08-09 的 4 轮打包迭代，每轮「build 中间目录 + release 候选」各留一份。`5C2F2E79…` 就是 `release/..._03/` 里那一份，与 08-09 发布记录中的哈希一致。

**放在 `release/*/` 根目录是当时的设计，不是疏漏**：Codex 在 08-09 的打包讨论中曾提出隔离为 `optional_hardware/MotionAcquisitionV9DirectOptional/`，但最终采用的方案是「根目录三 EXE + 说明文件」，并随包交付了 `README_DIRECT_HARDWARE_V9.txt`（明确写「不要启动，除非用户已授权该确切硬件动作」）和 `README_FIRST.txt` 第 6 条。

**因此真正待决的只有**：是否把 3 份 release 副本和 4 份 build 副本合并成一处（用户 09-16 提过的 `_LAB_ONLY/`）。这要动 `release/` 与 `build/` 的文件位置，按 `AGENTS.md` 不得手工编辑，须用户明确点头后再做；**不要**以「这 15 份是意外产生的」为由处理。

**D3（实验条件台账）** 与 **D4（`build/` 精简）** 状态不变，仍未决。

---

## 11. 硬性禁止（继承 `AGENTS.md`，不得放宽）

- 不得枚举、打开或连接 CAN / 串口 / USB CDC / ST-LINK VCP / GM1 / STM32 / EtherCAT / 驱动器 / 网络端点。
- 不得发送 NMT / SDO / PDO / SYNC / Modbus / 运动命令。
- 不得烧录、复位、ST-LINK/SWD 连接（2026-08-09 的 Gate 1 授权已执行完毕，后续须重新授权）。
- 不得手工编辑 `release/`；不得移动、改名或修改 `data/`、`logs/` 既有会话。
- `.sdo.download` 只能出现在 `src/motion_acquisition/dsbvm_direct.py`。
- 不得用删除测试、改黄金数据或 `skip` 的方式让红灯变绿。
