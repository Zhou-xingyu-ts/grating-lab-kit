# 时间语义与统一数据合同

> 本阶段只定义未来合同，不修改`motion_acquisition`。现有CSV仍按原字段解释，不能倒填不存在的逐对象时间戳。

## 1. 当前字段事实

当前`motion_acquisition`在一轮开始时调用一次`Timebase.now()`，随后顺序执行多个SDO upload；同一`sample_seq`的对象共享`wall_time_iso/monotonic_s/elapsed_s`。因此该时间戳近似轮次入口时刻，不是每个对象的真实请求、响应或驱动器采样时刻。

当前`estimated_rate_hz=1/mean_cycle_s`，其中`mean_cycle_s`只含读取与记录工作段，不含等待；它是工作段吞吐口径，不是实际轮次率。当前`deadline_miss_count`只统计工作段耗时大于目标周期，不能证明每轮准时开始、完整deadline或Windows硬实时。

## 2. 合同版本与行模型

- 合同版本：`phase_e_timing_contract_v1`。
- 一行表示一个对象的一次请求结果；命令/轨迹级字段可在同一轮重复，便于长表关联。
- 时间字段统一为同一主机单调时钟的整数纳秒；壁钟只用于人类关联，不参与时延计算。
- 不得用空字符串冒充0；未知事件写空值并在`error_type`或manifest说明。

## 3. 必需字段

| 字段 | 类型/单位 | 精确定义 |
|---|---|---|
| `experiment_id` | string | 一次不可混合配置的实验唯一ID；关联数据、manifest、标定和条件 |
| `cycle_id` | uint64 | 调度轮次递增ID；不表示对象同步采样 |
| `host_monotonic_ns` | int64 ns | 写记录时的通用主机单调时间；不得替代更具体事件字段 |
| `cycle_start_ns` | int64 ns | 主机实际进入本轮调度的时刻 |
| `object_request_ns` | int64 ns | 调用对象读取/提交总线请求前立即记录的主机时刻 |
| `object_response_ns` | int64 ns | 主机收到完整响应或确定失败的时刻 |
| `feedback_available_ns` | int64 ns | 解析后反馈值对控制/记录逻辑可用的时刻；不等于传感器锁存时刻 |
| `command_compute_ns` | int64 ns | 本轮目标/命令完成计算的时刻 |
| `command_tx_ns` | int64 ns | 命令提交给总线接口/驱动通信层的时刻；没有实际发送证据时须注明层级 |
| `object_index` | uint16/hex | CANopen索引或等价对象地址 |
| `object_subindex` | uint8/hex | 子索引 |
| `value` | number/string | 原始对象值；比例换算另有单位/标定ID |
| `success` | boolean | 本次对象事务是否得到可用结果 |
| `error_type` | enum/string | `none/timeout/abort/parse_error/deadline/transport/unknown`等 |
| `source_route` | enum | 如`servo_studio_usb`、`canopen_sdo_upload`、未来`canopen_pdo`/`ethercat`；路线不能混写 |
| `target_position` | count或m | 本轮参考目标；单位和`calibration_id`必须可追溯 |
| `actual_position` | count或m | 反馈位置；注明原始/换算和对象来源 |
| `following_error` | 同位置单位 | 统一定义为`target_position-actual_position`；若驱动对象符号不同需保留原始值另列 |
| `actual_velocity` | count/s或m/s | 可选反馈；未知缩放时保留原始单位，不猜测 |
| `experiment_condition` | JSON/string | 速度、A/D/J、负载、方向、温度、标定ID、软件版本等冻结条件 |

## 4. 强烈建议新增字段

| 字段 | 用途 |
|---|---|
| `scheduled_cycle_start_ns` | 计算轮次起点迟到和相位漂移 |
| `cycle_end_ns` | 定义完整轮次完成时刻 |
| `object_sequence_in_cycle` | 明确顺序SDO中对象次序 |
| `sensor_latch_ns` | 若设备/传感器能提供，测量真正采样时刻；没有则留空 |
| `bus_frame_tx_ns/bus_frame_rx_ns` | 独立抓包或硬件时间戳的帧时刻 |
| `sync_event_ns` | PDO/SYNC阶段的SYNC总线时刻 |
| `drive_target_effective_ns` | 若驱动能提供实际目标生效/回显时刻 |
| `calibration_id`、`unit` | 绑定count—m版本，防止无依据换算 |
| `software_commit_or_hash` | 绑定采集/解析代码版本 |
| `attempt` | 每次重试序号，而非只保留最终尝试数 |

## 5. 同步、轮次率和抖动

### 5.1 同一SDO轮询不是同步采样

对象A完成后才请求对象B。即使共享`cycle_id`，值也对应不同未知设备内部时刻。分析多对象关系时至少使用`object_request_ns/object_response_ns`区间；没有设备锁存时间时，只能称“同轮顺序读取”。

### 5.2 实际轮次率

对$N\ge2$轮：

$$f_{round}=\frac{N-1}{t_{start,N-1}-t_{start,0}}$$

同时报告轮间隔的均值、标准差、P50/P95/P99、最小和最大值。对象率另按成功对象事件计算，不与轮次率混用。

### 5.3 deadline miss

分开定义：

- `cycle_start_late`：`cycle_start_ns > scheduled_cycle_start_ns + start_tolerance_ns`；
- `cycle_completion_miss`：`cycle_end_ns > scheduled_cycle_start_ns + period_ns`；
- `object_deadline_miss`：特定对象的`feedback_available_ns`晚于其预定可用时刻；
- 当前旧字段只可改名解释为`work_segment_overrun`，不追溯性篡改旧CSV。

### 5.4 周期抖动

定义$j_k=cycle\_start_k-scheduled\_cycle\_start_k$。报告均值、标准差、RMS、P95/P99、最大绝对值、随时间漂移和连续miss长度；不能只给平均周期。

## 6. Windows时间戳的证明边界

同一进程中的高分辨率单调时钟可以证明：主机观察到的事件先后、主机代码段耗时、轮间隔和相对抖动。它不能单独证明：

- 编码器或驱动器内部采样时刻；
- USB-CAN适配器真正上总线的时刻；
- 多设备时钟同步；
- PDO/SYNC硬件相位；
- Windows具有硬实时最坏时延；
- 同轮多个SDO对象同步。

跨主机/总线/驱动结论需要硬件时间戳、共享SYNC/触发或经校准的时钟映射。

## 7. $d_y$和$d_u$需要的事件

### 7.1 B测量延迟$d_y$

理想定义从`sensor_latch_ns`到控制器实际采用该值的时刻。最低证据链：物理位置边沿或传感器锁存 → 总线反馈帧/对象响应 → `feedback_available_ns` → 控制计算采用的`cycle_id`。只有`actual_position`列不能区分传感器、通信、解析和调度延迟。

### 7.2 B/D命令延迟$d_u$

理想定义从`command_compute_ns`或明确的控制器输出时刻，到驱动实际采用目标的`drive_target_effective_ns`。最低证据链：计算完成 → `command_tx_ns` → 总线帧发送/接收 → 驱动目标回显或可辨识响应。只用位置响应会把$d_u$与$\tau_v$、D二阶动态和目标插值混合。

## 8. PDO/SYNC阶段新增证据

- SYNC帧的独立总线时间戳和周期抖动；
- 每轴RPDO目标帧相对SYNC的到达时刻；
- 每轴TPDO反馈帧及其设备采样/锁存语义；
- PDO映射、传输类型、抑制/事件定时器和丢帧计数；
- 驱动内部目标更新任务与SYNC/RPDO的相位证据；
- 适配器/主机时钟到抓包时钟的映射；
- 计划、发送、总线观察、驱动接收/回显四层事件对账。

没有这些证据，不能把主机`command_tx_ns`直接写成对象生效时刻，也不能从SDO轮询估计CSP周期。

## 9. 数据质量门

每次实验manifest必须记录合同版本、代码哈希、时钟来源、字段缺失率、失败/重试、实际轮次率、三类miss、对象顺序、标定ID和安全边界。时间不单调、`cycle_id`重复/倒退、单位或标定ID缺失、对象地址与名称冲突时，数据不得进入辨识训练集。

