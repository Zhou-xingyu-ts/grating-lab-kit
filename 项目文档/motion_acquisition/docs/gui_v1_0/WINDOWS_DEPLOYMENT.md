# Motion Acquisition V1.0 Windows 部署说明

## 复制与启动

1. 复制完整的 `release\MotionAcquisition` 目录，不要只复制 EXE。
2. 在目标 Windows 电脑上双击 `MotionAcquisition.exe`。
3. 首次启动默认是 Demo，不会自动打开 PCAN。
4. Python、源码目录、原电脑用户名和原盘符都不是运行依赖。

## 每个 Windows 用户独立保存的内容

Qt `AppConfigLocation` 下保存：

- `settings.json`
- `projects.json`
- `presets.json`
- `conversions.json`

默认数据目录位于当前用户 Documents 下的 `MotionAcquisitionData`。用户可在 GUI 中选择其他有写权限的目录。

## 真实 PCAN 使用条件

发布包不捆绑 PEAK 驱动。真实只读采集前必须：

- 安装与目标电脑匹配的 PEAK PCAN 驱动；
- 确认适配器和通道；
- 依据现场证据确认波特率和 Node-ID；
- 在 GUI 中先连接并完成逐对象 SDO upload 测试。

软件不会发送 SDO download、NMT 写入、PDO 配置、控制字或运动命令。

## 数据迁移

- 复制整个数据根目录即可迁移项目会话。
- `projects.json` 是当前用户的项目快捷配置；历史 manifest 已保存项目快照，即使不复制 `projects.json`，历史会话仍能显示项目信息。
- 旧版会话无需转换或改名。
- 不要修改真实实验会话中的 `samples.csv`、manifest 或 `checksums.sha256`。

## 校验发布包

发布目录的 `SHA256SUMS.txt` 记录 `MotionAcquisition.exe` 的 SHA-256。复制后可用 PowerShell 只读校验：

```powershell
Get-FileHash .\MotionAcquisition.exe -Algorithm SHA256
```

## 当前验证边界

- 已完成构建电脑上的离线 Demo、项目、事件、停止、回放、分析、比较、完整性和交付包验证。
- 未在第二台 Windows 电脑或干净虚拟机完成实测。
- 真实硬件验证仍需现场 PCAN/驱动器条件，不能由离线测试替代。
