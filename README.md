# 光栅尺实验室电脑工具包

在实验室 Windows 电脑上安装 CAN 驱动、解压上位机，即可准备采集；不需要安装 Python，也不依赖宿舍电脑、源码目录或开发环境。

**先打开 [Releases 下载页](https://github.com/Zhou-xingyu-ts/grating-lab-kit/releases/latest)，下载 `GratingLabKit-Windows-x64-20261031.zip`。** 它包含完整上位机、手册、PEAK 官方驱动下载包和已有 Servo Studio 安装包。GitHub 自动生成的 “Source code (zip)” 只有本仓库的手册与记录，不含上位机 EXE。

## 第一次在实验室电脑使用

1. 用自己的 GitHub 账号登录并下载完整工具包；也可提前下载到 U 盘带入实验室。
2. 完整解压到本地目录，例如 `C:\LabTools\GratingLabKit`；保留 `software\MotionAcquisitionV9\_internal` 全部内容。
3. 阅读 [首次安装](manuals/01_首次安装.md)，准备 Windows 10/11 **x64** 和安装驱动所需的管理员权限。软件自身以普通用户运行。
4. 先在**不接实验设备**时打开 `software\MotionAcquisitionV9\MotionAcquisitionV9.exe`，核对标题为 `20261023-ux2`。
5. 在界面设定本地数据目录，再按 [采集操作](manuals/02_采集操作.md) 选择只读采集或固定单轴直控。连接、预检、使能和运动均属于具体现场操作，按现场放行执行。
6. 完成后按照 [数据保存与带回](manuals/03_数据保存与带回.md)，复制完整会话文件夹到 U 盘，回宿舍用同一软件回放。

## 软件入口

| 软件 | 用途 | 必需程度 |
|---|---|---|
| `MotionAcquisitionV9.exe` | 主窗口：只读采集、会话管理、回放、PNG 和基础摘要 | 必需 |
| `MotionAcquisitionV9Direct.exe` | Route A 固定单轴运动与采集 | 仅需要该项运动时 |
| `MotionAcquisitionV9SilController.exe` | 主窗口离线辅助程序 | 随包保留，无需手动打开 |
| PEAK 驱动及 PCAN-Basic | 让上位机访问 PCAN 兼容 USB-CAN | 真实 CAN 采集必需 |
| Servo Studio | 驱动器调试及由它负责运动时使用 | 可选；Direct 运行前退出 |

## 手册

- [01 首次安装](manuals/01_首次安装.md)
- [02 只读采集与固定单轴直控](manuals/02_采集操作.md)
- [03 数据保存与带回](manuals/03_数据保存与带回.md)
- [04 常见问题](manuals/04_常见问题.md)
- [05 实验记录模板](manuals/05_实验记录模板.md)
- [软件来源、版本和验证范围](SOFTWARE.md)
- [第三方材料说明](THIRD_PARTY.md)

## 当前适用范围

沿用 Windows → PCAN 兼容 USB-CAN → CANopen → DS-BVM 的单轴 Route A。Direct 的配置固定为 `PCAN_USBBUS1 / 1,000,000 bit/s / Node 1`，仅支持既有 `+5000 counts → 驻留 → -5000 counts` 流程。现场不一致时不能用 Direct 强行连接。

这里读取的是驱动器反馈；不是自研光栅尺独立采集或精度标定。程序原始数据以 counts 和 raw 值保存。STM32、GM1、EtherCAT 和多轴不属于本工具包的现场操作流程。

上位机版本保持 `V9.0.0 · 20261023-ux2`；`20261031` 是本工具包整理日期。现有包有此前离线验证记录，本次整理不构成实验室电脑驱动兼容或真实采集验证。仓库为私有；原始实验数据另行备份，不提交到本仓库。
