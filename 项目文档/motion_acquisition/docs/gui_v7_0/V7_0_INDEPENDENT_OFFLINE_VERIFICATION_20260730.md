# Motion Acquisition V7.0.0 独立本机离线验证报告

日期：2026-07-30～2026-07-31  
验证对象：Motion Acquisition / Motion Controller Firmware V7.0.0  
验证性质：本机、离线、软件质量验证与加固，不是 V8.0 开发或现场验收

## 1. 总裁决

V7.0.0 在当前 Windows 电脑上的离线软件闭环为
**PASS_WITH_LIMITATIONS**。

本轮从冻结的源码、历史数据和既有发布物出发，独立重跑 Python、Qt、
native C、embedded C、ARM 交叉构建、历史会话和打包后黑盒验证。共确认
并修复 9 类真实软件缺陷；每类缺陷均保留修复前失败证据、根因、最小修复
和永久回归测试。

当前没有未解决的阻断级或高严重度本机软件缺陷。以下项目没有被本轮证明：

- STM32H743 板上启动、时钟、中断、fault handler 和长期运行；
- 真实 PC transport、FDCAN、收发器、终端电阻和三驱动器通信；
- 真实 Node-ID、EDS、对象字典、SDO 时序、heartbeat 和 EMCY；
- 真实反馈、三轴任务、周期、同步、运动、精度和鲁棒性；
- E-stop、STO、制动器、限位、硬件 watchdog 和任何安全等级；
- 第二台 Windows 电脑或干净虚拟机部署。

这些事项的裁决是 `BLOCKED_BY_HARDWARE` 或
`PASS_WITH_LIMITATIONS`，不得由 SIL、host CTest、交叉编译或本机复制
启动替代。

## 2. 冻结起点

本轮开始时冻结了三组逐文件 SHA-256：

| 对象 | 文件数 | 字节数 | 聚合清单 SHA-256 |
|---|---:|---:|---|
| 产品源码 | 276 | 5,513,839 | `0AC4DC4C56096F825FCA8B947C658335236D4459A23F1093CDA3C23169FD0947` |
| 既有 Windows 发布物 | 663 | 186,280,442 | `2123E491C8A561CC342BECD68AD1DD6D0F97FC5C3AED48BA0E53291C2F2D37E9` |
| 可访问历史数据 | 60 | 28,366,826 | `2197AD91950CC532B442585485E8982A76FD2A6FE331B4090E8D0F91100E2D0C` |

清单和环境记录位于
`build/V7_Independent_Verification_20260730/`。本目录没有可用 Git
历史，因此本轮使用逐文件基线、修复证据和最终产物清单建立可追溯性。

验证环境为 Windows 10.0.26200、PowerShell 7.6.3、Python 3.12.2、
本机已有 CMake/Ninja/Zig/Arm GNU 工具链。用户修订离线边界后未联网、
未下载、未安装工具，也未申请管理员权限。

## 3. 真实缺陷与修复

| ID | 严重度 | 缺陷 | 根因与修复 | 永久证据 |
|---|---|---|---|---|
| V7-AUD-001 | 中 | native helper 临时目录不能正常清理 | V7 创建前缀与旧 V5 清理白名单不一致；统一为实际 V7 前缀 | `T01_*`、helper 生命周期压力测试 |
| V7-AUD-002 | 当前中、未来高 | host 可接受陈旧但格式正确的 ACK/status | 缺少 command、payload 和 connection 三重关联；在 adapter 边界强制校验 | `T02_*` |
| V7-AUD-003 | 中 | malformed ready JSON 可造成 helper/临时目录泄漏并抛出 `KeyError` | readiness 解析无大小上限且字段验证不完整；增加 64 KiB 上限、容错解析、端口范围验证和统一清理 | `T03_*` |
| V7-AUD-004 | 高 | 第一个 snapshot 前断连会留下仍活动的 recorder/service | worker 将初始连接当成普通循环状态；增加启动不变量并以 error 完整收尾 | `T04_*` |
| V7-AUD-005 | 高 | 重复 JSON 成员在 Python 与 C 中解释不同 | Python 取最后一个，C 取第一个；三种实现均拒绝重复语义成员 | `T05_*`、最小输入 `duplicate_kind.json` |
| V7-AUD-006 | 未来现场高 | CANopen manager 接受 Node-ID 128，NMT 查找可命中未分配节点 0 | 节点范围/唯一性和请求入口未共同验证；非法或重复节点保持未分配，NMT 拒绝 0 和 >127 | `T06_*` |
| V7-AUD-007 | 高 | Python/native 对重复命令、断连和连接代际的顺序不同 | 幂等判断早于连接校验，SIL 拒绝还会覆盖原终态；统一同连接重试、跨连接复用和原终态保持语义 | `T07_*` |
| V7-AUD-008 | 高 | manifest 可引用会话外文件，错误级完整性问题仍可能标为 compatible，畸形 section 可崩溃；事件直接入口还可读绝对外部路径；浏览器对已判 invalid 的统计值仍会强制转换并中断扫描 | 文件归属、顶层裁决和可选 section 缺少统一 fail-closed；增加 4 MiB manifest 上限、类型/数值约束、会话根路径约束、任一 error 即不兼容，并让分析/事件/浏览器入口复用或执行同等边界 | `T08_*`、`T08_EVENT_PATH_*`、`T08_BROWSER_*` |
| V7-AUD-009 | 低 | 发布依赖清单把整个开发/审计环境误列为产品依赖 | `pip freeze` 没有区分运行时闭包和构建环境；从 `pyproject.toml` 运行时根递归解析 marker 有效的已安装依赖，构建器另列 | `T09_*` |

所有修复保持 protocol 4.0、16 列 `samples.csv` 和历史 manifest 字段含义
不变；没有增加设备写能力。

## 4. 测试有效性

- 最终 Python 总回归：`163 passed in 23.16 s`，无 skip、xfail 或失败。
- 分支覆盖采用两个互斥分区，避免 Qt 与单进程 coverage 组合的已知挂起：
  非 GUI `149 passed`，GUI `14 passed`；合并为 9,971 statements、
  2,356 branches、80% 总覆盖。
- 覆盖率不是单独的通过依据。高风险低覆盖分支由畸形 helper、协议差分、
  生命周期故障、队列满、历史只读和打包后黑盒测试补充。
- 对 stale-connection 判断做过受控 mutation：原测试按预期失败；恢复后
  源文件 SHA-256 与变更前完全一致且测试重新通过。
- 测试断言审查没有发现无断言、无依据 xfail 或通过吞异常维持绿灯的
  高风险用例。
- 全量 coverage 在同一进程运行时两次出现 Qt 退出挂起；仅终止了审计拥有
  的 coverage/pytest 进程。普通全量 pytest 正常完成，两个互斥分区也均
  完成，因此这是审计运行方式限制，不是产品线程或 helper 残留的通过证据。

## 5. 协议、状态机和耐久结果

- Python SIL、native C 和 embedded C 对 hello、capability、ACK、
  command status、connection/boot/command ID、重复命令和断连重连语义
  已重新对齐。
- 标准和 ASan/UBSan embedded probe 均完成 7 个分片请求与 7 个响应，
  响应可由现有 Python protocol 4.0 模型解析；二者均报告
  `real_hardware_access=false`、`device_write_capability=false`。
- 确定性 embedded decoder 压力：250,000 个输入，无 sanitizer 报告。
- C11 SPSC CAN 队列压力：2,000,000 次操作，与独立参考模型逐步一致，
  覆盖满队列、回绕和交错 push/pop。
- helper 生命周期压力共 204 次，包含 4 个并发实例和不同 loopback
  端口；预热后的 Windows handle 计数为 164→164，没有新增临时目录。
- 修复前遗留的 19 个 workspace 外 helper 临时目录被列入证据。项目规则
  阻止了删除，因此它们仍存在，但修复后压力运行没有新增残留。
- ACK 仍只代表命令受理；任务完成由独立 terminal status 表示。
  protective stop 始终是非安全等级的软件请求，不是 E-stop 或 STO。

## 6. C 与固件结果

- host C 使用 `-Wall -Wextra -Wpedantic -Werror` 及 ASan/UBSan 构建，
  最终 CTest `6/6` 通过。
- 当前 Zig/Windows 目标不提供所需 libFuzzer runtime，因此没有下载替代
  工具；使用 sanitizer 确定性输入生成器和独立队列模型作为本机替代。
- 最终 H743 交叉构建使用 Arm GNU 15.2.1，生成 ELF/HEX/BIN/map。
  ELF 具有 RX Flash、RW RAM，无 RWE load segment。
- 最终独立构建资源为 text 24,996 B、data 8 B、bss 71,592 B；
  ELF SHA-256 为
  `6959E3E0BAF7127142882E85F88C3ADFD65E9EF8C1478005FEB7254246721018`。
- ELF 不含 `malloc/calloc/realloc/free/_sbrk`、stdio formatter、
  `CO_SDOclientDownload`、`CO_NMT_sendCommand` 或
  `HAL_FDCAN_AddMessageToTxFifoQ`。
- 生产 H743 绑定 unavailable ports，Node-ID 默认为未分配，物理 CAN
  transmit 不存在。虚拟测试中的受权 NMT 不能从生产 GUI 或配置启用。

交叉构建只能裁决“可构建和离线静态边界”，不能证明板上行为。

## 7. 数据、文件系统和恢复

- 当前可访问历史语料为 19 个会话、60 个冻结文件、212,378 条记录；
  19/19 保持原 16 列合同并兼容，17 个有数据会话可回放，两个
  non-replayable 会话是有意停止的空会话。
- outcome 分布为 14 legacy、2 completed、3 stopped。分析、回放和完整性
  检查前后，60 个基线文件 SHA-256 全部不变。
- 中文、空格、171 字符路径上的 Demo 会话完成后，五个源文件设为只读，
  integrity、replay、analysis 仍通过且哈希不变。
- 284 字符路径在当前 Windows 环境触发 WinError 206。服务返回明确
  `error`，未发布部分 CSV/manifest。这是已知可移植性限制，不宣称
  Windows 超长路径支持。
- raw counts 和 16 列 `samples.csv` 继续是权威记录；回放、分析和恢复
  不写原始 CSV。未标定值不自动标成 mm 或 µm。

## 8. 主张—实现—测试—发布物—裁决矩阵

| 对象/主张 | 主要实现 | 独立测试 | 发布物检查 | 裁决 |
|---|---|---|---|---|
| Windows 应用可在本机离线启动/采集/回放/分析 | `src/motion_acquisition`、desktop GUI | 全量 pytest、隔离 AppData 黑盒 smoke | onedir EXE 与依赖清单 | PASS_WITH_LIMITATIONS |
| GUI/worker/helper 生命周期正确收尾 | desktop worker、remote adapter | 初始断连、helper crash、204 生命周期、并发端口 | 打包后启动/退出/helper smoke | PASS_WITH_LIMITATIONS |
| protocol 4.0 三实现语义一致 | Python protocol、native JSON/core、embedded protocol | 差分、重复字段、7 请求 roundtrip、状态序列 | Python/native/firmware 同包 | PASS |
| Python SIL | simulator、SIL helper | 生命周期、幂等、故障注入、打包后 smoke | `MotionAcquisitionSilController.exe` | PASS |
| native C reference | portable core、host Windows port | CTest、Python 黑盒、sanitizer、生命周期 | `MotionAcquisitionControllerReference.exe` | PASS |
| portable C 三轴核心 | `controller_firmware/src/core.c` | strict CTest、mutation、长序列 | native 与 firmware source package | PASS |
| embedded C 固件逻辑 | embedded runtime/protocol/CANopen manager | ASan/UBSan host tests、decoder/queue stress | H743 ELF 和独立源码包 | PASS_WITH_LIMITATIONS |
| H743 可交叉构建 | startup/CMSIS/linker/target main | clean ARM build、ELF/map/symbol审计 | ELF/HEX/BIN/map/report | PASS |
| CANopen boot/heartbeat/EMCY/只读 SDO 逻辑 | CANopen manager/master + virtual bus | CTest、abort/timeout/非法帧/回绕 | 源码编入 H743；发送端口不可用 | PARTIALLY_VERIFIED |
| Scheme A/B/C 责任配置不改 GUI/protocol | capability/profile/port seams | SIL profiles 与协议模型测试 | 同一桌面协议与固件接口 | PARTIALLY_VERIFIED |
| 历史数据和 raw CSV 完整 | integrity/replay/analysis | 19 会话、212,378 行、60 文件哈希 | 发布包不内嵌历史数据 | PASS |
| Windows 可移植部署 | QStandardPaths、相对资源、onedir | 中文/空格/只读/隔离目录、本机复制启动 | 固定用户名/盘符/源码路径扫描 | PASS_WITH_LIMITATIONS |
| 失效关闭和零真实设备写入 | unavailable ports、能力门、无写符号 | no-write tests、source/ELF/package scan | Node-ID 未分配，物理 transmit 缺失 | PASS |
| 真实 STM32 运行 | 尚无板级证据 | 未执行 | 仅未烧录候选 | BLOCKED_BY_HARDWARE |
| 真实 CANopen 三驱动器通信 | 尚无总线/EDS/节点证据 | 未执行 | 仅 virtual bus | BLOCKED_BY_HARDWARE |
| 真实运动、周期、同步、安全和精度 | 尚无反馈/安全/运动现场证据 | 未执行 | 无现场产物 | BLOCKED_BY_HARDWARE |
| 第二台 Windows 部署 | 尚无第二台电脑 | 未执行 | 本机 onedir | PASS_WITH_LIMITATIONS |

机器可读矩阵、命令结果、缺陷前后证据和最终发布清单位于
`build/V7_Independent_Verification_20260730/`。

## 9. 发布与可追溯性

最终发布流程会重新构建 native reference、独立 H743 固件包和 Windows
PyInstaller onedir。发布目录中的 `SHA256SUMS.txt` 是最终 EXE 与固件
入口的权威哈希，固件子包另含自身完整 SHA-256 清单和构建报告。

发布包必须通过以下检查后才能交付：

- 主 GUI、SIL helper、native helper 和固件包入口全部存在且哈希匹配；
- GUI 在隔离用户目录、源码目录不可见的工作目录中启动；
- SIL/native helper 各完成一次本地协议 smoke，硬件/写能力均为假；
- 包内无历史会话、临时文件、旧网页源码或 V0～V4 GUI 文档；
- 文本和二进制扫描无固定用户名、开发盘符或源码路径；
- 固件包从所带源码独立重编译，并与发布 ELF 一致；
- 第三方依赖、CMSIS 来源和许可证随包可追溯。
- `DEPENDENCIES.txt` 只列运行时依赖闭包；`BUILD_ENVIRONMENT.txt` 单独
  记录 Python/PyInstaller 构建版本。

Windows onedir 不是逐字节可复现构建：PyInstaller/Python/Qt 元数据和构建
时间可能改变二进制哈希。本轮要求当前源码与当前产物可追溯，不虚构跨机器
bit-for-bit reproducibility。

## 10. 最终边界

本报告只证明当前电脑、当前本地依赖和离线测试条件下的软件质量。没有访问
网络、CAN、USB、STM32、驱动器或电机，没有烧录固件，没有执行任何真实
设备命令，也没有修改历史实验数据。V8.0 未开始。
