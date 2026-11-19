# DS-BVM 反馈接口审计

## 1. 型号级物理接口

| 事实 ID | 结论 | 标签 | 官方文件完整路径 | 版本、PDF 页码、章节 / 表格 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|---|
| FB-B1-001 | DS-BVM-FETC/FCAO 列出增量编码器、分离/共享 Hall、模拟 Hall、Sin/Cos、SSI/BiSS-C 串行编码器五类反馈能力 | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.4，§1.2 `Electrical Data`，`Position Sensor` / `Motor Feedback` 表 | 手册将这些类型列为位置/电机反馈，并分别给出部分电气指标 | 这些信号类型属于该型号系列的官方输入能力 | 不证明任意两类可同时闭环，不证明当前参数或当前台架正在使用其中哪类 |
| FB-B1-002 | 机身暴露三个反馈连接器：M6 Digital Encoder、M7 Analog Encoder、M8 Serial Encoder | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.5，§2.2 `Connections` | 连接器总览明确给出 M6/M7/M8 的功能名称 | 存在三类物理输入入口 | 不证明三个入口会被固件同时采纳，也不证明它们可同时参与同一个闭环 |
| FB-B1-003 | Digital Encoder 只有一组 A/B/I 差分通道，并在同一 15 针口上提供 U/V/W 三组差分 Hall | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.8，§2.5 `Digital Encoder`，J201 引脚表 | J201 为 5 V/GND、A±、B±、I±、Hall U±/V±/W±、FG | 型号级数字口只能直接看到一套 A/B/I 引脚；Hall 可分离接入 | 不证明可在该口直接接两套独立 ABZ 编码器；不证明 Hall 已连接或有效 |
| FB-B1-004 | Digital Encoder 的增量输入为 RS422 差分 A-quad-B，可带或不带 Index，四倍频前最大输入频率 10 MHz | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.4，§1.2 `Incremental Encoder` | 手册给出信号类型和 `A-quad-B Max Input Frequency 10MHz (before quadrature)` | 适配增量式光栅尺时，输出电平/差分制式和边沿率有明确硬件边界 | 10 MHz 不是位置环带宽、PDO 周期或 1 kHz 控制保证 |
| FB-B1-005 | Analog Encoder 口包含 SIN±、COS±、REF±、模拟 Hall U/V/W 和一组 AIN± | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.9，§2.6 `Analog Encoder`，J901 引脚表 | J901 的 14 个引脚按上述功能列出 | Sin/Cos、模拟 Hall 和模拟输入的物理入口存在 | 不证明 AIN± 是第二位置编码器；手册没有把它定义为仅监测位置输入或双反馈专用输入 |
| FB-B1-006 | 模拟 Hall 与 Sin/Cos 的标称采样频率均为 24 kHz | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF p.4，§1.2 `Motor Feedback` | 模拟 Hall 与 Sin/Cos 各列 `Sampling Frequency 24KHz` | 驱动器内部这两类模拟反馈的采样指标有官方数值 | 不证明外部控制器能以 24 kHz 取得数据，也不证明内部位置/速度环、PDO 或 SYNC 以 24 kHz 运行 |
| FB-B1-007 | Serial Encoder 口提供 BiSS DATA/CLK 差分线和一组 RS485 RTX 差分线；支持 SSI/BiSS-C，列出 0.5/1/2/2.5/5 Mbps | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | VER 1.0，PDF pp.4、11，§1.2、§2.8 `Serial Encoder`，J801 引脚表 | 手册列出协议、比特率和物理引脚 | DS-BVM 具备串行绝对编码器输入路径 | 不证明任意 SSI 编码器均兼容，也不证明当前固件/当前参数已经选择该接口 |
| FB-B1-008 | 官方配件线束与硬件手册的数字、模拟、串行和 CAN 引脚一致 | E3-ACC | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Motion_Controller_Accessory_Selection_Manual_CN.pdf` | Ver1.2，PDF pp.31–34（印刷 pp.28–31），第七章；P4/P5/P6/P7/P10/P11 接口定义 | P4 为 BiSS/RS485，P5/P6 为 CAN HIGH/LOW/GND，P7 为 A/B/I+Hall，P10 为 Sin/Cos+模拟 Hall+AIN，P11 为 USB | 线束层面复核了接口名称和引脚数 | 不证明线束在当前台架存在、连接正确或参数已放行 |

## 2. 固件选择与双反馈

| 事实 ID | 结论 | 标签 | 官方文件完整路径 | 版本、PDF 页码、章节 / 对象 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|---|
| FB-B1-009 | Servo Studio 明确允许选择 Single Feedback 或 Dual Feedback | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf` | VER 1.1，PDF p.16，§4.2 `Feedback Setting (Encoder)` | GUI 步骤要求在单反馈和双反馈之间选择并 Apply，必要时 Save 到 flash | 系列固件概念上不是只能选择一个 Position Sensor；存在官方双反馈模式 | 不证明每一型号、每一固件和任意物理接口组合均可双反馈，也不证明当前驱动器已设为 Dual Feedback |
| FB-B1-010 | 双反馈时 Feedback1 用于 voltage、current、speed control，Feedback2 用于 position control | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf` | VER 1.1，PDF p.16，§4.2，第 (2) 项 | 手册直接给出两路反馈的控制职责分工 | 官方双反馈是“电机侧内环反馈 + 位置环反馈”的结构，而非两路都只做显示 | 不说明 `0x6064` 必然对应哪一路，也不说明两个反馈的具体物理端口分配和内部融合算法 |
| FB-B1-011 | GUI 列出 10 类可选反馈：Incremental、Incremental+Index、Hall only、Incremental+Hall、Incremental+Index+Hall、共享 Hall/Index、Analog Hall、Sin/Cos、BiSS、SSI | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf` | VER 1.1，PDF pp.17–22，§4.2.1 (1)–(10) | 各节分别说明反馈类型和主要配置 | GUI/固件系列支持的反馈选项比 `0x4008` 表中三个值更广 | 不证明 `0x4008` 单个对象能够枚举全部 GUI 选项，也不证明当前型号所有选项均可用 |
| FB-B1-012 | 双反馈组合矩阵允许多种 Feedback1/Feedback2 组合，但 Feedback2 为 Hall Sensor 的一列均标为不可用；双反馈最多只能选择一个带 Hall 的编码器 | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf` | VER 1.1，PDF p.23，§4.2.1(11) `Dual Feedback encoder correspondence table` | 表中给出组合 O/X，并附注双反馈只能有一个含 Hall 的编码器 | 双反馈组合受到固件兼容矩阵约束，不能凭多个连接器任意组合 | 不证明矩阵中的两个 Incremental 组合在 DS-BVM 上有两组独立 A/B/I 物理引脚；这是型号级接口缺口 |
| FB-B1-013 | `0x4008 Position Sensor Type` 只公开 0x00 Incremental、0x02 Hall、0x03 Incremental & Hall 三个值 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf` | VER 1.0，PDF p.104，§6.2.41，对象 `0x4008` 表 | 对象为 sub0、UNSIGNED8、RW、不可 PDO 映射；取值如结论所列 | `0x4008` 能表达三种数字/Hall 选择 | 不证明它是 Servo Studio 全部反馈类型的完整编码表；官方资料未解释 Sin/Cos、BiSS、SSI、模拟 Hall 如何映射到此对象 |
| FB-B1-014 | `0x4009` 只给位置编码器、Hall、Hall 位状态和电角度的极性位；`0x400A` 给 Hall 极对数 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf` | VER 1.0，PDF p.105，§6.2.42–6.2.43，对象 `0x4009`、`0x400A` | `0x4009` bit0–3 分别为 position encoder inverted、hall inverted、hall bit polarity、electric angle polarity；`0x400A` 为 Hall pole pair | Hall 与位置编码器在参数层面有分别的极性/电角度配置 | 不证明 Hall 是独立第二位置反馈；不证明当前极性和极对数设置正确 |

## 3. 双反馈组合矩阵的型号级解释

以下为 VER 1.1、PDF p.23 的系列级矩阵转录。`O` 表示 GUI 手册允许，`X` 表示不允许；仍须同时满足型号物理端口。

| Feedback1 \ Feedback2 | Incremental | Incremental with Index | Hall Sensor | Sin/Cos | BiSS/SSI | Analog Hall |
|---|---:|---:|---:|---:|---:|---:|
| Incremental | O | O | X | O | O | O |
| Incremental with Index | O | O | X | O | O | O |
| Incremental & Hall | O | O | X | O | O | O |
| Incremental with Index & Hall | O | O | X | O | O | O |
| Incremental with Index shared Hall | O | O | X | O | O | O |
| Sin/Cos | O | O | X | X | O | O |
| BiSS/SSI | O | O | X | O | X | X |
| Analog Hall | O | O | X | X | O | X |
| TAMAGAWA Encoder | O | O | X | O | X | X |

**冲突边界 FB-B1-C01**

- E3-GUI：系列矩阵把 `Incremental + Incremental` 标为允许。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.23，§4.2.1(11)。
- E3-HW：DS-BVM 型号级硬件手册只列一组 A/B/I 引脚。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.8，§2.5，J201 表。
- 证明：固件系列可能支持双增量反馈。
- 不证明：当前 DS-BVM 能在不增加外部转换/不同硬件版本的情况下直接接入两套增量编码器。此项保持“官方资料内部层级不一致”，不得自行选择一个覆盖另一个。

## 4. 当前光栅尺接入 Digital Encoder 后的信号职责边界

| 判断 ID | 信号职责判断 | 标签 | 官方文件完整路径与定位 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| FB-B1-I01 | 若光栅尺以 RS422 A/B 接入 J201，A/B 的官方定义是增量位置变化输入；I 仅在光栅尺提供 Index 且配置为带 Index 时参与基准脉冲功能 | E3-HW + E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF pp.4、8，§1.2/§2.5；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.17，§4.2.1(1)–(2) | A/B 是位置增量来源，I 是可选单脉冲 Index | 不证明当前 Ri0040 实际输出含 I，也不证明计数方向、分辨率或零点已正式标定 |
| FB-B1-I02 | 单反馈时，速度反馈很可能由所选位置传感器的变化量内部估算；但手册未给单反馈下“速度对象/速度环来源”的明确绑定表 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.16，§4.2；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.39–47，§5.5–5.7 | 双反馈时手册明确 Feedback1 负责 speed、Feedback2 负责 position；单反馈存在位置/速度闭环功能 | 不足以把当前 `0x606C` 或速度环严格归因于光栅尺；需参数备份或厂商确认关闭 |
| FB-B1-I03 | `Incremental & Hall` 配置中，Hall 的明确作用是寻找电机初始电角度；Hall-only 也可单独作为反馈 | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.18，§4.2.1(3)–(4) | Hall 可以作为单独反馈，或与增量编码器组合承担初始电角度识别 | 不证明当前平台有 Hall、Hall 已接线或换相全过程持续依赖 Hall；也不说明无 Hall 时当前 PMSM/BLDC 如何完成初始换相 |
| FB-B1-I04 | 仅凭“光栅尺已进入 Digital Encoder”无法从手册确定当前位置、速度和换相三者的实际信号来源 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.8，§2.5；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.16、18，§4.2/§4.2.1 | 手册提供可能的端口和配置分工 | 不证明当前 `Single/Dual Feedback`、`0x4008` 值、Hall 状态、电角度建立方式或速度估算来源 |

## 5. 是否存在“只监测、不控制”的额外位置输入

| 事实 ID | 结论 | 标签 | 官方文件完整路径、版本与定位 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| FB-B1-015 | 本轮官方资料未找到明确命名为“monitor-only encoder / auxiliary position input”的额外位置通道 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF pp.4–11，§1.2/§2.2–2.8；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.16–23，§4.2 | 已见额外反馈被描述为 Feedback1/Feedback2 并参与内环/位置环，而非单纯监测 | 不证明固件内部绝无诊断通道；只说明现有官方资料没有给出可据此设计的监测专口 |

## 6. 本文件的最小结论

1. DS-BVM 有数字、模拟和串行三类物理反馈口，但同时使用必须受 Servo Studio 双反馈规则和型号级端口数量共同约束。
2. 官方双反馈的职责是 Feedback1 管电压/电流/速度，Feedback2 管位置；这为“电机侧反馈 + 负载侧光栅外位置反馈”提供了手册级概念依据。
3. 对当前 DS-BVM，“两套增量编码器直接接入”存在系列 GUI 与型号硬件手册之间的接口证据冲突。
4. 当前光栅尺接到 J201 只足以说明它可能作为增量位置输入；实际速度和换相信号来源仍需当前参数与实物证据。
