# Motion Acquisition V0.3 Windows 部署说明

## 发布形态

- 平台：Windows x64。
- 形态：PyInstaller `onedir --windowed`。
- 启动文件：`MotionAcquisition.exe`。
- 目标电脑无需安装 Python，也无需保留源码目录。
- 必须复制整个 `MotionAcquisition` 文件夹，不能只复制 exe 或 `_internal`。

## 复制到另一台 Windows 电脑

1. 将完整发布目录复制到 U 盘、移动硬盘或可信文件共享位置。
2. 在目标电脑复制到本机普通目录，不建议直接从压缩包、网络盘或只读介质内运行。
3. 对照 `SHA256SUMS.txt` 核对 exe；发布版当前未做代码签名。
4. 双击 `MotionAcquisition.exe`，先完成 Demo 与历史回放验证。
5. 每台电脑会生成自己的用户配置和数据目录，不访问另一台电脑的实例或文件。

本次只在构建机完成离线验证，尚未在第二台 Windows 电脑或干净虚拟机实际运行。因此“可复制部署”已准备，但“两机实测”仍是待办，不能视为已经验收。

## 每用户目录

- 用户配置：Qt `AppConfigLocation` 下的 `settings.json`，实际路径由当前 Windows 用户和系统决定。
- 默认数据：当前用户“文档”目录下的 `MotionAcquisitionData`。
- 发布目录：运行时不写入源码路径或 exe 目录。
- 配置保存：连接建议值、Node-ID、周期、持续时间、对象选择和数据根目录按 Windows 用户独立保存；应用每次仍从安全的 Demo 模式启动。

## PCAN 外部依赖

PEAK-System 驱动/PCAN-Basic 不随发布包分发。没有驱动或适配器时，Demo 与回放应正常工作；真实模式会给出连接错误。现场重新开放前不要接入或测试硬件。

## 发布目录文件

- `MotionAcquisition.exe` 和 `_internal/`：程序及运行库；
- `README_FIRST.txt`：最短启动说明；
- `V0_3_GUI_USER_GUIDE.md`：零命令行使用教程；
- `WINDOWS_DEPLOYMENT.md`：本部署说明；
- `canopen_readonly.example.yaml`：只读配置示例；
- `RELEASE_INFO.txt`：版本、构建时间和验收边界；
- `DEPENDENCIES.txt`：构建环境依赖锁定；
- `SHA256SUMS.txt`：主程序校验值。

## 当前未完成的外部验收

- 第二台 Windows 实体机或干净 Windows 虚拟机启动测试；
- 安装 PCAN 驱动后的真实连接测试；
- Node 1 静止采集和 Servo Studio 配合运动采集；
- 代码签名、安装器和 onefile 包。

这些项目不影响当前电脑上的无硬件 Demo、回放、保存和完整性检查。
