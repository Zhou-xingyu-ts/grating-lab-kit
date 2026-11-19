# Motion Acquisition V4.0 本机构建与验收

日期：2026-07-26  
验收范围：当前 Windows 电脑、源码测试、历史数据兼容和软件在环；未访问 CAN、STM32、驱动器、电机或安全硬件。

## 1. 结论

V4.0 的当前电脑离线软件验收覆盖：

- 方案 A、方案 B 和 velocity 命令方案 C 共用一套能力驱动契约；
- 进程内 SIL 与独立 helper 进程的本机回环协议；
- completed、operator-stopped、timeout/error 控制会话；
- V1～V3 采集/研究数据与 V3 控制记录兼容；
- GUI、控制记录、分析、回放和 Windows `onedir` 发布。

这不是 PC/STM32 实机通信、三轴运动、CANopen 对象字典、安全、同步或精度验收。

## 2. 自动测试与持久证据

- V4 持久 SIL 输出：`build/V4_Acceptance_Smoke_20260726/`；
- 方案 A：跨进程、自然完成；
- 方案 B：主动停止；
- 方案 C：velocity 接口任务、自然完成；
- 异常：跨进程超时并按 error 收尾；
- 所有控制会话均通过 SHA-256 账本校验并可生成分析摘要。

历史兼容审计覆盖 19 个可访问采集会话、53,954 个采样轮次、212,378 行 CSV；其中 17 个数据会话可回放/分析，两个零轮次 stopped 会话保持结构兼容。源 `samples.csv` 哈希变化为零。三组现有 V3 控制记录均通过校验和分析。

- 全量测试：`136 passed in 18.77s`；
- `compileall`：通过；
- `pip check`：无损坏依赖；
- PyInstaller 构建：通过，136.9 s；
- 发布目录：387 个文件，`172171973` bytes（164.20 MiB）；
- `MotionAcquisition.exe` SHA-256：`A1310558DD419253603A83C3A294BD20C09574B84FB63CBECCAFDE2803EDB5D1`；
- `MotionAcquisitionSilController.exe` SHA-256：`DDEA6FA2A8184981BAFDCE119DDF233228C601737CC2B346A10746DF6EEDE655`；
- 固定用户名、构建盘符、源码路径、历史会话、旧网页资产扫描：零命中；
- 主 GUI 在隔离用户目录下保持运行 5 s，且未在用户连接前自动启动 helper；
- 冻结 helper 完成 4.0 协商、Scheme A 能力读取、X 轴 SIL 命令、ACK、completed 状态查询、断开和进程正常退出；能力明确报告 `hardware_access=false`、`device_write=false`。

## 3. 发布结构

```text
release/
  MotionAcquisition/
    MotionAcquisition.exe
    MotionAcquisitionSilController.exe
    _internal/
    README_FIRST.txt
    RELEASE_INFO.txt
    SHA256SUMS.txt
    V4_0_*.md
    WINDOWS_DEPLOYMENT.md
```

必须复制整个 `MotionAcquisition` 目录。主程序只会在用户进入三轴控制并选择跨进程 SIL 时启动同目录 helper；helper 只监听随机数字回环地址。

## 4. 仍未验收

- PC 与真实 STM32 的传输、重连和命令对账；
- 三个驱动器的实际 CANopen 能力与对象字典；
- 闭环位置、反馈接法、驱动模式和方案 A/B/C 的现场选择；
- 物理急停、STO、安全链和故障复位职责；
- 三轴同步、轨迹周期、抖动、运动、精度和外部计量；
- PCAN 现场只读采集；
- 第二台 Windows 电脑部署。
