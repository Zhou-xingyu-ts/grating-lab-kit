# Motion Acquisition V2.0 Windows 部署

## 复制与启动

1. 在构建电脑取得整个 `release\MotionAcquisition` 目录。
2. 将整个目录复制到目标 Windows x64 电脑本地磁盘。
3. 不要只复制 EXE，也不要在压缩包内直接运行。
4. 双击 `MotionAcquisition.exe`。
5. 首次启动保持默认 Demo，完成一次短采集、停止、扫描、质量评估、回放和离线算法冒烟。

运行不需要 Python、源码或构建盘符。

## 用户独立目录

- 设置、项目、预设、换算、标定和实验组位于当前用户的 Qt `AppConfigLocation`。
- 默认数据目录位于当前用户 Documents 下的 `MotionAcquisitionData`。
- 用户可以在 GUI 选择其他有写权限的数据根目录。
- 历史会话和算法结果使用相对路径/哈希快照，不依赖构建用户名。

跨用户使用时各自拥有独立配置。复制数据根目录可以迁移会话；实验组引用要求保持相对目录结构和 CSV 内容。

## 真实 PCAN 前置条件

真实只读采集还需要目标电脑正确安装 PEAK PCAN 驱动并现场确认：

- PCAN_USBBUS1；
- 1 Mbps；
- Node-ID 1；
- CANH/CANL/GND 和终端；
- 驱动器/平台安全条件；
- Servo Studio 独立运动操作边界。

程序不安装驱动，也不会把 Demo 成功解释为 PCAN 成功。无现场授权时不要选择真实模式。

## 安全边界

- 只有 CANopen SDO upload；
- 没有 SDO download、NMT/PDO、原始 CAN 写帧、EtherCAT 或运动命令；
- 实时和离线影子不发送输出；
- 停止采集不停止电机；
- raw 16 列 CSV 始终权威。

## 建议目标机检查

- EXE 能启动且标题显示 V2.0；
- 默认 Demo；
- 当前用户目录可写；
- Demo completed 与 stopped 会话完整；
- 会话扫描、回放、质量报告、成组评价和算法运行可用；
- `samples.csv` 为 16 列且前后 SHA-256 不变；
- 发布目录未依赖源码目录；
- 关闭后无残留研究/采集线程。

## 当前验收声明

V2.0 发布包只在构建电脑完成离线构建和冒烟。未在第二台 Windows 电脑或干净虚拟机实测，因此不能声明跨电脑验收已经完成。真实 PCAN、运动配合、外部标定和设备同步同样延期。
