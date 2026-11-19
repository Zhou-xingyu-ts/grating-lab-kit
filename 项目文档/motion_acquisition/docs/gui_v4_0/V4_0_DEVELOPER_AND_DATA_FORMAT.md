# Motion Acquisition V4.0 开发与数据格式

## 1. 控制模块

```text
control.models          V3 兼容模型 + V4 接口/状态/程序/安全/生命周期
control.adapter         中立能力、适配器 Protocol、发布 endpoint policy
control.profiles        Scheme A/B/C SIL 能力档案
control.protocol        3.0/4.0 envelope + 4-byte framing
control.service         capability-driven 主机监督与证据顺序
control.simulator       进程内确定性 SIL
control.remote_adapter  数字回环地址限定的同步客户端与 helper 生命周期
control.sil_process     独立模拟 STM32 helper 入口
control.recorder        V3 文件兼容的追加式控制会话
desktop.control_worker  socket/service/recorder 所有权
desktop.control_workspace capability-driven GUI
```

采集、回放、V2 研究模块未并入控制域。

## 2. 发布安全策略

中立 `ControllerCapabilities` 不拒绝描述未来真实端点。V4 产品由 `ControllerEndpointPolicy` 拒绝：

- 非 `in_process|loopback_tcp`；
- `simulated=false`；
- `real_hardware_access=true`；
- `device_write_capability=true`。

只有 `remote_adapter.py` 和 `sil_process.py` 可以导入 socket；控制业务代码不得导入 CANopen client、python-can、serial 或硬件传输。

## 3. 能力和接口

`AxisInterfaceCapabilities` 把 state 与 command interface 分开。业务代码按 descriptor 验证 name/unit/range/rate，禁止按 scheme ID 分支。

兼容字段 `AxisSnapshot.position/velocity` 继续存在；V4 权威状态语义在 `interface_values[]`：

```json
{
  "name": "position",
  "value": 1.25,
  "unit": "mm",
  "source": "drive",
  "validity": "valid",
  "sampled_monotonic_s": 12.5
}
```

## 4. 协议兼容

- 新消息编码为 4.0；
- envelope 解码接受 3.0 和 4.0；
- V3 snapshot 缺 interface values 时，解码器保守补为 source=`legacy_v3`；
- V3 emergency flag 不升级成物理 E-stop 证据；
- 未知字段由 JSON 读取方按可选扩展处理。

## 5. 控制会话

schema 4.0 保留 V3 文件并新增：

```text
command_statuses.jsonl
```

`control_manifest.json.controller` 保存完整协商能力与职责档案。`evidence_boundary` 明确：

```json
{
  "software_in_the_loop": true,
  "real_hardware_access": false,
  "device_write_capability": false,
  "physical_safety_validated": false,
  "physical_accuracy_validated": false,
  "physical_synchronization_validated": false,
  "physical_estop_status_available": false,
  "pc_stop_is_safety_rated": false
}
```

V3 schema-3.0 控制会话没有 status stream 时仍可校验和分析。

## 6. ACK 写入顺序

```text
record command
submit
record ACK immediately
query status
poll snapshot
drain events
```

这样 status/poll 超时不会抹掉已接受证据。命令终态从 status/snapshot/event 获取，不能从 ACK 推断。

## 7. 跨进程 helper

源码运行时，主程序启动：

```text
python -m motion_acquisition.control.sil_process
```

发布包运行时启动同目录：

```text
MotionAcquisitionSilController.exe
```

helper 通过当前用户临时目录中的原子 ready file 返回随机回环端口。GUI 拥有 helper 生命周期；worker 拥有 socket。退出先断开 socket，再等待/终止自己创建的 helper。

## 8. 回归

核心测试覆盖：

- V3 decode/V4 encode；
- frame 长度与协商；
- Scheme A/B/C profiles；
- position/velocity program；
- command lifecycle；
- stale 反馈；
- 非安全保护性停止；
- 回环地址限制；
- 独立进程、超时、断开；
- ACK-before-status-timeout；
- V3 控制记录和 19 组采集数据兼容；
- 无 CAN/硬件写入。

持久 SIL：

```powershell
.\.venv\Scripts\python.exe .\scripts\v4_acceptance_smoke.py <new-output-dir>
```

脚本拒绝覆盖已存在目录，只生成软件在环控制会话。
