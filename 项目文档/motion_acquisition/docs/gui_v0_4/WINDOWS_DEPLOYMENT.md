# Motion Acquisition V0.4 Windows 部署说明

## 发布形态

- Windows x64 PyInstaller `onedir --windowed`。
- 必须复制整个 `MotionAcquisition` 文件夹，不能只复制 EXE。
- 目标电脑按设计无需 Python、源码目录、固定用户名或固定盘符。
- 双击 `MotionAcquisition.exe`，先以 Demo 验证采集、回放、会话扫描和分析。

## 每用户数据

- 配置位于 Qt `AppConfigLocation` 下的 `settings.json`。
- 默认数据位于当前用户“文档/MotionAcquisitionData”。
- 发布目录不保存会话数据；不同 Windows 用户使用独立配置和数据目录。

## 复制到其他 Windows 电脑

1. 复制完整发布文件夹到目标电脑普通可写目录。
2. 对照 `SHA256SUMS.txt` 核对 `MotionAcquisition.exe`。
3. 双击启动，保持默认 Demo。
4. 完成短时采集、主动停止、会话扫描、单会话分析、两会话比较和摘要导出。
5. 没有 PCAN 驱动或适配器时，Demo、回放和分析仍应可用。

当前只在构建电脑完成离线验证，未在第二台 Windows 电脑或干净 VM 实测，不能宣称跨电脑验收完成。

## 外部依赖与边界

PCAN 驱动/PCAN-Basic 不随包分发。真实硬件条件恢复前，不进行连接或采集验证。软件不包含写控制、PDO、EtherCAT或运动功能。
