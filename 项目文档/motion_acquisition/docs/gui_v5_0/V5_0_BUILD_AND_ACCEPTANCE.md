# Motion Acquisition V5.0 本机构建与验收

## 离线范围

V5.0 本机验收包括：

- portable C core、semantic DrivePort 和 virtual drive；
- 原生 Windows loopback reference process；
- 现有 protocol 4.0 adapter 无改写互操作；
- Python A/B/C SIL 与原生 Scheme-A fixture；
- GUI 控制记录、采集、回放、研究和历史兼容；
- PyInstaller Windows `onedir`。

明确不包括：

- STM32 编译、下载或运行；
- FDCAN/CAN transceiver/CANopen master；
- DS-BVM 对象/PDO/CiA 402 控制；
- 三轴物理运动、安全、时序、同步或精度；
- 第二台 Windows 电脑。

## 控制器构建

```powershell
powershell -ExecutionPolicy Bypass -File packaging\build_native_controller.ps1
```

当前已验证：

- CTest：1/1；
- `MotionAcquisitionControllerReference.exe`：223,232 bytes；
- SHA-256：`F59EBD37E7BFBC785E171D61F529C73D3C132BE1FAD81DFB751D03C267EB6FF2`。

## 自动化与持久证据

- Python 全量：`145 passed in 20.45s`；
- `compileall`：通过；
- `pip check`：无损坏依赖；
- V5 控制/协议/GUI目标子集：21/21；
- safety/portability/native 子集：16/16；
- 采集历史：19/19 compatible，212,378 条 replay records，零源 CSV hash 变化；
- V3/V4 durable control records：7/7 checksum/analyze；
- V5 native durable sessions：3/3 checksum/analyze：
  - `completed`：1000 count setpoint；
  - `stopped`：运行中非安全 protective stop；
  - `error`：原生进程丢失后 communication_lost/stale。

## Windows onedir

最终目录：

```text
release\MotionAcquisition
```

- 文件数：389；
- 总大小：172,394,627 bytes（164.41 MiB）；
- `MotionAcquisition.exe`：
  `E2EC086697C172B5827B12B16E9EFA54F2AB1D1959C216C6026C8ABCF3E66692`；
- `MotionAcquisitionSilController.exe`：
  `B21CA7C3C7716DAB654E0E3F295BDF0FD4ED591AF807D741A60A7008A7504751`；
- `MotionAcquisitionControllerReference.exe`：
  `F59EBD37E7BFBC785E171D61F529C73D3C132BE1FAD81DFB751D03C267EB6FF2`。

三条哈希均与发布目录 `SHA256SUMS.txt` 复算一致。静态/二进制扫描发现：

- 固定用户名、构建盘符、源码路径：0；
- editable dependency 路径：0；
- 误带 acquisition/control session：0；
- V0～V4 旧 GUI 发布文档：0。

## 发布冒烟

- 主 GUI 在隔离 APPDATA/LOCALAPPDATA、offscreen 下保持运行 5 秒；
- 未连接前自动启动 helper 数：0；
- 发布版 native helper：capability → connect → enable → 100 count setpoint → completed → protective stop → disconnect，通过；
- 发布版 Python helper：capability → connect → command → protective stop → disconnect，通过；
- 两个 helper 都报告 `real_hardware_access=false` 和 `device_write_capability=false`，正常退出 0。

## 本机离线结论

V5.0 通过当前构建电脑的离线软件验收。它证明 portable controller core、真实跨语言帧边界、上位机集成、记录和发布闭环；不证明下面任何现场事项：

- H743 board/RTOS/HAL/FDCAN；
- CAN transceiver、CANopen master、DS-BVM 对象/PDO；
- 真实反馈、三轴运动、物理 E-stop/STO；
- 真实周期、同步、精度或第二台 Windows 部署。

## 软件证据边界

原生进程通过真正的 C/Python 序列化边界并不等于真实 STM32 通信。100 Hz、count 范围、virtual position follow 和 protective stop 都只是 fixture 行为。
