# Motion Acquisition V9.0.0 离线候选文档索引

<!-- last_reviewed: 2026-10-23 -->

20261023-ux2 桌面修订先读 [发布说明](V9_0_RELEASE_NOTES.md) 和工程 `CLAUDE.md`；包内操作说明来自 `packaging/README_DESKTOP.txt`。下方三路线与完整候选门是历史项目框架，不把它们当作本轮桌面改进重新构建固件或连接硬件的任务清单。

## 文档状态与适用范围

本目录描述 Motion Acquisition V9.0.0 的离线候选。它服务于源码、SIL/Mock、固件候选、配置和发布验证完成后的线下测试准备，**不构成连接、烧录、总线扫描、Servo On、控制或运动授权**。

本版唯一允许的离线完成结论是：

> READY_FOR_LAB_TEST

该结论只在离线验证报告的全部必填证据已填写、哈希已复核且没有未处理的阻断项时使用。它不表示 STM32、串口、FDCAN、CANopen、EtherCAT、PDO/SYNC、GM1、三轴、安全链或现场控制已经通过。

当前工作状态由工程根目录的 AGENTS.md 和 CLAUDE.md 决定；本目录不覆盖其硬件安全边界。

## 三条物理路线

| Route ID | 拓扑 | V9 定位 | 当前可写的结论 |
|---|---|---|---|
| windows_pcan_direct | Windows → PCAN → DS-BVM | Route A，既有固定单轴现场基线的兼容纳管 | 历史基线存在；V9 集成仍须离线回归证明 |
| stm32_canopen_bridge | Windows → STM32H743 → CANopen → DS-BVM | Route B，主推路线 | 仅离线候选，待上板、串口、FDCAN 和现场证据 |
| gm1_ethercat_controller | Windows → Modbus TCP → GM1 → EtherCAT → DS-BVM | Route C，GM1/EtherCAT 候选 | 仅 Mock/契约/ESI 审计候选，待 GM1 工程与现场证据 |

GM1 是 Route C 的 EtherCAT 主站；Windows 只管理 GM1，不成为 EtherCAT 主站。共享硬件底座任一时刻只能有一个运动主站。

## 阅读顺序

1. [V9 架构与安全边界](V9_0_ARCHITECTURE.md)
2. [V9 离线用户指南](V9_0_OFFLINE_USER_GUIDE.md)
3. [三路线配置说明](V9_0_ROUTE_CONFIGURATION.md)
4. [离线验证报告模板](V9_0_OFFLINE_VERIFICATION_REPORT.md)
5. [声明—证据矩阵](V9_0_CLAIM_EVIDENCE_MATRIX.md)
6. [缺陷与未闭合项](V9_0_DEFECT_LEDGER.md)
7. [构建与发布证据记录](V9_0_BUILD_RELEASE_EVIDENCE.md)
8. [V9 候选发布说明](V9_0_RELEASE_NOTES.md)
9. [Route A 回归检查卡](ROUTE_A_REGRESSION_CHECK_CARD.md)
10. [Route B 首次实验入口](ROUTE_B_STM32_LAB_ENTRY.md)
11. [Route C GM1/EtherCAT 实验入口](ROUTE_C_GM1_ETHERCAT_LAB_ENTRY.md)
12. [Route C 官方 ESI 只读审计入口](V9_0_ROUTE_C_ESI_AUDIT_ENTRY.md)

## 文档与证据分层

| 层次 | 可写内容 | 不可替代的内容 |
|---|---|---|
| 源码与单元测试 | 接口、Mock、SIL、错误处理、配置校验 | 真实设备行为 |
| 构建与发布候选 | 工具链、依赖、清单、哈希、可启动性 | 板级或现场验收 |
| Route A 历史资料 | 固定单轴基线的既有受限证据 | 对 B/C、三轴或实时性的外推 |
| 线下测试入口 | 顺序、授权门、待采集证据、停止条件 | 已完成的现场结果 |

原始数据、16 列 samples.csv、历史控制记录、checksums.sha256、release 和既有 V8 固件证据均保持原样。新增 V9 元数据只能是可选字段，不能重写历史解释。

## 主代理待填写的最小证据包

在形成最终 V9 离线结论前，主代理须将以下事实填入离线验证报告和声明—证据矩阵：

| 证据 | 待填写内容 |
|---|---|
| 源码基线 | [待填：源文件清单、版本、哈希或等价可追溯标识] |
| 测试 | [待填：定向测试命令、全量回归命令、实际通过数、零失败证据] |
| 固件候选 | [待填：GNU/Keil 实际构建输出、符号/内存/哈希、写权限审计] |
| 数据保护 | [待填：正式 Route A 会话 checksums、16 列 CSV 完整性和回放核验结果] |
| 发布验证 | [待填：源码目录外启动记录、依赖清单、文件清单、发布物哈希] |
| 无硬件访问 | [待填：所有本轮命令均为 Fake/Mock/SIL/离线构建，未打开任何真实端点的记录] |

没有上述证据时，状态只能写“实施中”或“待离线验证”，不能写 READY_FOR_LAB_TEST。
