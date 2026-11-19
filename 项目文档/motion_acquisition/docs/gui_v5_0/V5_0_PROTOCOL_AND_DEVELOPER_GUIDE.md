# Motion Acquisition V5.0 协议与开发指南

## 版本关系

- 产品版本：5.0.0；
- 控制协议：4.0；
- 兼容解码：3.0 envelope；
- 采集 CSV：原 16 列不变；
- V3/V4 控制记录：不改 schema 已有字段。

V5 没有为了产品版本升级而制造 wire protocol 5.0。原生进程必须通过现有 `RemoteSilControllerAdapter`，这正是兼容验收的一部分。

## 原生进程

```text
MotionAcquisitionControllerReference.exe
  --ready-file <temporary ready.json>
  --profile scheme-a-drive-position-loop
```

仅绑定 `127.0.0.1`，一次服务一个连接并在 DISCONNECT 后退出。ready 文件同时声明：

```json
{
  "simulated": true,
  "real_hardware_access": false,
  "device_write_capability": false
}
```

## 帧与请求

帧仍为：

```text
4-byte unsigned big-endian payload length
UTF-8 JSON ProtocolEnvelope
```

最大 payload 为 4 MiB。空帧、超长帧、无效 JSON、非对象 payload、未知请求和协议版本不匹配返回 `protocol_error`。自动测试覆盖逐字节分片。

实现的请求：

| 请求 | 响应 |
|---|---|
| HELLO | CAPABILITIES |
| POLL connect/poll | SNAPSHOT |
| COMMAND | ACKNOWLEDGEMENT |
| STATUS_REQUEST | COMMAND_STATUS |
| EVENT_REQUEST | EVENT_BATCH |
| DISCONNECT | SNAPSHOT 后退出 |

未声明 diagnostic、task、Jog、home 或 raw bus 命令。

## 命令生命周期

ACK 只表示控制器是否接受语义意图。terminal status 独立查询：

```text
accepted
running
completed / cancelled / failed / rejected / unknown
```

重复 `command_id` 不重复应用 DrivePort intent，而返回 `DUPLICATE_RECONCILED`。`expected_connection_id` 不匹配返回 `STALE_CONNECTION`。bounded history 之外返回 `unknown`。

## C 核心

公共入口：

```c
mc_core_init(...)
mc_core_connect(...)
mc_core_tick(...)
mc_core_submit(...)
mc_core_query_status(...)
mc_core_drain_events(...)
mc_core_disconnect(...)
```

核心使用固定容量，不分配内存。Windows/JSON/socket 和 virtual plant 均在 port 层。未来固件只应替换 DrivePort、时间/调度和 PC transport。

## 构建

```powershell
powershell -ExecutionPolicy Bypass -File packaging\build_native_controller.ps1
```

脚本自动发现 CMake、Ninja 和 Zig。当前 host 编译器只是 Windows 软件在环工具，不是已选择的 STM32 cross-toolchain。

## 测试层

```text
CTest
  core lifecycle/setpoint/duplicate/stale connection/protective stop
  deadline miss/drive fault/read failure

pytest
  existing Python adapter ↔ native process
  fragmented/invalid/version/oversize frames
  process restart/boot identity/process loss/stale state
  GUI native connect/enable/setpoint/save
  full V1–V4 regression and compatibility
```

## STM32 接入

参见 `STM32H743_PORT_CONTRACT.md`。在真实对象字典、板卡、安全和反馈证据关闭前，不得新增 flashable target、CANopen write profile 或真实 endpoint 放行。

