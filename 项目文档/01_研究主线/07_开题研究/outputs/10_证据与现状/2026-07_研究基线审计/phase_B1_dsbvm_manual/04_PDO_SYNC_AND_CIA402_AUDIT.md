# PDO、SYNC 与 CiA 402 边界审计

## 1. CANopen PDO 数量与对象结构

| 事实 ID | 结论 | 标签 | 官方文件完整路径、版本、PDF 页码、章节 / 对象 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| BUS-B1-001 | CANopen 提供 4 个 RPDO 和 4 个 TPDO | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.13、20、81–92，§3.1、§3.7.1、§6.2.13–6.2.22 | 默认标识按 Node-ID 形成；RPDO1–4 与 TPDO1–4 分别有通信参数和映射对象 | 该系列 CANopen 层有 4 收 4 发 PDO 通道 | 不证明当前 Node-ID、COB-ID、映射内容、传输类型或使能状态已经配置 |
| BUS-B1-002 | CANopen RPDO 通信参数为 `0x1400–0x1403`，映射为 `0x1600–0x1603`；TPDO 通信参数为 `0x1800–0x1803`，映射为 `0x1A00–0x1A03` | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.81–92，§6.2.13–6.2.22 | 每个 PDO 以通信参数决定 COB-ID/传输方式，以映射参数决定 payload 对象 | PDO 通信与对象映射必须分别配置 | 不证明只设置 COB-ID 就能传输目标/实际值，也不证明对象一定允许映射 |
| BUS-B1-003 | CANopen 单个 PDO 最大 8 byte；EtherCAT 对相应可配置映射允许到 40 byte | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.19、82、91，§3.7、§6.2.14、§6.2.22 | 手册明确区分两种总线的映射长度 | CANopen PDO payload 容量有限，需要选择关键对象 | 不证明总线负载、周期或所有 8 byte 都能在任意频率稳定传输 |
| BUS-B1-004 | 修改 PDO 映射须在 Pre-Operational：先把 sub0 写 0，再改映射项，最后把 sub0 写为对象数量 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.82–83、91，§6.2.14、§6.2.22 | 映射修改有明确状态与顺序要求 | PDO 配置本身是写操作，并非只读采集的自然延伸 | 不证明当前只读软件已经或可以安全执行这些写入；本轮未执行任何写操作 |
| BUS-B1-005 | Reference 对 `0x1600–0x1603` 和 `0x1A00–0x1A03` 的映射数量 sub0 默认值给为 0 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.83、91–92，§6.2.14.1、§6.2.22.1 对象表 | CANopen 可配置 PDO 的默认 payload 映射为空 | 不等于 PDO 通信对象不存在，也不证明当前实物仍保留默认值；必须回读参数确认 |
| BUS-B1-006 | RPDO transmission type 默认 255，TPDO transmission type 默认 0；TPDO inhibit time 与 event timer 默认 0 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.81–82、88–90，§6.2.13、§6.2.21 对象表 | 传输类型与定时器具有官方默认值 | 映射为空时，这些默认值不构成可用控制链；也不证明当前参数未被改过 |

## 2. Transmission Type、SYNC 与定时语义

| 事实 ID | 结论 | 标签 | 官方文件完整路径、版本、PDF 页码、章节 / 表格 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| BUS-B1-007 | PDO 可异步事件触发、SYNC 同步触发、event timer 周期触发或 remote frame 轮询触发 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.19，§3.7 `Process Data Object` | 手册列出四种 PDO 发送触发机制 | “用了 PDO”不自动等于“用了 SYNC” | 不证明任一机制已在当前台架配置，也不证明它们具有相同确定性 |
| BUS-B1-008 | SYNC 由 CANopen master 广播；只有 PDO communication parameters 配置成 synchronous 时才作用于该 PDO | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.15，§3.4 `Synchronization Telegram (SYNC)` | SYNC 是同步 PDO 的触发基准，不是选择 CSP/CSV/CST 后自动出现的内部时钟 | 不证明主站已发送 SYNC，不证明多个节点的实际采样时刻完全一致 |
| BUS-B1-009 | transmission type 0 表示在下一次 SYNC 发送一次，1–240 表示每接收 N 次 SYNC 发送一次；254/255 为事件方式 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.20–21，§3.7.1 `PDO Communication Parameters` 表 | 可以通过 transmission type 选择同步倍频/事件模式 | 不证明主站 SYNC 周期、从站响应延迟或闭环周期达到某个数值 |
| BUS-B1-010 | TPDO inhibit time 分辨率为 100 μs，但实际发送可最多再晚约 2 ms；event timer 单位为 1 ms，TPDO event 精度在约 2 ms 内 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.20–21，§3.7.1 的 inhibit/event 定时说明 | 手册对异步 TPDO 定时器给出了分辨率/误差量级 | 这不是同步控制误差保证，也不能转换成“稳定 1 kHz”；2 ms 级说明反而要求区分 event timer 与同步 PDO |

## 3. NMT 与 CiA 402 是两层状态机

| 事实 ID | 结论 | 标签 | 官方文件完整路径、版本、PDF 页码、章节 / 对象 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| BUS-B1-011 | CANopen 节点上电/复位后进入 Pre-Operational；Pre-Operational 允许 SDO、不允许 PDO；Operational 才允许 PDO；Stopped 不允许 PDO/SDO | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.13–14，§3.2 `Network Management` 状态图/表 | PDO 运行需要网络层 NMT Operational | NMT Operational 不等于电机已 Servo On 或 CiA 402 已 Operation Enabled |
| BUS-B1-012 | CiA 402 驱动状态由 `0x6040`/内部事件触发，`0x6041` 回读；必须等待当前转换完成再发新转换 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.24–27、140–145，§4.2–4.2.3、§6.2.80–6.2.81 | 网络状态和驱动功率/运动状态是独立条件 | 不证明只要进入 NMT Operational 就能运动，也不证明当前安全链/STO/限位满足 |
| BUS-B1-013 | 模式写入 `0x6060` 后应以 `0x6061` 确认实际模式 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.153–154，§6.2.94–6.2.95 | 模式请求与当前模式显示分离 | 不证明模式切换能在任意 CiA 402 状态下无条件完成；手册没有给当前台架切换流程验证 |

## 4. 关键目标/实际对象的映射能力

| 对象 | Reference Manual 结论 | 典型用途 | 标签与官方证据 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| `0x6040 Controlword` | RxPDO 可映射 | CiA 402 状态控制 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.140–143，§6.2.80 对象表 | 状态控制可进入过程数据 | 不证明当前已映射或允许写 |
| `0x6041 Statusword` | TxPDO 可映射 | 驱动状态反馈 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.143–145，§6.2.81 对象表 | 状态可周期回传 | 不证明当前 TPDO 已配置 |
| `0x6060/0x6061` | `6060` RxPDO，`6061` TxPDO | 模式请求/实际模式 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.153–154，§6.2.94–6.2.95 对象表 | 模式可通过过程数据请求并回读 | 不证明动态切换安全或当前映射存在 |
| `0x607A` | RxPDO | PPM/CSP/PRM 目标位置 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.163，§6.2.114 对象表 | 位置目标可映射 | 不证明是光栅位置环输入，也不证明周期性能 |
| `0x60FF` | RxPDO | PVM/CSV/VRM 目标速度 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.35、42、180，§5.4、§5.6、§6.2.147 对象/模式表 | 速度目标可映射 | 不证明 CSV 已配置 |
| `0x6071` | RxPDO | CST/TRM 目标转矩 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.39–41、160，§5.5、§6.2.108 对象/模式表 | 转矩目标可映射 | 不证明电流/转矩标定或安全限幅已确认 |
| `0x6064` | TxPDO | 位置实际值 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.155，§6.2.98 对象表 | 实际位置可周期回传 | 不说明双反馈中的传感器来源 |
| `0x606C` | TxPDO | 速度实际值 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.41、44、47、161，模式输出表/§6.2.111 | 实际速度可周期回传 | 不说明速度估算信号来源和滤波延迟 |
| `0x6077` | TxPDO | 转矩实际值 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.41、44、47、162，模式输出表/§6.2.113 | 实际转矩可周期回传 | 不证明它是独立转矩传感器测量值 |
| `0x6062/0x6063/0x60F4` | Reference 均标 PDO Mapping NO | demand/internal actual/following error | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.154–155、176，§6.2.96–6.2.97、§6.2.141 | 对 CANopen Reference 版本，不应把三者列入默认可 PDO 映射对象 | EtherCAT XML 对三者给出不同结论，见下一节；也不影响 SDO 读取能力 |

## 5. EtherCAT ESI 的映射与身份边界

| 事实 ID | 结论 | 标签 | 官方文件完整路径、版本、行号 / XML 节点 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| BUS-B1-014 | ESI 定义可配置 RxPDO1–4 / TxPDO1–4，以及固定 RxPDO5–10 / TxPDO5–10 | E3-EDS | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 11491–12018，`RxPdo`/`TxPdo` | 该 ESI 描述了 EtherCAT 过程数据候选布局 | 不证明 CANopen 也有 10 对 PDO，不证明该 XML 与当前实物固件精确一致 |
| BUS-B1-015 | 固定 RxPDO9 `0x1608` 包含 `6040/607A/60FF/6071/60FE:01/6060`；固定 TxPDO9 `0x1A08` 包含 `6041/6064/606C/6077/60FD/6061`，并分别分配到 SM2/SM3 | E3-EDS | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 11635–11680、11920–11965 | ESI 给出一种可覆盖 CSP/CSV/CST 的 EtherCAT 固定过程数据集合 | 不证明当前 EtherCAT master 已选择它，不证明 CANopen 映射相同 |
| BUS-B1-016 | ESI 将 `6062/6063/6064/60F4` 都标为 TxPDO 可映射，并把 `6062`、`60F4` 放入可配置 TxPDO2/4 | E3-EDS | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 10389–10426、11380–11390、11748–11784 | EtherCAT ESI 的对象映射能力比 Reference 的 CANopen 表更宽 | 不得覆盖 CANopen Reference；XML 内部设备为 WELCON、Revision `0x20200525`，当前硬件/固件匹配未验证 |
| BUS-B1-017 | ESI 的 DC 段列出 `DC-Synchron` 和 `FreeRun`；`CycleTimeSync0`/ShiftTime 默认字段为 0 | E3-EDS | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 12031–12046，`Dc/OpMode` | ESI 声明存在 EtherCAT DC 同步运行选项 | 0 不是最小周期或精度保证；不证明现场 EtherCAT 链已配置 DC |

## 6. 是否存在官方周期保证

| 周期事实 ID | 官方数值 / 字段 | 正确解释 | 标签与官方证据 | 不能推出的结论 |
|---|---|---|---|---|
| TIME-B1-001 | Digital Encoder 最大输入频率 10 MHz（四倍频前） | 反馈输入电气边沿能力 | E3-HW：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.4，§1.2 | 不是位置环、CANopen PDO、SYNC 或主站周期保证 |
| TIME-B1-002 | 模拟 Hall / Sin-Cos 采样 24 kHz | 驱动器内部模拟反馈采样指标 | E3-HW：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.4，§1.2 | 不是外部可读数据率、内部控制环频率或总线周期保证 |
| TIME-B1-003 | TPDO inhibit 100 μs 分辨率；event timer 1 ms 单位；事件发送误差可到约 2 ms | CANopen 异步 TPDO 定时器口径 | E3-REF：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.20–21，§3.7.1 | 不是 1 kHz 稳定同步闭环保证，也不是 SYNC 精度 |
| TIME-B1-004 | ESI `0x60C2 Interpolation Time Period` 默认 period=2、index=-3，即字段表达 2×10^-3 s | 某一 EtherCAT ESI 的默认插补时间设置 | E3-EDS：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 11101–11123 | 不能改写成“最大 500 Hz”或“保证 500 Hz”；它是默认参数，且 Reference 未收录该对象、设备身份未匹配 |
| TIME-B1-005 | ESI `0x1C32/0x1C33` 的 Cycle Time、Minimum Cycle Time 默认均为 0；DC `CycleTimeSync0` 为 0 | 字段存在但没有给出非零能力下限 | E3-EDS：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 4977–5184、12031–12046 | 不能把 0 解释为“任意快”或“零延迟” |

**结论：本轮官方资料中未找到明确的 1 kHz 保证，也未找到可作为保证使用的最小 PDO 周期、最大 PDO 频率、SYNC 同步误差或最小更新时间。**

## 7. 本文件的严格边界

1. PDO 和 SYNC 是官方支持能力；当前映射、NMT Operational、CiA 402 Operation Enabled 和安全链均未由手册证明为现场事实。
2. CANopen 默认可配置 PDO 映射数量为 0，因此“支持 PDO”不能写成“开箱已有目标/实际值 PDO”。
3. 循环同步模式要求周期目标，但手册没有把“周期”量化成 1 ms。
4. EtherCAT ESI 与 CANopen Reference 的对象映射差异必须按总线与版本分开处理。
