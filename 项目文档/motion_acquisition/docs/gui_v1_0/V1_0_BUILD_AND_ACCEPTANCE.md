# Motion Acquisition V1.0 本机离线构建与验收记录

验收日期：2026-07-24

## 结论

V1.0 通过当前构建电脑上的离线验收。它已经形成“项目 → 采集与事件 → 会话检查 → 分析/比较 → 回放 → 交付”的完整只读工作流。真实 PCAN/CANopen、运动配合采集和第二台 Windows 电脑部署仍未验证，不能用本记录中的 Demo 结果替代。

## 产品闭环

- 实验工作台支持轻量项目的新建、保存、复制、重命名、删除和历史快照。
- 会话按项目归档，旧会话继续作为“未归类”加载，不迁移、不改写。
- Demo/真实只读入口、采集预设、人工事件、未验证换算、实时数值和有界曲线保留。
- 会话浏览器显示项目、outcome、轮次、频率、失败数、deadline miss、完整性、事件和配置快照。
- 单会话分析、多会话比较、历史回放、事件跳转和三种分析摘要继续可用。
- 最终会话增加 SHA-256 账本；损坏只报告，不自动修复原始文件。
- 交付包按项目或会话一键导出，包含源文件、完整性报告、分析摘要、包清单和独立 SHA-256 账本；取消不留下成品或 `.part` 文件。
- 帮助页提供零硬件探测的产品诊断和 JSON 导出。

## 数据格式与兼容性

新会话的权威数据文件仍为 16 列 `samples.csv`，只保存 raw counts。每个终态会话包含：

1. `samples.csv`
2. `manifest.json`
3. `acquisition.log`
4. `config_snapshot.yaml`
5. `events.jsonl`
6. `checksums.sha256`

`manifest.json` schema 仍为 `1.0`。V1.0 只增加可选的 `project`、`integrity` 和 `files.checksums`；既有 `preset`、`conversion`、`events_summary` 语义不变。没有这些字段或没有校验账本的旧会话继续兼容，并显示为 `not_available`，而不是误报损坏。

持久数据兼容审计覆盖 23 个 V1.0 之前的会话、266,056 条记录：

- legacy：14
- completed：7
- stopped：2
- 兼容：23/23
- 原始 CSV SHA-256 改变：0

## 测试结果

- V1.0 首次全量回归：`86 passed in 15.08s`
- 兼容选择器修复后全量回归：`86 passed in 15.98s`
- 最终全量回归：`86 passed in 16.24s`
- 最终 GUI/交付/无写入/可移植专项：`9 passed in 3.24s`
- Python 依赖检查：`No broken requirements found.`

自动化测试覆盖项目 CRUD/快照、completed/stopped/error、异常退出恢复、事件保留、16 列 CSV、预设与未验证换算、会话扫描、分析、比较、回放、事件跳转、完整性校验、损坏检测、交付包校验/取消、固定路径扫描和真实写 API 禁止。

## 发布包

- 路径：`release\MotionAcquisition`
- 类型：Windows x64 PyInstaller `onedir --windowed`
- 文件数：379
- 总大小：170,117,950 bytes（162.24 MiB）
- `MotionAcquisition.exe` SHA-256：`50F669DB9D0DAF9C7D57E87DBAB9BFF0FDEF33DB999A4F55F37C0D2B59B72E4B`
- 固定用户名、构建盘符或源码路径命中：0
- 内嵌历史会话文件：0
- 旧网页发布候选：0

发布包已直接启动并成功渲染 V1.0 只读实验工作台，标题、项目区、健康概览和七个功能页均可通过 Windows 可访问性树读取。完整项目—Demo—事件—停止—扫描—交付—回放交互链由同一构建源码的 Qt 离屏 GUI 测试通过。

额外的发布包鼠标/键盘注入冒烟未形成证据：Windows Computer Use 能读取窗口和截图，但两次输入尝试均返回 `failed to activate captured window`。按其恢复规则停止继续注入，未把自动化工具无法置前窗口误写成应用失败，也未伪造点击结果。

## 安全和延期项

- 包内真实设备实现只有 CANopen SDO upload；未新增 SDO download、NMT/PDO、控制字、Servo On、Jog、回零、运动命令、EtherCAT 或多轴控制路径。
- 本次没有访问 CAN、USB 或运动硬件。
- 待现场验证：PCAN_USBBUS1、1 Mbps、Node-ID 1 的只读连接、静止采集、Servo Studio 配合运动采集、断线/总线异常收尾。
- 待第二台 Windows 电脑验证：完整目录复制、依赖/驱动环境、启动、Demo、回放、分析和交付。

