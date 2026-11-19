# Motion Acquisition V4.0 控制协议与适配器

## 1. 边界

V4.0 定义 Windows 与未来实时控制器之间的逻辑契约，但发布版只实现：

- `in_process`：进程内单元 SIL；
- `loopback_tcp`：本机独立模拟 STM32 进程。

发布策略要求 `simulated=true`、`real_hardware_access=false`、`device_write_capability=false`。没有真实 STM32/CAN 写入适配器。

## 2. framing 与握手

每帧为：

```text
4-byte big-endian payload length
UTF-8 JSON payload
```

单帧上限 4 MiB。默认协议为 `4.0`，解码器继续接受 V3 `3.0` envelope。

客户端先发 `HELLO`，声明支持版本；控制器回 `CAPABILITIES`。之后才允许：

```text
COMMAND -> ACKNOWLEDGEMENT
STATUS_REQUEST -> COMMAND_STATUS
POLL -> SNAPSHOT
EVENT_REQUEST -> EVENT_BATCH
DISCONNECT -> SNAPSHOT
异常 -> PROTOCOL_ERROR
```

所有 envelope 含 protocol version、message ID、sequence、sent time、kind、correlation ID 和 payload。

## 3. 能力

`ControllerCapabilities` 包含：

- adapter/controller/boot ID；
- 协议范围和 transport kind；
- axes 与 supported commands；
- 每轴 state/command interface；
- unit、minimum、maximum、max rate；
- responsibility profile；
- 名义状态率、控制器周期配置和命令对账支持；
- simulated、hardware access、device write 的事实声明；
- limitations。

`ControllerEndpointPolicy` 是产品放行策略，不是能力数据的一部分。未来真实适配器可以被能力模型描述，但不能被 V4 发布版启用。

## 4. 职责档案

档案只做描述，不驱动方案专用业务分支：

| 档案 | loop owner | feedback source | drive mode | host command interface |
|---|---|---|---|---|
| Scheme A | drive | drive | position | position |
| Scheme B | STM32 | controller direct | velocity | position |
| Scheme C fixture | external/待定 | external sensor | velocity | velocity |

Windows 业务层只检查被声明的接口是否存在、值/rate 是否在范围内。

## 5. 命令与任务

V3 位置命令和 `MotionTask` 继续解码。V4 新增：

```json
{
  "axis_id": "X",
  "interface": "position|velocity|effort|custom",
  "value": 1.0,
  "rate_limit": 2.0
}
```

`MotionProgram` 由一个或多个 `ProgramSegment` 组成；每段包含若干轴/接口 setpoint、可选 duration、dwell、tolerance 和 timeout。非位置连续命令必须提供 duration。

V4 业务命令不公布 `emergency_stop` 或 `reset_emergency_stop`。旧枚举只为历史解码保留。普通停止动作是：

```text
request_protective_stop
```

它不是急停、STO 或安全功能。

## 6. ACK 与命令状态

ACK 只表示控制器接收或拒绝命令。终态使用独立生命周期：

```text
submitted / accepted / queued / running /
completed / cancelled / failed / rejected / unknown
```

ACK 在提交后立即记录，后续 status/poll 超时不能抹掉已接收证据。控制器 boot ID 或连接变化后，无法对账的命令必须是 `unknown`，不能猜成 completed。

## 7. 状态与安全

每个 `InterfaceValue` 含：

- name、value、unit；
- source；
- validity：valid/stale/unknown/invalid；
- controller-side sampled monotonic time。

通信丢失时保留最后值并改为 stale，不把速度伪造为 0。

`SafetySnapshot` 独立于 controller/axis/task 状态，包含：

- safety chain state；
- physical E-stop/STO：允许 `unknown`；
- protective stop；
- source；
- `safety_rated`。

SIL 永远报告 `safety_rated=false`，物理 E-stop/STO 不可由模拟结果推断。

## 8. 适配器接口

```python
class ControllerAdapter(Protocol):
    capabilities
    negotiate()
    connect()
    disconnect()
    submit(command)
    query_command_status(command_id)
    poll()
    drain_events()
```

故障注入仍在独立 `FaultInjectionPort`。远程 SIL 通过专用 diagnostic frame 实现；普通服务不依赖注入接口。

## 9. 真实控制器延期项

以下未被 V4 定义为现场事实：

- PC/STM32 最终物理传输与认证；
- 实际控制周期、遥测率、时间同步和缓冲；
- 驱动器对象字典、CiA 402 mode、PDO；
- 反馈接法、单位与标定；
- 急停/STO/制动/限位/故障复位权限；
- 断连与 STM32 重启后的真实动作。

启用真实适配器前必须以硬件和安全方案证据另行放行。
