# Motion Acquisition V9.0.0 三路线配置说明

## 1. 配置的安全原则

V9 配置是“路线契约与权限声明”，不是设备发现工具。读取、列举或校验配置不得打开 CAN、串口、USB、网络、GM1、STM32 或 EtherCAT。

所有示例必须默认：

~~~yaml
offline_mode: true
real_hardware_access: false
device_write_capability: false
~~~

配置缺失、字段不一致、身份不匹配、超时、重启、权限不足或状态无法确认时，路线必须保持 unavailable 或 unknown。禁止填入猜测的 COM 口、波特率、IP、Node ID、寄存器、引脚、收发器或 PDO 映射来“补全”可运行配置。

## 2. 统一配置字段

| 字段 | 含义 | 离线默认要求 |
|---|---|---|
| route_id | 固定物理路线 ID | 必须为三个白名单 ID 之一 |
| enabled | 是否允许 Route Manager 展示/创建离线会话 | 可为 true，但不代表可连接设备 |
| transport_kind | host transport 的类型 | 仅 Fake/Mock/SIL 可启用 |
| real_hardware_access | 是否允许真实端点 I/O | 必须 false |
| device_write_capability | 是否允许设备写 | 必须 false |
| authority_level | locked、read_only 或 control_armed | 安全默认 locked |
| configuration_evidence | 配置来源、版本、缺口 | 必填，未知写 unknown |
| time_semantics | host/设备时间戳与同步边界 | 不得声称已同步 |
| stop_strategy | 路线专属关闭策略名 | 必须明确，不能使用空泛通用替代 |

配置解析器应拒绝：

- 未知 Route ID；
- 同时两个 active master；
- Online/real endpoint 与 offline_mode=true 的矛盾组合；
- 缺失身份、权限或预检条件时的控制请求；
- 将历史闭环职责档案 SCHEME_A/B/C 当作物理 Route ID。

## 3. 推荐的安全默认结构

下列结构表达 V9 配置意图。字段名称应以实际实现的 schema 为准；未被解析器接受的字段不得靠人工解释生效。

~~~yaml
route_manager:
  offline_mode: true
  active_route: null
  shared_motion_master_lock: required

routes:
  windows_pcan_direct:
    enabled: true
    transport_kind: fake_or_replay
    real_hardware_access: false
    device_write_capability: false
    authority_level: locked
    stop_strategy: route_a_halt_then_disable

  stm32_canopen_bridge:
    enabled: true
    transport_kind: in_memory_protocol_4
    real_hardware_access: false
    device_write_capability: false
    authority_level: locked
    node_id: null
    bitrate: null
    real_port: unavailable

  gm1_ethercat_controller:
    enabled: true
    transport_kind: mock_modbus
    real_hardware_access: false
    device_write_capability: false
    authority_level: locked
    gm1_contract: gm1_route_v1
    real_endpoint: unavailable
~~~

上述不是现场配置，也不包含可用的 PCAN 设备名、COM 口、IP、Node ID、寄存器或引脚。

## 4. Route A 配置

### 4.1 离线允许项

- fake SDO 写序列验证；
- 历史会话回放、checksum 和 16 列 CSV 完整性检查；
- 固定单轴计划和收尾逻辑的静态/单元级回归；
- Route Manager 对既有 DirectMotionAcquisitionService 的委托描述。

### 4.2 受保护的现场基线

历史现场基线中的 PCAN_USBBUS1、Node 1、1 Mbps、固定 +5000/-5000 counts 和七对象采集，是 Route A 已有受限证据的属性。V9 离线配置不得因此自动打开 PCAN，也不得扩展为任意轨迹、多轴或新控制模式。

配置必须保证：

- 真实 PCAN 入口在 Offline/SIL 中锁定；
- Route A 使用同一个既有 CANopen client 供控制与采集；
- 正常结束、主动停止和异常结束均采用既有 Route A 专属收尾；
- 无法确认 Disabled 时，active route 变为 unknown，不能切换。

## 5. Route B 配置

### 5.1 主机—STM32 传输

| 配置项 | 安全要求 |
|---|---|
| transport_kind | Fake、loopback 或明确的 serial；未明确即 unavailable |
| serial_port / baudrate | 无猜测默认值；缺一个即 unavailable |
| protocol_version | 固定与 protocol 4.0 兼容 |
| expected_boot_id | 只用于对账；变化后未确认命令转 unknown |
| reconnect policy | 不得自动重发运动命令 |

串口实现可支持 USB CDC、ST-LINK VCP 或 UART 的 COM 端口形态，但模块导入、GUI 启动和路线枚举都不得打开串口。

### 5.2 固件与 CANopen profile

安全默认镜像必须至少满足：

~~~yaml
write_authority: locked
node_id: null
bitrate: null
fdcan_port: unavailable
drive_profile: unassigned
~~~

实验模板可以记载“Node 1 / 1 Mbps”，但必须同时标注“待现场证实，不是生产默认”。identity、heartbeat、EMCY、SDO upload、CiA 402、PPM、PDO/SYNC、诊断、对象能力与周期均应来自受版本控制的配置/契约，不能硬编码为现场事实。

## 6. Route C 配置

### 6.1 GM1 Modbus 契约

真实 Route C 只有在当前 GM1 工程、gm1_route_v1 契约、可核验 build ID 与寄存器映射都已获得证据时才可能进入后续门。V9 默认使用 Mock。

gm1_route_v1 的配置/响应应覆盖：

| 组 | 内容 |
|---|---|
| 身份 | magic、contract version、GM1 build ID、boot counter |
| 健康 | heartbeat、controller state、safety state、per-axis state |
| 数据 | 32 位有符号 raw counts，高字在前 |
| 命令 | command sequence、kind、参数、commit |
| 对账 | ACK sequence、执行阶段、error code、完成状态 |

magic、版本、build ID、心跳、字序或 sequence 不符合时，只能显示诊断；绝不发送命令。

### 6.2 EtherCAT 与 ESI

Route C PC 配置不包含 Windows EtherCAT master。ESI/XML 原件只读；审计输出必须列出 identity、PDO、DC、InitCmd、SafeOp→OP 相关写、对象映射和未闭合项。历史 GM1 IP、线圈/寄存器地址与位置偏移算法只可作为审计线索。

## 7. 配置与会话元数据

每次显式保存的离线预检记录或控制记录至少附带：

~~~yaml
route:
  route_id: [白名单 Route ID]
  host_transport: [offline_replay_or_fake_sdo/in_memory_protocol4/in_memory_gm1_mock]
  fieldbus: [canopen/ethercat]
  master_owner: [Windows/STM32H743/GM1]
  adapter_capability_snapshot: [版本化摘要]
  time_semantics: [host-only/virtual/unknown]
  permission_state: locked
  preflight: [passed/state/checks/summary]
  real_hardware_access: false
  device_write_capability: false
~~~

旧 control_route=windows_pcan_ds_bvm 继续映射为 Route A。可选 route 字段的缺失不能使历史会话失效，也不能放宽新写会话的完整性要求。

Route Manager 只在用户明确点击“保存离线预检记录”后才把上述字段写入 `route_preflight_records/`；该记录必须保持零设备 I/O、零样本值和两个 false 边界标志。
