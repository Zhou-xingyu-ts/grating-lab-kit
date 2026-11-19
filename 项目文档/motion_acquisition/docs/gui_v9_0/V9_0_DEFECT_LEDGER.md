# Motion Acquisition V9.0.0 缺陷与未闭合项

<!-- last_reviewed: 2026-10-23 -->

## 当前修复：2026-10-23（20261023-ux2）

本轮修复以下 6 项复查问题，并新增回放双图/完整 PNG、回到开头及轮次时间显示、Direct 目录/实验名记忆。验证使用 Fake SDO、临时 CSV 和离线 GUI；不连接设备、不修改历史会话、不新增固件或现场结论。最终全量与实际 EXE 验证统一以工程 `CLAUDE.md` 为准。

| ID | 原问题与修复 | 回归入口 |
|---|---|---|
| REVIEW-20261023-01 | 预采集/驻留/最终采集漏查状态异常；每轮有效状态字执行现有异常检查，阻止后续命令并进入 Halt→Disable | `tests/test_dsbvm_direct.py`：三类异常 × 三阶段，及合法未使能状态 |
| REVIEW-20261023-02 | 主窗口实时/回放/比较跳过失败点而误连；改为 NaN + finite，比较抽点保留断点及恢复后的最后有效值，统计口径不变 | `tests/test_replay_ux.py`、`tests/test_analysis.py` |
| REVIEW-20261023-03 | 旧分析忙时新会话摘要丢失；保留最新待分析会话，避免旧结果覆盖，取消/关闭清理待处理项 | `tests/test_replay_ux.py`：A 忙时 B/C、取消及关闭 |
| REVIEW-20261023-04 | 回放使用实时采集的最后成功时间；独立维护回放状态，切换和跳转后按目标位置重建 | `tests/test_replay_ux.py`：前后跳转及实时值隔离 |
| REVIEW-20261023-05 | 受管直控沿用过期预检起点；运行前刷新同一 client 的快照，用于起点、profile、manifest 和最终偏差；失败时保留正确的关闭/未知状态 | `tests/test_routes_v9.py`、`tests/test_direct_control_window.py` |
| REVIEW-20261023-06 | 独立 Direct 外部目录只能回放、不能索引分析；只读查看窗加载后索引并选中当前 CSV，扫描按钮沿用该目录 | `tests/test_replay_ux.py`、`tests/test_desktop_package.py` |

Direct 偏好单独存储，不覆盖主窗口配置，也不恢复预检/连接/使能；写入失败不阻断关闭收尾。PNG 导出只画已记录对象，明确缺失字段，包含名称、时间和 counts；不会按运动计划伪造数据。相应回归在 `tests/test_user_settings.py`、`tests/test_direct_control_window.py` 和 `tests/test_replay_ux.py`。包的 smoke 在设备 I/O 阻断器下实际加载合成 CSV、生成摘要、导出 PNG 并关闭。

## 前轮代码审核与修复（2026-10-22）

范围：自用开题上位机的 Route A 直控、停止/退出、CANopen 读取、会话落盘与离线指标路径。核对本地 canopen 2.4.1 实现后，已修复以下 7 项源码缺陷；下列行号是修复前的定位，触发与影响描述的是原行为。验证使用离线 Fake、offscreen GUI 和只读历史 CSV，不访问硬件，不代表现场复现或放行。历史 EXE 和分析报告原样保留；当前桌面修订、包及启动核验见工程 `CLAUDE.md`。

### 本轮修复与验证

| 条目 | 实际修复 | 回归入口 |
|---|---|---|
| 01 | 内部单次尝试设为 1，连接时即设置超时；直控逐对象检查停止 | `tests/test_canopen_retry.py`；`test_stop_between_objects_does_not_wait_for_remaining_reads` |
| 02 | 保存日志首次 I/O 错误，停止路径继续执行并回读 Disable，错误仍进入运行结果 | `test_log_io_failure_cannot_block_halt_disable`（write/flush × 正常/停止路径） |
| 03 | 收尾 I/O 失败仍停机；worker 增加带使能状态的运行失败信号，GUI 退出运行态、保留关闭/恢复入口 | `test_finalization_error_still_disables`；`tests/test_direct_control_window.py` |
| 04 | 时间相关计算保留 NaN 对齐，极值等统计单独过滤 | `test_metrics_keep_missing_samples_aligned_and_include_dwell`（正向/反向及全缺测） |
| 05 | 最终读取/保持使能/收尾检查停止，GUI 接收终态时补处理已提交请求；受管入口仍委托原 Halt→Disable | `test_stop_during_successful_finalization_disables`；GUI 终态交接与受管 Route A 测试 |
| 06 | 预检清理未确认时保留连接和未知状态，允许独立窗口重试 Disable | `test_preflight_unknown_keeps_connection_and_allows_disable_retry` |
| 07 | 超调窗口含驻留期，其他指标保留原有窗口 | `test_existing_sysid_dwell_overshoot_is_not_reported_as_zero`（24 counts）及合成序列 |

验证命令（工程目录）：`.venv\Scripts\python.exe -m pytest -q tests/test_canopen_retry.py tests/test_dsbvm_direct.py tests/test_direct_control_window.py tests/test_offline_analysis.py` 首轮 44 passed；补充受管收尾入口和 GUI 终态交接定向 2 passed。最终全量与工作区检查结果见下方签核。

采集已成功 finalize 后才收到停止时，采集 outcome 仍为 completed；随后停止是否成功由返回的 phase/servo_enabled 表达，避免把已落盘的完成记录改写成中途停止。落盘本身失败会报告错误，不声称文件完整。真实通信仍有有界超时等待，软件停止不是物理急停。

验证签核（2026-10-23）：7 项修复及桌面体验改进的最新全量 `.venv\Scripts\python.exe -m pytest -q` 结果统一记录在工程 `CLAUDE.md`，未删除或跳过测试。输入为 Fake/离线替身、临时目录、只读历史会话；`real_hardware_access=false`，`device_write_capability=false`。新桌面包独立验证，未烧录、未发 CAN 或运动命令。7 项记为 FIXED_OFFLINE，不等于硬件验收；缺测与分段超调修复属于源码离线分析管线，不能等同于 GUI 基础摘要。

### REVIEW-20261022-01 · P1：MAX_RETRIES=0 导致底层超时无限重试 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/canopen_client.py:107-111`。
- 触发：一次 SDO 读取时设备持续不返回响应；首次读取会把底层 `node.sdo.MAX_RETRIES` 写成 0。
- 证据：本地 canopen 2.4.1 的 `sdo/client.py:86-101` 从该值初始化计数，异常后先减 1，再用 `if not retries_left` 决定退出。0 会变成 -1、-2……，因此永远到不了退出条件；发送失败的循环也有同样计数方式。
- 影响：上层配置的 3 次尝试无法接管，读取函数不返回；直控线程无法再次检查 StopToken，停止/关闭可一直等待。此前“零读取失败”的成功数据没有覆盖这个条件。
- 最小修正：按该库语义把内部单次尝试设为 1，上层保留有限重试；不要靠修改第三方库绕过。修复时用无真实总线的超时替身验证请求次数有界，再检查采集循环在对象之间/重试之间响应取消，避免恢复有限超时后仍整轮等待。

### REVIEW-20261022-02 · P1：日志写入异常会阻断 Halt/Disable · FIXED_OFFLINE

- 位置：`src/motion_acquisition/dsbvm_direct.py:447-450`、`:760-775`，关联 `set_phase` 和 `_log`。
- 触发：运动期间控制日志的 write/flush 失败，例如磁盘空间耗尽或存储设备 I/O 错误。
- 证据：异常收尾先调用 `_phase(STOPPING)`，它立即写日志；写入失败便直接进入异常分支，尚未调用 `halt_and_disable()`。该函数本身也在保护 Disable 的 try/finally **之前**调用 `set_phase(STOPPING)`，同样能被日志异常提前打断。状态回读随后也经过日志，不能恢复这一过程。
- 影响：通信仍可用时，也可能因为本地日志失败而未尝试发送 Halt/Disable，使驱动器保留之前状态。
- 最小修正：停止路径不能以日志成功为前提，确保日志失败时仍尝试原有 Halt→Disable 并回读；保存错误并报告状态未知，不吞掉故障或伪报已关闭。用会抛 OSError 的日志替身验证实际调用顺序。

### REVIEW-20261022-03 · P1：运行异常只有错误文字，GUI 不结束运行状态 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/desktop/direct_control_window.py:208-213`、`:551-556`。
- 触发：点击开始后，重新预检抛异常，或 service.run 在日志关闭/会话 finalize 时抛异常。这些均由 worker 的外层 except 接住。
- 证据：worker 仅发送 operation_failed 并清理自己的 `_running`；GUI 的 `_running` 已在 `_start_run()` 置 True，却只有 `_run_finished()` 才清零。`_operation_failed()` 在 GUI 仍处于运行时只更新文字，不清理状态或同步 worker 保留的使能未知状态。
- 影响：GUI 认为还在运动，Disable 按钮不可用；停止只设置已无任务读取的 StopToken。关闭窗口走 `_request_stop()` 后返回，且以后没有 run_finished 触发 shutdown，窗口无法正常退出。若异常发生在正常运动完成后的落盘环节，驱动器可能仍使能。
- 最小修正：区分运行终止异常与普通操作失败，让运行异常发出终态/使能状态结果；GUI 结束运行状态，并在未知使能时保留 Disable/关闭收尾能力。不应把所有 operation_failed 都无条件标成 Servo Off。

### REVIEW-20261022-04 · P2：缺测过滤破坏数值与时间一一对应 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/offline_analysis/metrics.py:223-226`、`:289`、`:297-299`。
- 触发：运动段内某次 following_error 或 position_actual 读取失败，宽表在该对象上出现 NaN；其他对象仍可读，时间行保留。
- 证据：numeric() 单独删掉数值中的 NaN，ramp_times 却保留原长。following_error 缺测时，detrended_rms 的联合有限值掩码可能因长度不等抛 ValueError；position_actual 缺测时，rise_time 用压缩后的位置索引查未压缩的时间，会静默错配。
- 手算示例（未执行）：时间 [0,0.1,0.2,0.3,0.4]，位置 [0,20,NaN,50,100]，目标 100。真实已观测的 10%→90% 跨越为 0.4−0.1=0.3 s；当前压缩后按索引得到 0.3−0.1=0.2 s。
- 最小修正：时间相关计算保留原长 NaN 数组，或对时间与数值使用同一个掩码；无足够数据时返回缺失值。用包含中间缺测的短序列验证，不改历史原始数据。

### REVIEW-20261022-05 · P1：正常收尾期间的停止请求可能被丢弃 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/dsbvm_direct.py:641-647`，关联 `hold_enabled()` 和独立窗口的 `_request_stop()`、`_run_finished()`。
- 触发：最后一次 capture 已结束，用户在最终位置读取、保持使能或会话收尾期间点击停止。
- 证据：此时 GUI 仍处于 running，停止按钮只设置 StopToken；service 在最终 capture 之后没有再检查该 token，`hold_enabled()` 的下载与状态等待也没有传入 token。成功返回后 GUI 清掉 token，报告 COMPLETED—SERVO ON，没有执行这次停止所承诺的 Halt→Disable。
- 影响：用户已点击停止，但驱动器仍保持使能；该问题不以通信或磁盘故障为前提。
- 最小修正：取消请求应覆盖整个运行收尾及终态交接；控制层检查/传递 token，GUI 或 worker 在处理成功终态时仍需处理已提交的停止，不能只在最终位置读取前补一次检查。用受控阻塞的最终读取/收尾替身检查这一时序。

### REVIEW-20261022-06 · P1：预检失败时丢弃未确认关闭的驱动器状态 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/desktop/direct_control_window.py:119-128`，关联 `_close_client()`。
- 触发：独立直控窗口连接到已使能驱动器，预检中的 Halt/Disable 失败，异常清理阶段的状态回读或再次关闭也失败。
- 证据：清理阶段的异常被 `except Exception: pass` 忽略，随后 `_close_client()` 清空 client/controller 并把 `_servo_enabled` 设为 False。GUI 只收到 preflight_failed，既没有未知使能状态，也没有可用的 Disable 恢复入口；之后 shutdown 可直接结束。
- 影响：尚未确认 Disable 的驱动器可能仍使能，但软件已丢掉连接及状态。断开通信不能证明 Servo Off。
- 最小修正：在已有控制器且关闭未确认时保留未知状态和恢复路径，报告关闭失败并允许重试 Disable；普通的未建立连接错误仍按原逻辑处理。不要把所有连接失败都升级成此状态。

### REVIEW-20261022-07 · P2：超调量漏掉指令到位后的响应 · FIXED_OFFLINE

- 位置：`src/motion_acquisition/offline_analysis/metrics.py:241-246`，关联 ramp/dwell 分段。
- 证据：`overshoot_counts` 只对 ramp 内的 position_actual 取极值；指令停止变化后、下一段运动前的 dwell 被排除，恰好漏掉可能滞后出现的实际位置峰值。
- 已有数据核对：`build/offline_analysis_20260918/tables/METRICS_BY_SEGMENT.csv` 中 v7-sysid-01 第 5 段报告超调 0 counts；对应 `MOTION_SEGMENTS.csv` 的目标为 994，ramp 结束于 267.960506 s，本段结束于 272.566977 s。原始 `data/20260801/2026/08/20260801_205522_run-dbc20599/samples.csv` 在 sample_seq=36112、268.390867 s 记录 position_actual=1018、position_demand=994，读取状态均为 ok，因此该段实际至少存在 24 counts 超调。
- 验证边界：审查时只读核对 CSV；修复回归通过指标函数重算该会话确认 24 counts，没有修改任何原始数据/历史报告，也没有运行完整报告生成管线。不是声称所有会话的超调均算错。
- 最小修正：超调极值窗口覆盖本次运动开始至下一次运动前的响应，保留上升时间等指标各自的时间定义；增加峰值出现在驻留期的短序列检查。修复后重新生成新的分析产物，保留历史记录。

### 次要已知问题与审核边界

- 独立直控窗口 `direct_control_window.py:371-373` 仍默认使用历史 data/20260801，与主窗口本地数据根不一致；修正默认路径即可，不需目录重构。
- 本次在正常往返、使能/握手停止和 Disable 失败的原有回归之上，补充真实 canopen 库配离线网络替身、日志 I/O 故障、GUI 异常/交接及指标缺测/驻留峰值检查。未扩展硬件或研究功能。
- 本次不审核 STM32/GM1 固件全量实现，不把模块规模、命名、没有 CI 等列作缺陷。上述条目优先于功能扩展；运行源代码与发布包的结果需分别验证。

## 历史验证快照（2026-08-09，不作为本轮修复证据）

| ID | 类别 | 当前事实 | 状态 |
|---|---|---|---|
| V9-OPEN-001 | Python 回归 | 最新全量结果为 **257 passed / 0 failed / 0 skipped，40.00 s**；定向静态 + ESI 为 **31 passed，1.26 s** | CLOSED（离线回归） |
| V9-OPEN-002 / 003 | A/B/C 离线源码行为 | 由上述零失败全量 Python 回归覆盖；不包含任何真实端点访问 | CLOSED（离线回归） |
| V9-OPEN-004 | Route B portable 构建 | `build/v9_offline_completion_20260809_15/` 的 Host CTest 非空且零失败，Arm GNU 与 Keil target 均成功 | CLOSED（离线构建；未烧录/未上板） |
| V9-OPEN-007 | Route C ESI 审计 | `build/v9_offline_completion_20260809_22/` 已保存 JSON、Markdown、SHA256SUMS；输入 SHA-256 为 `92733290d5adc3fad608b78b174043123cfae44c2c9d834a574c8b10010cb69d` | CLOSED（本地只读审计） |
| V9-OPEN-008 | 最终发布候选启动 | `build/v9_offline_completion_20260809_27/FINAL_CANDIDATE_LAUNCH_VERIFICATION.md` 记录 `MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 的默认 GUI、未连接的 Direct 窗口及 SIL localhost protocol 4.0 hello/connect/poll/disconnect 均 PASS；415 条候选清单 SHA-256 通过且无固定路径、原始 data、logs 或 CSV | CLOSED（最终离线候选外部工作目录验证） |
| V9-OPEN-005 | Route B 硬件 | H743 板型、收发器、FDCAN/时钟、终端与 CAN 电气未现场证实 | OPEN（现场） |
| V9-OPEN-006 | Route C 硬件 | GM1 当前工程、Modbus 映射、identity 与 EtherCAT 实配未现场证实 | OPEN（现场） |
| V9-OPEN-009 | 现场安全 | DS-BVM PDO/SYNC/DC/identity、反馈、STO/E-stop、制动、限位和机械安全未现场证实 | OPEN（现场） |

历史 74 passed / 1 Route B failure 已修复并保留为追溯证据。上表已关闭项只代表当时对应的离线验证结果；当时状态为 `READY_FOR_LAB_TEST`、候选为 `MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03`。它们不关闭 2026-10-22 新发现的代码缺陷，也不代表当前源码或实际发布 EXE 已通过本次审核。

## 使用规则

本表记录 V9 离线候选的缺陷、证据缺口和现场前置条件。它不以删除条目、降低严重度、修改黄金数据或 skip 测试作为关闭方式。修复后必须留下定向测试、全量回归和证据位置。

“OPEN”表示尚无关闭证据，不等于已发生现场故障。“硬件证据缺口”不等于软件缺陷，但会阻止相应现场路线进入下一门。

## 历史未闭合项快照（已由当前账本替代）

| ID | 类别 | 描述 | 影响 | 当前状态 | 关闭所需证据 |
|---|---|---|---|---|---|
| V9-OPEN-001 | 离线验证 | 历史 74 passed / 1 stable failure 已保留。修复后 Route B 文件 20 passed / 0.80 s、七文件定向集 77 passed / 10.76 s、全量 Python 244 passed / 36.84 s，均零失败 | Python 回归门已关闭；不覆盖构建/发布/硬件门 | CLOSED (PYTHON) | 三份命令与输出已记录；后续仅按构建/发布证据门推进 |
| V9-OPEN-002 | Route A 兼容 | Route Manager 委托 Route A 后的 Fake SDO、共享 client、Halt→Disable、独立直控入口和历史会话兼容尚待复核 | 阻止 Route A V9 集成声明 | OPEN | 对应测试结果、正式会话 checksum/回放记录 |
| V9-OPEN-003 | Route B host | 串口惰性导入、protocol 4.0 分帧/重启/unknown 对账和默认 Offline 门禁尚待离线证据 | 阻止 Route B host 候选声明 | OPEN | Fake transport/协议回归和无端口打开证据 |
| V9-OPEN-004 | Route B firmware | 静态 source-contract 报告已生成；GNU CMake configure 因 ARM 交叉编译器承担 Windows 主机 CTest 而失败，GNU 编译/CTest 和 Keil 未执行 | 阻止 Route B 固件候选声明 | BLOCKED | 由用户选择/提供主机 CTest 编译器，或明确修订验证设计后在全新目录重启双工具链构建 |
| V9-OPEN-005 | Route B 硬件 | 精确 H743 板型、收发器、引脚、时钟、电源、终端和总线电气未证实 | 阻止烧录后 FDCAN/总线阶段 | OPEN | 受控现场记录与相关安全放行证据 |
| V9-OPEN-006 | Route C 契约 | 当前 GM1 工程、gm1_route_v1 实际寄存器映射、build ID 和 endpoint 未证实 | 真实 Route C 必须 unavailable | OPEN | GM1 工程导出、契约评审和身份/映射证据 |
| V9-OPEN-007 | Route C ESI | JSON/Markdown/SHA256 的本地只读审计入口已写入源码，但尚无对官方原件的实际新产物、输入 hash 或回归结果 | 阻止 Route C 离线证据完整性声明 | OPEN | 只读审计报告、来源说明、输入原件 hash、回归结果 |
| V9-OPEN-008 | 发布验证 | 三 EXE、pyserial/pymodbus、源码目录外启动、依赖/文件清单、执行证据附加、发布物 hash 和无自动 I/O 记录尚待实际执行 | 阻止离线候选发布结论 | OPEN | 发布验证报告、三 EXE/transport 清单、附加证据与产物 hash |
| V9-OPEN-009 | 现场安全 | DS-BVM identity、PDO/SYNC/DC、反馈、STO/E-stop、制动、限位和机械放行仍待现场证实 | 阻止任何真实控制/运动结论 | OPEN | 父工作区放行/验收/缺口记录 |

## 历史静态审查修复记录（不等于当前账本）

下列修复已完成源码级复核并由本次通过的 Python 回归覆盖；这不代替
GNU/Keil、发布或现场证据，仍不能写成“离线候选完成”或硬件通过。

| ID | 静态发现与修复 | 关联回归 | 状态 |
|---|---|---|---|
| V9-SRC-001 | Route B 命令状态查询、陈旧 ACK、boot 变化或普通 poll/event 交换失败都会使路线进入 unknown，并保留资源锁；poll/event 失败还会将所有 outstanding command 逐个转为 unknown，原异常继续抛出，不得自动重发。 | `tests/test_route_b_v9.py` | SOURCE_FIXED_VERIFIED_PYTHON |
| V9-SRC-002 | Virtual DS-BVM 必须处于 NMT Operational 才能接受虚拟 PPM；Stop/Pre-operational 清除待执行目标，避免后续重进 Operational 误运动。 | `tests/test_route_b_v9.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-003 | Route C 预检实际验证 Mock GM1 handshake；断线、重启、magic/heartbeat/boot 异常使路线 unknown 并保留 GM1 锁。 | `tests/test_route_c_v9.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-004 | route metadata 只接受真实 bool；B/C/A 离线记录走只读边界，显式写意图仍要求精确授权。预检记录还要求 passed、locked/read_only 和 false/false。 | `tests/test_routes_v9.py`、`tests/test_direct_session_data.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-005 | GUI 仅在用户显式保存时写入无样本离线预检记录；列表、创建、预检不落盘也不探测端点。 | `tests/test_route_manager_workspace_v9.py`、`tests/test_routes_v9.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-006 | 2026-08-01 Route A 受保护会话的完整、失败但结构完整、失败且边界不完整分类已成为只读黄金回归。 | `tests/test_direct_session_data.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-007 | A/B/C 的公开 ActiveRoute 构造均复用或执行完整离线 policy/config 门；Route A 无 live delegate，B 拒绝 serial-shaped 配置，C 拒绝非 Mock transport、auto-connect/discover、Windows master、scan/SafeOp/OP、非有限 heartbeat 和歧义 identity。 | `tests/test_routes_v9.py`、`tests/test_route_b_v9.py`、`tests/test_route_c_v9.py` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-008 | V9 打包说明准确区分默认 Demo/离线 Route Manager、虚拟 NMT/SDO 行为与历史页面经用户显式 real+连接后的只读 SDO upload；最终候选要求保留 pyserial/pymodbus，但其路径仍 lazy/fail-closed，且 Direct EXE 不随默认桌面启动。 | `tests/test_v1_portability.py`、`packaging/README_FIRST.txt`、`packaging/motion_acquisition_v9.spec` | SOURCE_FIXED_PENDING_EXECUTION |
| V9-SRC-009 | 实际定向失败发现 restart-poll 的测试未预期 `ControllerDisconnected`，且 adapter 未将 outstanding command 置 unknown。修复保留异常、阻止重发，并补 event-drain 同类回归；历史 74/1 结果不覆盖。 | `tests/test_route_b_v9.py` | SOURCE_FIXED_VERIFIED_PYTHON |

| V9-SRC-010 | Route B shutdown 遇到已接受但尚未查询最终状态的命令时，必须保留共享主站资源锁；不得将该路线标记为已关闭或允许切换。74 passed / 1 failed 是本修正之前的历史结果。 | `tests/test_route_b_v9.py::test_route_b_shutdown_with_unqueried_commands_keeps_manager_resource_lock` | SOURCE_FIXED_VERIFIED_PYTHON |
| V9-SRC-011 | Route C ESI 产物入口要求显式只读确认、来源说明、全新输出目录和可选输入 hash；生成 JSON/Markdown/SHA256，并在发布附加时二次核验账本。它不实现 Windows EtherCAT、GM1 TCP、扫描或状态切换。 | `tests/test_route_c_v9.py`、`packaging/write_v9_route_c_esi_audit.ps1`、`packaging/package_v9_release.ps1` | SOURCE_FIXED_PENDING_EXECUTION |

## 事实性限制（不作为可“修复”的软件缺陷）

| ID | 限制 | 当前可用表述 |
|---|---|---|
| V9-LIMIT-001 | Route A 的历史现场基线是固定单轴、固定 profile 且带限制的结果 | 可引用历史 PASS_WITH_LIMITATIONS；不可外推为 B/C、三轴、实时或安全验证 |
| V9-LIMIT-002 | V8 固件包和既有证据受保护 | V9 可引用但不覆盖、改名或反向解释为已上板 |
| V9-LIMIT-003 | 16 列 samples.csv、raw counts、checksums 和历史会话不可修改 | 新路线只能新增可选 route 元数据和关联记录 |
| V9-LIMIT-004 | Windows 在 Route C 中不是 EtherCAT 主站 | 只能以 Modbus TCP 管理 GM1；不得引入 Windows EtherCAT master |

## 关闭记录模板

每个条目关闭时补充以下内容，而不是删除原条目：

| 字段 | 内容 |
|---|---|
| 关闭日期/执行人 | [待填] |
| 相关源码或配置 | [待填] |
| 定向测试 | [待填：命令、输入、结果] |
| 全量回归 | [待填：命令、实际通过数、零失败] |
| 证据路径及 SHA-256 | [待填] |
| 是否真实硬件访问 | [待填；V9 离线关闭必须为否] |
| 残余限制 | [待填] |

## 历史裁决模板（不再适用当前离线证据）

- 任一 V9-OPEN-001、002、003、004、007 或 008 未关闭时，不得写 READY_FOR_LAB_TEST。
- V9-OPEN-005、006、009 可以作为明确列出的“现场待确认项”存在于 READY_FOR_LAB_TEST 结论中，但不能被误写为已验证。
- unknown、超时、断线、身份不匹配或安全状态不明必须保留为故障/缺口，禁止自动降级为成功。
