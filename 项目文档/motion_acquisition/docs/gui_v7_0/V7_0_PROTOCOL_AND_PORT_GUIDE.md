# V7.0 控制器协议与端口指南

## PC 协议

V7 延续 protocol 4.0，不改变既有桌面模型：

```text
4-byte big-endian JSON length
JSON envelope:
  protocol_version
  message_id
  sequence
  sent_at
  kind
  correlation_id
  payload
```

请求：`hello`、`poll(connect/status)`、`command`、`status_request`、
`event_request`、`disconnect`。

响应：`capabilities`、`snapshot`、`acknowledgement`、
`command_status`、`event_batch`、`protocol_error`。

嵌入式编码器不使用 heap、stdio 或动态 JSON 库。响应队列固定为 2 帧，
单帧 JSON 上限 8192 bytes。ByteStreamPort 可以 partial write；队列满时
runtime 暂停读取，直到有响应槽位。

## 能力与责任

能力响应继续包含：

- state interfaces 与 command interfaces；
- `loop_owner`、`feedback_source`、`drive_command_mode`；
- `deterministic_execution_owner=stm32`；
- `physical_safety_authority=external_hardware`；
- `real_hardware_access` 与 `device_write_capability`。

生产 H743 target：

- `simulated=false`；
- `supported_commands=[]`；
- profile=`hardware-profile-unconfigured`；
- real hardware/write 均为 false。

虚拟 fixture：

- `simulated=true`；
- profile=`scheme-a-drive-position-loop-virtual`；
- 允许 semantic commands，但只作用于内存虚拟驱动器。

## 端口

| 端口 | 状态方向 | 命令方向 | V7 H743 |
|---|---|---|---|
| ByteStreamPort | PC bytes in | response bytes out | unavailable |
| CANopenStackPort | CAN frames / node state | read SDO 与 control 分权发送 | unavailable |
| DrivePort | axis state | semantic intent | unavailable |
| FeedbackPort | 单一反馈样本 | 无 | unavailable |
| SafetyPort | external stop/STO/watchdog observation | 无安全输出 | unavailable |

所有 target port 为空不代表协议不可用；相同 codec/runtime 已在 stdio
虚拟 byte stream 中通过 Python 双向验证。

## CANopen 主站

V7 实现以下 CiA 301 标准行为：

- 三个可配置节点槽；`0` 表示 unassigned；
- boot-up/heartbeat NMT state；
- heartbeat timeout；
- EMCY code 与计数；
- expedited SDO upload；
- SDO abort/timeout/成功/失败计数；
- 显式 NMT start/stop/pre-op/reset frame；
- 读发送与 control/NMT 发送两个独立 authority。

内置只读 probe 列表：

`0x1000:0`、`0x1001:0`、`0x1018:1..4`、`0x6041:0`、
`0x6061:0`、`0x6064:0`。

这些对象来自标准/系列手册语义，不代表现场驱动器已支持。生产 H743
配置为 node IDs `{0,0,0}`、SDO upload disabled、NMT disabled，且 port
无 transmit 函数。

V7 不包含：

- SDO download；
- controlword/mode write；
- PDO mapping/configuration；
- SYNC producer；
- DS-BVM 专用控制序列；
- LSS、固件升级或安全协议。

## 数据所有权

- SysTick ISR 只更新单调时间；
- mainline 是 core、protocol、CANopen manager 的唯一写者；
- CAN ISR/port 将来只能通过固定 32 槽 C11 acquire/release SPSC 队列
  交付完整经典 CAN frame；
- 队列溢出、deadline miss、protocol egress overflow、watchdog stale mask
  都进入诊断，不允许无界追赶。

