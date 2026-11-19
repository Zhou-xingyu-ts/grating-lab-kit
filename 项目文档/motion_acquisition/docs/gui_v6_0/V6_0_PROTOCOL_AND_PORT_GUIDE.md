# Motion Controller Firmware V6.0 协议与端口指南

## 稳定边界

上位机继续使用 protocol 4.0 的四字节大端长度 + UTF-8 JSON envelope。
桌面端允许较大帧；嵌入式端将单帧上限固定为 8192 字节，避免把 4 MiB
桌面上限复制到 MCU。

```text
ByteStreamPort
→ FrameStream
→ EmbeddedProtocol decoder / typed dispatcher
→ mc_core_t
→ logical DrivePort
```

H743 当前 `ByteStreamPort` 不可用，因此没有串口、USB 或 Ethernet 引脚假设。
decoder 和 typed response 已进入 ARM 镜像；选定真实 transport 后还需实现
响应 envelope encoder/egress。

## request 语义

| kind | 必要字段 | typed 动作 |
|---|---|---|
| `hello` | `supported_protocol_versions` 包含 `4.0` | 返回无硬件/无写能力 capability |
| `poll` | `operation=connect/status` | 建立逻辑 connection 或返回 snapshot |
| `command` | command ID、正 sequence、kind；按命令提供 axis/parameters | 调用 `mc_core_submit` |
| `status_request` | command ID | 查询有界 reconciliation history |
| `event_request` | object payload | 有界 drain events |
| `disconnect` | object payload | core disconnect，并标记 response 后关闭 |

未知版本、JSON、kind 或 payload 返回显式 decode error，不产生设备动作。

## 端口

### `mc_byte_stream_port_t`

- `state`：unavailable/ready/degraded/fault；
- `read`、`write`：只处理 PC 字节流；
- 不知道 JSON、任务、CANopen 或驱动器。

### `mc_canopen_stack_port_t`

- `process_fast_receive`；
- `process_realtime`；
- `process_mainline`；
- `observe_node`。

接口故意不包含 raw transmit、SDO download、NMT command、PDO mapping、
controlword 或 mode write。未来写权限只能在另一个经过对象字典和安全审查的
drive profile 内实现，不能从 PC 暴露原始总线 API。

### `mc_drive_port_t`

核心只使用语义 intent：

```text
enable / disable / reset_fault
position_setpoint
stop / protective_stop
```

V6 H743 target 的 apply/read 均返回 `MC_RESULT_UNAVAILABLE`。Scheme A/B/C
可通过 logical DrivePort facade 更换闭环位置、反馈来源和驱动模式。

### `mc_feedback_port_t`

返回 raw counts、速度、来源码、采样时刻和 validity。V6 只做健康诊断，
不在 unavailable 状态下生成数值。

### `mc_safety_port_t`

读取 external stop、STO feedback 和 watchdog health 的带有效性快照。
V6 不知道引脚、极性、电平或安全动作，因此不会据此宣称安全停止。

## 调度

`mc_embedded_runtime_step(now_us)` 使用绝对 next-deadline：

- 到期只执行一次；
- 跳过的周期计入 miss；
- 不做无界 while 追赶；
- fast receive hook 每次 step 被服务；
- PC 每次 mainline 最多读取 128 字节；
- frame、event、status 和 node 数量均固定有界。

目标中的 1 ms/10 ms/100 ms 是 bring-up 软件配置，不是 H743 或 CANopen
现场性能证明。

## debugger 诊断区

全局符号：

```text
g_motion_controller_diagnostics
```

位于 D3 RAM `.diagnostics` NOLOAD section，记录：

- 版本、reset 次数和 last fault；
- hardware/write/board mapping 三个 false flag；
- PC/CANopen/feedback/safety port state；
- scheduler run/miss；
- frame complete/reject；
- feedback/safety read failure；
- core cycle/deadline miss。

该区便于未来 SWD 首次上板检查，但当前未经过真实 debugger 验证。
