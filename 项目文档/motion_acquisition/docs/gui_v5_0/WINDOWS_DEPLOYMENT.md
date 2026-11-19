# Motion Acquisition V5.0 Windows 部署

## 发布目录

必须整体复制：

```text
MotionAcquisition/
  MotionAcquisition.exe
  MotionAcquisitionSilController.exe
  MotionAcquisitionControllerReference.exe
  README_FIRST.txt
  RELEASE_INFO.txt
  SHA256SUMS.txt
  _internal/
  docs...
```

- 主 EXE：GUI；
- `SilController`：Python A/B/C 软件在环与故障注入；
- `ControllerReference`：V5 原生 C 核心 Scheme-A fixture。

三个 EXE 都不能单独脱离目录使用。

## 另一台 Windows 电脑

1. 复制整个 `MotionAcquisition` 目录；
2. 用 `SHA256SUMS.txt` 核对三个 EXE；
3. 双击主 EXE；
4. 先运行 Demo；
5. 再分别测试 Python 跨进程 SIL 和 V5 原生参考进程；
6. 不连接 CAN/USB，仅验证启动、保存、回放和控制记录；
7. 若 SmartScreen 拦截，按单位安全策略处理，不关闭系统安全功能。

发布包不需要 Python、CMake、Zig 或源码。构建工具只存在于开发电脑。

## 用户目录

配置与数据继续按 Windows 当前用户保存，不依赖固定用户名、盘符或源码路径。复制发布目录不会带走开发电脑的实验数据。

## 当前验证边界

本项目只完成构建电脑本机验收，不能宣称已在第二台电脑验证。PCAN 驱动、真实 STM32/CANopen、驱动器、运动、安全、同步与精度均需另行现场验证。

