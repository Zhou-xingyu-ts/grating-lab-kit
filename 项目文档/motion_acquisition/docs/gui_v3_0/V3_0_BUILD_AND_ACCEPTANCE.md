# Motion Acquisition V3.0 本机软件在环构建与验收

验收日期：2026-07-26

## 结论

V3.0 在当前构建电脑完成软件在环功能闭环。三轴命令、状态、任务、故障、记录、分析和 GUI 都只连接进程内模拟控制器。V2 采集、回放、分析、研究和 16 列 raw CSV 保持兼容。

本验收没有访问 CAN、USB、EtherCAT、Servo Studio 或运动硬件，也没有第二台 Windows 电脑证据。

## SIL 场景

持久证据位于 `build\V3_Acceptance_Smoke_20260726`：

| 场景 | outcome | 命令 | 接受 | 拒绝 | 反馈 | 事件 | checksum |
|---|---|---:|---:|---:|---:|---:|---|
| normal-three-axis-task | completed | 7 | 7 | 0 | 11 | 18 | verified |
| fault-recovery-and-operator-stop | stopped | 6 | 5 | 1 | 8 | 12 | verified |
| communication-timeout | error | 0 | 0 | 0 | 3 | 3 | verified |

覆盖三轴使能/回零、双步同步任务、任务完成、主机超限拒绝、模拟轴故障/复位、急停/复位、人工停止、连续超时和 `communication_lost` 错误收尾。

## 当前可访问旧数据

- 扫描：19 个会话；
- 兼容：19/19；
- 记录：212,378；
- 14 组 legacy CSV 全部回放/分析；
- 3 个非空 V2 会话和 1 个当前用户会话全部回放/分析；
- 2 个合法零样本 stopped 会话结构兼容、明确不可回放；
- 审计前后原始 CSV SHA-256 改变：0。

V2.0 在 2026-07-24 曾验收 23/23 持久会话；当前文件系统只可访问上述 19 个，因此 V3 不伪造缺失四个会话的本轮复验。

## 自动测试

- V2 原始基线：`109 passed in 26.03s`；
- V3 首次完整回归：`124 passed in 21.23s`；
- 最终源码完整回归：`124 passed in 22.95s`；
- `compileall`：通过；
- `pip check`：`No broken requirements found.`。

## 发布包

- 路径：`release\MotionAcquisition`
- 类型：Windows x64 PyInstaller `onedir --windowed`
- PyInstaller 构建耗时：约 145.7 s
- 文件数：385
- 总大小：170,314,388 bytes（162.42 MiB）
- EXE SHA-256：`15664B89F3818DB28EC951BF8A81F5F9E90BEB9B47BED625F2E04A75745F3B2F`
- V3 必需文档缺失：0
- 固定用户名/构建盘符/源码路径命中：0
- 内嵌采集或控制会话文件：0
- 旧网页资源：0
- 隔离配置和 offscreen 模式启动后进程持续存活 5 s，随后只终止该测试进程。

PyInstaller 提示可选 `pyqtgraph.opengl` 缺少 OpenGL；V3 使用二维 `PlotWidget`，不依赖该可选模块，构建成功。

GUI 完整三轴交互由同一终版源码的 Qt 离屏端到端测试覆盖；成品 EXE 本轮只做隔离启动检查，不宣称通过桌面自动化点击完成成品交互。

## 延期项

- 未来实时控制器的真实协议和适配器；
- 真实三轴使能、回零、Jog、定位、同步、停止、急停和安全互锁；
- 物理软/硬限位、故障恢复、通信超时和断连；
- 真实周期、延迟、抖动、轨迹、精度和同步；
- PCAN 只读现场采集；
- 第二台 Windows 电脑部署。
