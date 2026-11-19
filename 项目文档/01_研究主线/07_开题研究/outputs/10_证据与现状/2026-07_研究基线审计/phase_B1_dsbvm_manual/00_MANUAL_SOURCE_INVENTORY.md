# 阶段 B1 官方资料清单

## 1. 审计边界

- 本清单只登记 DS-BVM/DS-BVS-BVM 系列官方手册、官方配件资料和工作区官方软件目录中的 ESI XML。
- 未读取师兄论文、师兄 STM32/CANopen 源码、普通网络文章、论坛或通用 CANopen 教程。
- PDF 页码均指 PDF 阅读器页码；配件手册同时注明印刷页码。XML 以行号定位。
- `接口存在 ≠ 固件允许同时使用 ≠ 当前参数已经配置 ≠ 当前台架已经验证`。
- `支持某模式 ≠ 已配置 PDO ≠ 能稳定运行在 1 kHz ≠ 已实现实时外环`。

## 2. 纳入来源

| 来源 ID | 标签 | 官方文件完整路径 | 文件版本 / 标识 | 定位范围 | 用途 | 限制 |
|---|---|---|---|---|---|---|
| SRC-B1-001 | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | `DS-BVM-FETC/FCAO Hardware Manual`, VER 1.0；15 PDF 页；SHA-256 `360C77E178962F2F84DF242F3F3F75CA0774F941401C4223EB6D5AA02E8B1814` | PDF pp.4–14，§1.2、§2.2、§2.5、§2.6、§2.8、CAN 接口页 | 型号级物理反馈接口、引脚、电气输入能力、通信接口 | 不说明当前台架接法、当前参数、双反馈固件组合或控制周期保证 |
| SRC-B1-002 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf` | `DS-BVS/BVM-Series Reference Manual`, VER 1.0；181 PDF 页；SHA-256 `08B06528D1E7D0FE1C48D944D036AE7F6E47754F9561D1A8C84070739329AEE8` | PDF pp.13–27、28–61、81–105、140–176 | CANopen、CiA 402、运行模式、对象字典、PDO/SYNC | 系列级手册；若与型号级硬件手册或 ESI 冲突，不自动覆盖；部分表格只给字段而无语义正文 |
| SRC-B1-003 | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf` | `DINGS’ Servo Studio Manual`, DS-BVS/BVM-Series, VER 1.1；57 PDF 页；SHA-256 `AC7E3896EDA6E6C7B962C8E3068555F83B7AABA0E86F50A37CCC3D79BE8BF801` | PDF pp.16–23，§4.2、§4.2.1 | 单/双反馈配置、反馈类型、Feedback1/Feedback2 分工、组合矩阵 | 系列级 GUI 手册；不能单凭 GUI 选项证明某一 DS-BVM 型号有足够的物理端口或当前固件已启用 |
| SRC-B1-004 | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual_官网20260601下载版.pdf` | `DINGS’ Servo Studio Manual`, VER 1.0；58 PDF 页；SHA-256 `397DA7A314CCCB6A4D5E49DEDA1ACAAF4EC5A58354122551AF4E3B0BE06C1FF7` | 与 SRC-B1-003 对照，反馈设置章节页码较旧版偏移 | 版本交叉核对 | 本审计以更新的 VER 1.1 为主；旧版不是独立功能验证 |
| SRC-B1-005 | E3-ACC | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Motion_Controller_Accessory_Selection_Manual_CN.pdf` | 《分体驱动配件选型手册》Ver1.2；41 PDF 页 / 印刷 38 页；SHA-256 `4DD79FEEE431E59A672EC3ED3018B8A02E4ED574CDA0B86F760B8A0AF9C2D027` | PDF pp.29–34（印刷 pp.26–31），第七章 DS-BVM-FETC/FCAO | DS-BVM 配件线 P4/P5/P6/P7/P10/P11 与引脚复核 | 线束定义不能证明固件配置、传感器兼容性或闭环性能 |
| SRC-B1-006 | E3-EDS | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml` | EtherCATInfo Version 1.1；设备 `WELCON EtherCAT Servo Drive`，ProductCode `0x00000001`，RevisionNo `0x20200525`；文件时间 2022-05-16；SHA-256 `92733290D5ADC3FAD608B78B174043123CFAE44C2C9D834A574C8B10010CB69D` | XML lines 1–20、4977–5184、10364–10426、11101–11123、11381–11390、11489–12046 | EtherCAT 对象可映射标志、固定/可配置 PDO、DC/周期字段 | 官方包目录指向 DS-BVS-FETC/DS-BVM-FETC，但 XML 内部品牌为 WELCON，且未证明与当前驱动器硬件/固件精确匹配；不能代替 CANopen EDS |

## 3. 重复件与未发现项

| 来源 ID | 标签 | 官方文件完整路径 | 版本 / 定位 | 结论 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| SRC-B1-007 | E3-HW | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\2026-06-01_聊天记录补充资料\DS-BVM-FETC.pdf` | 与 SRC-B1-001 字节数及 SHA-256 完全相同 | 精确重复件，不作为第二份独立证据 | 工作区有同一硬件手册的另一份副本 | 不能提高结论的独立证据强度 |
| SRC-B1-008 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\` | 本轮只在官方软件/官方手册定向范围内核查 | 未找到明确标注为当前 DS-BVM CANopen 型号的官方 EDS；找到的是 EtherCAT ESI XML | 当前工作区优先官方资料中缺少可直接核对 CANopen 默认映射的型号级 EDS | 不证明厂商从未发布 CANopen EDS，也不证明现场驱动器不存在内部对象字典 |

## 4. 资料优先级规则

1. 物理端口和引脚优先采用型号级 `SRC-B1-001`、`SRC-B1-005`。
2. CANopen/CiA 402 语义优先采用 `SRC-B1-002`。
3. 单/双反馈配置采用 `SRC-B1-003`，但必须与型号级物理端口交叉检查。
4. `SRC-B1-006` 仅作为 EtherCAT ESI 证据；其内部设备身份和当前实物匹配仍为未关闭缺口。
5. 官方资料冲突只列冲突，不在本阶段自行决定哪个版本代表当前固件。
