# Motion Acquisition V0.3 GUI 使用教程

本教程全程不需要命令行。软件是只读数据采集工具，不是运动控制台。

## 第一次启动

1. 将完整的 `MotionAcquisition` 发布文件夹复制到本机普通目录，例如“文档”或桌面下的独立文件夹。不要只复制 exe。
2. 双击 `MotionAcquisition.exe`。
3. 首次遇到 Windows 安全提示时，核对文件来源和 `SHA256SUMS.txt` 后再决定是否运行。本版本未做代码签名。
4. 软件默认进入 `DEMO — NO HARDWARE`，不会打开 CAN。

## 五步完成 Demo 采集

1. 保持数据源为 `Demo（无硬件）`。
2. 填写实验名称；选择所需对象、采样周期和持续时间。位置、速度和电流均按 raw/counts 显示，不做物理单位换算。
3. 核对数据根目录。默认目录属于当前 Windows 用户：`文档/MotionAcquisitionData`。
4. 点击“开始采集”。可切换到“实时曲线”页观察；切页不会控制 CSV 保存。
5. 到达持续时间后自动显示 `completed`。也可点击“停止采集”，此时结果应为 `stopped`。两种结果都会生成四个文件。

每个会话目录包含：

- `samples.csv`：权威的16列逐条记录；
- `manifest.json`：配置、统计、安全边界和明确结果；
- `acquisition.log`：会话打开、停止和最终化日志；
- `config_snapshot.yaml`：启动采集瞬间的有效配置。

## 历史会话和完整性检查

1. 打开“历史回放”页。
2. 新版会话点击“打开会话目录”；旧版数据点击“打开 CSV”。
3. 页面会显示状态：`completed`、`stopped`、`error`、`incomplete`、`recovered_incomplete`、`legacy` 或 `csv_only`。
4. 使用播放/暂停、速度和轮次滑块查看数据。
5. “导出诊断包”会导出完整性报告、manifest、日志和配置快照，不包含体积较大的 `samples.csv`。

若会话因断电或进程终止仍为 `in_progress`，页面显示 `incomplete`。确认后点击“标记中断会话为已恢复”，manifest 会原子更新为 `recovered_incomplete`；已有 CSV 不会被修改，也不会被冒充为正常完成。

## 真实 PCAN 只读模式

只有现场条件重新具备并明确放行后才使用：

1. 安装 PEAK-System PCAN 驱动并连接适配器。
2. 在实时采集页选择“真实 PCAN/CANopen”。默认 `PCAN_USBBUS1 / 1 Mbps / Node 1` 只是当前项目建议值，必须以当次现场证据为准。
3. 点击“连接”，再点击“只读通信测试”。只有所选对象全部完成 SDO upload 后才能开始真实采集。
4. 上位机不提供任何运动按钮。运动仍由另行允许的 Servo Studio 路径操作。

禁止把“停止采集”理解为停止电机。它只设置线程安全停止事件，停止新的读取并完成文件收尾。

## 常见问题

- 无法启动：确认复制了整个发布目录，而不是只复制 exe。
- 提示缺少 PCAN 驱动：Demo 和回放仍可使用；真实模式需要另行安装厂商驱动。
- 历史数据不兼容：查看完整性问题列表，原始文件不会被自动删除或改写。
- 找不到数据：查看实时采集页的数据根目录；每个 Windows 用户使用自己的配置和数据目录。
- 退出时仍在采集：确认停止后，软件会等待 CSV 和 manifest 收尾，不会强杀采集线程。

## 安全边界

发布版仅含 CANopen SDO upload 读取路径；不提供 SDO download、控制字、目标值、NMT 写入、PDO 配置、EtherCAT、Servo On、Jog、Home、Move、Fault Reset 或 Quick Stop。
