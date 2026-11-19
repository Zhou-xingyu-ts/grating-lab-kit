# 官方资料歧义与证据缺口

## 1. 官方资料冲突表

| 冲突 ID | 资料 A | 资料 B | 冲突 / 歧义 | 本阶段处理 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| GAP-B1-C01 | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.23，§4.2.1(11)：系列双反馈矩阵允许 Incremental + Incremental | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.8，§2.5：DS-BVM 只列一组 A/B/I 引脚 | 系列固件组合与型号级物理接口数量不一致 | 不推断两套增量尺可直接接入；列为型号/端口映射待厂商确认 | 双增量组合在系列 GUI 层存在 | 不证明当前 DS-BVM 有第二套增量输入 |
| GAP-B1-C02 | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.17–22，§4.2.1：列 10 类反馈 | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.104，§6.2.41：`0x4008` 仅列 0x00/0x02/0x03 | `0x4008` 显然不是已公开的完整 GUI 类型枚举，或手册对象表不完整 | 不用 `0x4008` 三个值否定 Sin/Cos/BiSS/SSI/Analog Hall；也不猜缺失编码 | GUI 和硬件资料支持更多反馈类型 | 不证明这些类型由哪个对象/子对象配置 |
| GAP-B1-C03 | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.154–155、176，§6.2.96–6.2.98、§6.2.141：`6062/6063/60F4` 不可 PDO，`6064` 可 TxPDO | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 10389–10426、11380–11390、11748–11784：四者均标 TxPDO，且 6062/60F4 有 TxPDO 条目 | CANopen Reference 与 EtherCAT ESI 的映射能力不同 | 按总线分开；CANopen 采用 Reference，EtherCAT 先核对 ESI 与设备身份 | 不同官方文件确实给出不同映射定义 | 不能合并成“DS-BVM 对所有总线都可/不可映射” |
| GAP-B1-C04 | ESI 位于 DS-BVS-FETC/DS-BVM-FETC 官方包目录 | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 1–20：Vendor/Type/Name 为 WELCON，Revision `0x20200525` | 包目录品牌与文件内部设备身份不一致 | 降级为候选 E3-EDS，不作为当前实物固件的最终对象字典 | 该 XML 是工作区官方软件包中的 EtherCAT 设备描述 | 不证明当前 DS-BVM 的 ProductCode/Revision 与其一致 |

## 2. 按影响排序的证据缺口

| 缺口 ID | 影响 | 当前官方证据 | 缺什么 | 可关闭该缺口的资料 / 验证 | 当前结论 |
|---|---|---|---|---|---|
| GAP-B1-001 | 极高：决定双反馈硬件能否搭建 | GUI 允许双反馈并给组合矩阵；BVM 只有一套 A/B/I。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.16、23；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF pp.5、8–11 | DS-BVM 双反馈时 Feedback1/2 到 M6/M7/M8 的明确端口映射；两套增量信号的型号级接法 | 厂商型号级双反馈应用说明、当前固件参数导出、供应商书面确认、后续只读 GUI 截图 | 双反馈概念明确；双增量物理接入冲突未闭合 |
| GAP-B1-002 | 极高：决定当前单反馈的位置/速度/换相链 | GUI 只说明双反馈职责；Incremental+Hall 中 Hall 找初始电角度。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF pp.16、18 | 当前 Single/Dual、`0x4008`、Hall 状态、极对数、电角度建立方式、速度估算来源 | 当前参数备份、型号/电机配置页、厂商说明、后续受控台架验证 | 光栅接 J201 不足以确定三类信号实际来源 |
| GAP-B1-003 | 极高：决定实时外环是否可承诺 | 仅有 10 MHz 输入、24 kHz 模拟采样、TPDO 定时器误差和 ESI 零/默认周期字段。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf`，VER 1.0，PDF p.4；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.20–21；ESI lines 4977–5184、12031–12046 | 最小 PDO 周期、SYNC/DC 抖动、内部环频率、输入到输出延迟、1 kHz 额定保证 | 厂商实时性能规格、固件 release note、型号级 EtherCAT/CANopen 应用说明、后续总线时间戳实测 | 未找到官方 1 kHz 或其他确定周期保证 |
| GAP-B1-004 | 高：决定对象能否区分两路反馈 | `6063` 只有名称；`6064` 只称 sensor-measured motor position。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.155，§6.2.97–6.2.98 | 双反馈时 `6063/6064` 分别绑定哪路；是否存在 Feedback1/2 的独立位置对象 | 厂商对象字典补充说明、GUI 监视变量映射、受控双反馈对照实验 | 不得写成 `6063=电机侧、6064=负载侧` |
| GAP-B1-005 | 高：决定跟随误差论文定义 | `60F4` 只称 actual following error；`6065` 说明 demand/actual 差的窗口。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.156、176，§6.2.99、§6.2.141 | 精确公式、符号、滤波、饱和、更新时刻 | 厂商公式说明、固件对象字典、后续对照采样 | 只能称驱动器报告的跟随误差，不能自行指定公式 |
| GAP-B1-006 | 高：决定 CANopen PDO 实现 | Reference 默认 4 对映射为空；无型号级 CANopen EDS。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.81–92；工作区官方软件范围只找到 EtherCAT ESI | 当前固件允许的 CANopen 映射对象、COB-ID/Transmission Type 默认实值 | 型号匹配 CANopen EDS、当前驱动器只读对象备份、厂商配置样例 | 支持 PDO，但当前映射和可用 payload 未验证 |
| GAP-B1-007 | 高：决定 EtherCAT 证据能否用于当前型号 | 官方包 XML 内部为 WELCON，ProductCode/Revision 未与实物比对。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，lines 1–20 | 当前 EtherCAT identity、ProductCode、Revision、固件版本 | 从站 EEPROM/Servo Studio 只读身份页、厂商发布包说明 | ESI 只能作为候选，不可覆盖实物 |
| GAP-B1-008 | 中高：决定 CSP 是否是真正的光栅反馈方案 | CSP 保留 DS 位置环；GUI 双反馈允许 Feedback2 管位置。证据：Reference VER 1.0，PDF pp.45–47；Servo Studio VER 1.1，PDF p.16 | 光栅究竟进入 DS Feedback2 还是只进入 STM32；`6064` 的反馈来源 | 明确拓扑图、当前/候选参数页、端口映射确认 | “STM32 轨迹 + CSP”本身不足以说明光栅参与位置闭环 |
| GAP-B1-009 | 中：决定自研尺电气适配 | BVM 数字口要求 RS422 A/B(/I)，四倍频前 10 MHz；其他口有各自信号。证据：Hardware VER 1.0，PDF pp.4、8–11 | 自研尺输出电平、差分驱动、最大边沿率、Index/绝对协议、供电与隔离 | 自研尺接口规格、示波/逻辑分析、电气原理图 | 只能给接口门槛，不能断言自研尺可直接接入 |
| GAP-B1-010 | 中：决定是否有纯监测第二位置量 | 现有资料只定义 Single/Dual Feedback 的控制职责，没有 monitor-only encoder。证据：Servo Studio VER 1.1，PDF pp.16–23；Hardware VER 1.0，PDF pp.4–11 | 明确的辅助位置监测通道或对象 | 厂商应用说明/扩展对象说明 | 未找到，不等于绝对不存在 |

## 3. 三条最容易误解的手册表述

1. **“有三个反馈连接器”不等于“三路反馈能同时参加闭环”。** 物理端口存在只证明接入口；并发组合仍受 Single/Dual 和兼容矩阵约束。
2. **“CSP/CSV/CST 每个 PDO 更新周期续写目标”不等于“官方保证 1 ms”。** 手册没有给最小周期、最大频率或同步精度保证。
3. **“Dual Feedback”不等于“`0x6063` 和 `0x6064` 分别就是两只编码器”。** Reference 没有建立这种对象绑定。

## 4. 阶段 B2 可关闭的范围

师兄论文和 STM32/CANopen 源码最多能回答“既有方案声称/实现了什么”，不能替代厂商关闭以下型号级缺口：双增量物理端口、对象内部语义、固件周期保证、ESI 精确设备身份。B2 应把“源码存在、论文仿真、论文实物实验、当前台架复现”分别标记。
