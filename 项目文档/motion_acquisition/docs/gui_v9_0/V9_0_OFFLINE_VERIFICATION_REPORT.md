# Motion Acquisition V9.0.0 离线验证报告（离线软件就绪；候选外部启动已核验）

> 本页以 2026-08-09 的最新离线证据为准。下方保留的旧模板和早期失败记录仅供追溯，不是当前状态。

## 当前离线证据与结论

| 范围 | 实际证据 | 结论 |
|---|---|---|
| 全量 Python 回归 | **257 passed / 0 failed / 0 skipped，40.00 s**；最终执行记录见 `build/v9_offline_completion_20260809_23/FINAL_PYTEST_EXECUTION.md` | 通过 |
| 定向静态 + ESI 回归 | **31 passed，1.26 s** | 通过 |
| Route B 三 lane 构建 | `build/v9_offline_completion_20260809_15/`：Host CTest 非空且零失败；Arm GNU target 与 Keil target 均构建成功 | 通过；未烧录、未连接设备 |
| Route C 官方 ESI 审计 | `build/v9_offline_completion_20260809_22/`：输入 SHA-256 `92733290d5adc3fad608b78b174043123cfae44c2c9d834a574c8b10010cb69d`；JSON、Markdown 与 SHA256SUMS 齐全 | 通过；本地只读、无网络/EtherCAT I/O |
| 最终离线候选外部工作目录验证 | `build/v9_offline_completion_20260809_27/FINAL_CANDIDATE_LAUNCH_VERIFICATION.md`：`MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03` 的默认 GUI 启动/关闭、Direct 窗口启动/关闭（均未点击连接）、SIL localhost protocol 4.0 hello/connect/poll/disconnect 均 PASS；候选清单 415 条 SHA-256 通过，未发现固定路径、原始 data、logs 或 CSV | 通过；无真实端点 I/O |

本批所有上述离线证据均为 `real_hardware_access=false`、`device_write_capability=false`：未打开、枚举或连接 CAN、串口、USB、ST-LINK、网络、STM32、GM1、EtherCAT、驱动器或电机；未烧录、接线、扫描或发送真实命令。

当前结论为 `READY_FOR_LAB_TEST`，仅表示 V9 离线软件证据就绪，不表示任何真实硬件、STM32、CANopen、EtherCAT 或现场控制通过。外部工作目录验证覆盖最终离线候选 `MotionAcquisitionV9_0_0_OfflineCandidate_20260809_03`。STM32 板型/收发器/FDCAN 与总线电气、GM1 工程与 Modbus 映射、DS-BVM PDO/SYNC/DC/identity，以及反馈、STO/E-stop、制动、限位和机械安全仍须另行现场确认。

## 历史模板与早期证据（非当前状态）

### 0. 2026-08-09 早期 Python 回归记录

- 执行环境：项目 `.venv`，Python 3.12.2，pytest 9.1.1；未安装或升级依赖。
- 获批定向命令：

  ```powershell
  .\.venv\Scripts\python.exe -m pytest -q tests/test_routes_v9.py tests/test_route_b_v9.py tests/test_route_c_v9.py tests/test_route_manager_workspace_v9.py tests/test_direct_session_data.py tests/test_desktop_gui.py tests/test_v1_portability.py
  ```

- 实际结果：退出码 1，**74 passed / 1 failed / 0 skipped，16.87 s**。
  失败为 `tests/test_route_b_v9.py::test_route_b_unconfirmed_command_keeps_manager_resource_lock`；在 runtime 重启后 `adapter.poll()` 抛出 `ControllerDisconnected`。
- 复现命令仅运行该失败用例，结果仍为退出码 1、1 failed、0.84 s。
- 该结果保留为失败证据。其后以最小源码处置保留 `ControllerDisconnected`，使
  outstanding command 变为 `unknown`、不可重发，并使 shutdown 保留共享主站锁。
- 修复后 Route B 文件完成 **20 passed / 0.80 s**；计划的七文件定向集完成
  **77 passed / 10.76 s**；全量 `.\.venv\Scripts\python.exe -m pytest -q` 完成
  **244 passed / 36.84 s**，均为零失败。
- 已授权的 V9 portable build 创建了新的失败证据目录，并成功生成静态
  source-contract 报告（SHA-256：
  `897C2FD138D6F8C59C383124B2E2EC427444500F17E142EC9DFA145B9122BE15`）。GNU
  CMake configure 随后退出码 1：`arm-none-eabi-gcc` 被用于 Windows 主机 CTest，
  其 linker 不接受 Windows `.exe` 自检参数。GNU 编译/CTest 与 Keil 未运行。
- Python 和构建批次均未打开、枚举或连接真实 CAN、串口、USB、ST-LINK、网络、
  STM32、GM1、EtherCAT、驱动器或电机；未烧录、接线、扫描或发送真实命令。受保护
  会话、配置和历史发布物均未修改；失败构建目录保留，不复用、不删除。
- 上述 244-pass Python 与 source-contract 记录发生在本次正式 ESI 产物入口、三 EXE
  打包改造和 pyserial/pymodbus 候选包含要求之前。新增源码与测试尚未执行，不能被旧
  回归或旧静态报告覆盖；Route C 输入原件也尚未由新入口读取。

当前结论仍为：

~~~text
待离线验证完成；不得进入现场结论。
~~~

### 1. 历史基本信息模板

| 项目 | 记录 |
|---|---|
| 验证日期 | [待填] |
| 执行人 | [待填] |
| 源码基线/哈希 | [待填] |
| Python/OS/虚拟环境 | [待填] |
| GNU 工具链版本 | [待填] |
| Keil 工具链版本 | [待填] |
| 验证范围 | [待填：模块、测试文件、构建/发布物] |
| 排除范围 | 真实 CAN、串口、USB、ST-LINK、网络、GM1、STM32、EtherCAT、驱动器和运动 |

#### 历史硬件访问声明模板

| 项目 | 结果 |
|---|---|
| real_hardware_access | [待填；离线验证应为 false] |
| device_write_capability | [待填；离线验证应为 false] |
| 是否打开或枚举物理端点 | [待填；离线验证应为否] |
| 是否烧录、接线、扫描或发送真实命令 | [待填；离线验证应为否] |

### 2. 历史阶段结果模板

| 阶段 | 验收目标 | 定向测试/检查证据 | 全量回归证据 | 结果 |
|---|---|---|---|---|
| 1. 规则与兼容基线 | V9 版本、原始数据哈希、历史会话和发布基线未被改动 | [待填] | [待填] | [待填] |
| 2. 路线模型 | 注册唯一性、lazy factory、配置校验、状态机、资源互斥、零 I/O 副作用 | [待填] | [待填] | [待填] |
| 3. Route A | Fake 写序列、共享 client、Halt→Disable、会话兼容、Qt worker | [待填] | [待填] | [待填] |
| 4. Route B host | protocol 4.0 分帧、partial write、超时、重连、boot ID、命令对账 | [待填] | [待填] | [待填] |
| 5. Route B firmware | portable C、Virtual DS-BVM、协议一致性、GNU/Keil、写权限审计 | [待填] | [待填] | [待填] |
| 6. Route C | Mock Modbus、字序、负数 counts、故障 fail-closed、ESI 审计 | [待填] | [待填] | [待填] |
| 7. GUI 与会话 | 路线切换、按钮门禁、unknown、route metadata、旧会话回放 | [待填] | [待填] | [待填] |
| 8. 发布验证 | 源码外启动、无固定盘符/用户名、无原始数据夹带、无自动 I/O | [待填] | [待填] | [待填] |

“通过”必须附可复现命令与输出摘要；“未执行”不允许用通过、N/A 或 skip 掩盖。

### 3. 历史关键证据清单模板

#### 3.1 路线模型与 GUI

| 证据 | 路径/命令/哈希 | 结果 | 备注 |
|---|---|---|---|
| Route descriptor/state/factory 测试 | [待填] | [待填] | 路线列举不得打开端点 |
| 单一主站资源锁测试 | [待填] | [待填] | unknown 阻止切换 |
| GUI 离线门禁测试 | [待填] | [待填] | 真实连接/运行不可用 |
| 旧闭环职责档案兼容测试 | [待填] | [待填] | SCHEME_* 不作为物理路线 |

#### 3.2 Route A

| 证据 | 路径/命令/哈希 | 结果 | 备注 |
|---|---|---|---|
| Fake SDO 序列与既有实现逐项对比 | [待填] | [待填] | 只比较离线 Fake |
| 正常/主动停止/异常收尾 | [待填] | [待填] | 必须为 Halt→Disable |
| 2026-08-01 正式会话 checksum | [待填] | [待填] | 原始字节不得变化 |
| 16 列 CSV、完整性、回放/分析 | [待填] | [待填] | 不改列或解释 |
| 独立 motion-acq-direct 回归 | [待填] | [待填] | 不打开 PCAN |

#### 3.3 Route B

| 证据 | 路径/命令/哈希 | 结果 | 备注 |
|---|---|---|---|
| FramedByteTransport/serial 惰性导入 | [待填] | [待填] | 不打开 COM |
| protocol 4.0 双向一致性 | [待填] | [待填] | 分片、重复、陈旧 ACK、重启 |
| Virtual DS-BVM 帧级场景 | [待填] | [待填] | boot-up/heartbeat/abort/EMCY/PPM/PDO/SYNC |
| safe image 配置审计 | [待填] | [待填] | locked、未分配、port unavailable |
| GNU 构建与哈希 | [待填] | [待填] | 未烧录 |
| Keil 构建与哈希 | [待填] | [待填] | 未烧录 |
| 写权限静态审计 | [待填] | [待填] | 不能产生真实可写端口 |

#### 3.4 Route C

| 证据 | 路径/命令/哈希 | 结果 | 备注 |
|---|---|---|---|
| Mock GM1 正常握手与状态轮询 | [待填] | [待填] | 不开网络端点 |
| 高字在前/负数 raw counts | [待填] | [待填] | 显式字序断言 |
| 超时、断线、陈旧心跳、重复命令 | [待填] | [待填] | 必须 fail-closed |
| ESI/XML 审计报告 | [待填] | [待填] | InitCmd/PDO/DC/未闭合项 |
| ESI 产物完整性 | [待填] | [待填] | 来源说明、输入 hash、JSON、Markdown、SHA256SUMS；不联网/不扫描 |
| Windows EtherCAT master 依赖检查 | [待填] | [待填] | 不得新增 |

### 4. 历史数据与发布物模板

构建、依赖、源文件和发布物的逐项清单请填写在
[构建与发布证据记录](V9_0_BUILD_RELEASE_EVIDENCE.md)；本节只记录其最终核验结果。

| 检查 | 证据 | 结果 |
|---|---|---|
| 原始 data/ 会话没有字节变化 | [待填：checksums 对账] | [待填] |
| logs/ 与历史控制记录保持可读 | [待填] | [待填] |
| 16 列 samples.csv 不变 | [待填] | [待填] |
| 发布候选无原始数据夹带 | [待填：文件清单] | [待填] |
| 发布候选无固定用户名/盘符 | [待填] | [待填] |
| 源码目录外可启动 | [待填] | [待填] |
| 启动不自动打开 CAN/串口/网络/帮助进程 | [待填] | [待填] |
| 三个根目录 EXE 存在且发布物 hash 可核验 | [待填] | [待填] |
| 默认 Desktop 零 I/O 与 Direct EXE 未启动/不自动连接分别观察 | [待填] | [待填] |
| pyserial/pymodbus bundle/import 清单 | [待填] | [待填] |
| 附加测试/firmware/ESI 证据目录 hash 二次核验 | [待填] | [待填] |
| 发布物、依赖清单、源文件清单哈希 | [待填] | [待填] |

### 5. 历史缺陷、限制与现场待确认项模板

关联 [缺陷与未闭合项](V9_0_DEFECT_LEDGER.md)，在此列出本轮实际未关闭项：

- [待填：精确 STM32 板型、收发器、FDCAN 引脚/时钟与总线电气]
- [待填：GM1 当前工程、Modbus 映射、EtherCAT 设备 identity]
- [待填：DS-BVM identity、PDO/SYNC/DC/对象能力]
- [待填：反馈、STO/E-stop、制动、限位与机械安全门]
- [待填：其余测试或构建失败证据]

### 6. 历史结论模板

只有在第 2～4 节的全部必填项有可复查证据、全量回归零失败、无阻断缺陷、无真实硬件访问，并且第 5 节已清晰列出现场缺口时，才可填写：

~~~text
READY_FOR_LAB_TEST
~~~

否则填写：

~~~text
待离线验证完成；不得进入现场结论。
~~~
