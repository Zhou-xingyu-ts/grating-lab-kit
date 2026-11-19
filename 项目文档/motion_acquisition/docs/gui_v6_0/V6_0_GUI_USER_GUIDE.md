# Motion Acquisition V6.0 GUI 使用教程

V6.0 的桌面操作沿用 V5.0，没有增加真实设备控制按钮。新增成果主要位于
STM32H743 固件工程、离线构建和未来上板边界。

## 启动

1. 保留发布目录全部文件；
2. 双击 `MotionAcquisition.exe`；
3. 默认使用 Demo，不会自动连接 CAN、STM32 或驱动器；
4. 采集、回放、会话、分析、比较和研究流程继续按既有页面使用。

## 三轴控制页面

可选择：

- 进程内 Python SIL；
- 跨进程 Python SIL；
- `MotionAcquisitionControllerReference.exe` 原生 C reference。

这三种都是本机软件模型。页面上的 enable、Jog、home、reset、position task 和
protective stop 不会发往 STM32 或驱动器。

V6 的 `MotionControllerFirmwareH743.elf/.hex/.bin` 是开发构建产物，不是
Windows 可执行文件，GUI 当前也不会连接或烧录它们。

## 只读采集

- Demo 可直接使用；
- PCAN/CANopen 现场路径仍只允许 SDO upload；
- 停止采集只停止读取，不停止 Servo Studio 或电机；
- raw counts 和 16 列 `samples.csv` 始终权威。

## 数据和兼容

项目、预设、事件、换算、回放、分析、比较、研究输出和 V3～V5 control
records 均保持兼容。V6 不修改现有会话 schema。

更完整的既有 GUI 操作可参见随包保留的
`V5_0_GUI_USER_GUIDE.md`；V6 固件与上板边界参见：

- `V6_0_ARCHITECTURE_AND_FIRMWARE.md`；
- `V6_0_PROTOCOL_AND_PORT_GUIDE.md`；
- `V6_0_HARDWARE_BRINGUP_CHECKLIST.md`。
