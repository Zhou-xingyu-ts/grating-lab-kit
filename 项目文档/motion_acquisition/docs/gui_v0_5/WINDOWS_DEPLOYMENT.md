# Motion Acquisition V0.5 Windows 部署说明

## 发布形态

- Windows x64 PyInstaller `onedir --windowed`。
- 必须复制整个 `MotionAcquisition` 文件夹，不能只复制 EXE。
- 目标电脑按设计无需 Python、源码目录、固定用户名或固定盘符。
- 双击 `MotionAcquisition.exe` 后先使用 Demo 验证预设、事件、停止、回放、换算和分析。

## 每个 Windows 用户的文件

- `settings.json`、`presets.json` 和 `conversions.json` 位于 Qt `AppConfigLocation` 对应的当前用户配置目录。
- 默认数据位于当前用户“文档/MotionAcquisitionData”。
- 发布目录不保存用户配置或会话；不同 Windows 用户互不共享预设和换算配置。

## 复制到其他 Windows 电脑

1. 将完整发布目录复制到目标电脑普通本地文件夹。
2. 用系统或可信工具核对 `SHA256SUMS.txt` 中的 EXE SHA-256。
3. 双击 `MotionAcquisition.exe`，保持默认 Demo。
4. 新建采集预设和换算配置，运行短时 Demo，记录至少两个事件，主动停止。
5. 确认会话含五个文件，打开回放、点击事件跳转，并完成会话扫描与单会话分析。
6. 没有 PCAN 驱动或适配器时，Demo、回放、预设、换算和分析仍应可用。

当前只在构建电脑完成离线验证，尚未在第二台 Windows 电脑或干净 VM 实测，因此不能宣称跨电脑验收完成。

## 真实只读模式外部依赖

PCAN 驱动/PCAN-Basic 不随包分发。恢复真实硬件条件后，仍需按现场放行文件核对 Node-ID、bitrate、接线和安全条件。软件只允许 SDO upload，不包含写控制、PDO、EtherCAT、Servo Studio 自动化或运动功能。
