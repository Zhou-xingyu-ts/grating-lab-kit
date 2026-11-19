# V3.0 控制协议与适配器边界

## 目标

V3.0 定义 PC 与未来实时控制器之间稳定的逻辑模型，同时只实现进程内模拟器。协议是 transport-neutral 的 JSON-safe 语义，不决定未来使用 TCP、共享内存、现场总线或其他物理传输。

## 责任划分

PC：

- GUI、项目和设备视图；
- 命令参数和软限位预检查；
- 命令序号、连接关联和超时判断；
- 采集关联、控制记录、回放和分析。

未来实时控制器：

- 确定性现场通信；
- 位置环、轨迹和同步执行；
- 真实限位、互锁、急停和安全状态；
- 周期反馈与控制器事件。

V3 模拟器只近似后者的软件语义，不证明其物理实现。

## 消息封装

所有逻辑帧使用协议版本 `3.0`：

```json
{
  "protocol_version": "3.0",
  "message_id": "msg-...",
  "sequence": 17,
  "sent_at": "ISO-8601",
  "kind": "command|acknowledgement|snapshot|event|heartbeat",
  "correlation_id": "cmd-...",
  "payload": {}
}
```

编码为 UTF-8 JSON，禁止 NaN/Infinity，键顺序规范化以便测试和后续签名。命令还包含 `command_id`、单调序号、`expected_connection_id` 和超时值。确认明确记录 `accepted`、代码、消息、控制器序号和连接 ID。

## 状态模型

轴状态：

```text
disconnected -> disabled -> enabled
enabled -> homing/jogging/moving -> enabled
任意活动状态 -> fault
任意已连接状态 -> emergency_stop
```

控制器状态独立为：

```text
disconnected / connecting / ready / running /
fault / emergency_stop / communication_lost
```

任务状态独立为：

```text
idle / queued / running / completed / cancelled / failed
```

GUI 不能通过按钮状态推断真实轴状态，只能渲染控制器快照。

## 命令

| 命令 | 主要参数 | 关键门禁 |
|---|---|---|
| `enable_axis` / `disable_axis` | axis | 已连接、无急停/故障 |
| `reset_fault` | axis | 轴存在故障 |
| `home_axis` | axis | 轴已使能 |
| `jog_start` / `jog_stop` | axis, direction, velocity | 已使能、已回零、速度合法 |
| `move_absolute` | axis, position, velocity | 已使能、已回零、目标在软限位 |
| `move_relative` | axis, distance, velocity | 当前位置加增量在软限位 |
| `stop_axis` / `stop_all` | axis/none | 已连接 |
| `emergency_stop` / `reset_emergency_stop` | none | 复位后仍保持去使能 |
| `execute_task` / `cancel_task` | `MotionTask` | 所有参与轴就绪、目标合法、任务互斥 |

## 适配器接口

业务层只依赖：

```python
class ControllerAdapter(Protocol):
    capabilities
    connect()
    disconnect()
    submit(command)
    poll()
    drain_events()
```

V3 的 `ControllerCapabilities` 构造器拒绝：

- `simulated=false`；
- `device_write_capability=true`；
- `real_hardware_access=true`。

发布包只包含 `SimulatedControllerAdapter`。故障注入另由 `FaultInjectionPort` 提供，正常业务层不依赖它。

## 未来替换真实适配器

未来硬件版本应新建独立适配器并满足同一模型，不修改 GUI 或控制服务。启用前必须另行确定真实传输、身份认证、重放保护、时钟、更新率、超时、幂等性、设备字典、安全状态和现场验收方案。

V3.0 不包含这一实现，也不得把逻辑协议文档解释成现场协议已经确定。
