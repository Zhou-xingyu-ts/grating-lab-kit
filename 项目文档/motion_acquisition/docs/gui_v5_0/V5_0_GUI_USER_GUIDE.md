# Motion Acquisition V5.0 GUI 使用教程

## 1. 启动

在发布目录双击 `MotionAcquisition.exe`。不要单独复制 EXE；`_internal`、两个 helper 和文档必须与主程序一起保留。

程序默认不会连接 CAN、STM32 或驱动器，也不会启动控制器进程。

## 2. 选择三轴软件在环控制器

进入“三轴控制（SIL）”，在“SIL 传输”选择：

- **V5 原生控制器参考进程（纯 C 核心）**：验证未来固件侧 C 状态机和真实帧边界；仅支持 Scheme A synthetic count fixture；
- **本机跨进程 STM32 边界 SIL**：V4 Python helper，支持 A/B/C 和故障注入；
- **进程内单元 SIL**：快速单元回归。

原生模式会自动固定 Scheme A。它不是 STM32 仿真器，也没有 CANopen、DS-BVM 或硬件写入。

## 3. 原生模式最小操作

1. 填写控制记录名称；
2. 选择“V5 原生控制器参考进程”；
3. 点击“连接模拟控制器并开始记录”；
4. 确认协商能力包含：
   - `scheme-a-drive-position-loop-native-fixture`
   - `count`
   - `real_hardware_access=false`
   - `device_write_capability=false`
5. 在某轴点击“使能”；
6. 填写 synthetic count 目标和 count/s rate；
7. 点击“绝对”。GUI 会按能力发送 generic position setpoint，而不是假定 legacy move verb；
8. 观察 `accepted → running → completed`、位置和事件；
9. 点击“完成并保存”或“停止并保存”。

原生模式不支持回零、Jog、相对运动、同步任务或故障注入，相关按钮按 capability 禁用。禁用代表“控制器未声明能力”，不是软件故障。

## 4. 控制记录

控制记录仍写入用户数据目录的 `control_sessions/`：

```text
control_manifest.json
commands.jsonl
acknowledgements.jsonl
command_statuses.jsonl
feedback.jsonl
control_events.jsonl
tasks.jsonl
control.log
checksums.sha256
```

`control_manifest.json` 保存控制器 capability、责任 profile、transport、boot ID 和无硬件/无写能力证据。旧 V3/V4 控制记录继续加载。

## 5. 采集、回放和研究

实时采集、16 列 raw `samples.csv`、历史回放、事件、分析、实验组和研究输出与 V4 相同。V5 原生控制器记录不会改写任何采集会话。

真实 PCAN 页面仍只有 SDO upload 只读采集。没有现场条件时使用 Demo。

## 6. 安全与证据边界

- 所有三轴 GUI 操作只到软件模型；
- PC 保护性停止不是物理急停、STO 或安全功能；
- 原生 fixture 的 count 范围、rate、100 Hz 和轨迹都不是硬件参数；
- 停止上位机采集不会停止电机；
- 真实 STM32、CANopen、驱动模式、反馈、同步、精度和安全必须现场另验；
- 第二台 Windows 电脑部署也尚未由本机验收替代。

