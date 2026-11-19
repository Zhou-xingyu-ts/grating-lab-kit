# Motion Acquisition V6.0 Windows 部署

## 普通 GUI 用户

复制整个 `MotionAcquisition` onedir，双击 `MotionAcquisition.exe`。不要只复制
单个 EXE。运行 GUI 不需要 Python、CMake、Ninja、Zig 或 Arm GCC。

用户配置和实验数据继续位于当前 Windows 用户自己的配置/数据目录，不依赖
固定用户名、盘符或源码路径。

## 固件开发者

发布包中的 `MotionControllerFirmwareV6` 是独立开发成果：

- ELF/HEX/BIN/map/build report；
- 可复现 source snapshot；
- CMSIS 最小依赖和许可证；
- 构建与未来上板清单。

重新交叉构建需要 Arm GNU Toolchain 15.2.Rel1、CMake 和 Ninja。工具链路径
通过 `ARM_GNU_TOOLCHAIN_ROOT` 或构建脚本参数传入，不写死。

## 当前限制

- 未在第二台 Windows 电脑验证；
- 未在真实 H743 板卡启动或调试；
- 未连接 CAN、驱动器、电机、反馈或安全 IO；
- 固件 artifacts 不得因“可以生成”而直接烧录；
- 真实部署必须先完成 `V6_0_HARDWARE_BRINGUP_CHECKLIST.md` 的 Gate 0。
