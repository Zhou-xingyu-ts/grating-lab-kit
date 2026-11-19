# Motion Acquisition V7.0 Windows 部署

1. 将整个 `MotionAcquisition` 目录复制到另一台 64-bit Windows 电脑；
2. 不要只复制 EXE；
3. 双击 `MotionAcquisition.exe`；
4. 先运行 Demo、停止、保存、回放、分析和 SIL；
5. PCAN 真实采集还需要目标电脑安装匹配的 PEAK 驱动；
6. H743 固件包不会自动烧录，也不会自动访问 USB/CAN。

用户配置与数据继续按 Windows 用户目录保存，不依赖源码路径、固定用户
名或盘符。

本机 PyInstaller 冒烟不能代替第二台电脑验证。第二台电脑仍需记录：

- Windows 版本与架构；
- GUI 启动；
- Demo/停止/保存；
- 回放/分析；
- Python SIL/native C SIL；
- 杀毒软件和依赖提示；
- 如需 PCAN，再单独验证只读硬件链路。

