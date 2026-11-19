# Windows 直连 DS-BVM：PCAN / CANopen PPM 首次控制—采集闭环

## 1. 实验对象与目标

- 日期：2026-08-01
- 轴：Z 轴
- 控制路线：`Windows -> XCAN-USB / PCAN_USBBUS1 -> CANopen -> DS-BVM Node 1`
- 总线参数：`1 Mbps`，Node-ID=`1`
- 控制模式：Profile Position Mode，相对位置
- 固定轨迹：`+5000 counts -> 停留 2 s -> -5000 counts -> 采集 2 s`
- 目标：绕过 STM32 和 Servo Studio 的运动控制，验证 Windows 直接写 CANopen 控制对象并同步读取七个对象、保存可回放会话的最小闭环。

本次实验不验证多轴、Jog、Home、PDO/SYNC、外部 PID、硬实时或完整 Servo Studio 功能替代。

## 2. 实验前状态与现场确认

用户现场确认以下条件已满足：

- 轴静止、无报警，正向 5000 counts 有机械余量；
- 现场具备独立立即停止手段；
- STM32/GM1 不在本次控制链；
- Servo Studio 在 PCAN 直连前退出；
- 当前反馈、驱动器参数、刹车和机械支撑保持既有可运动状态；
- 当前 Profile 对应历史 50 RPM、Accel/Decel/Jerk=300，本轮只读回原始对象值，不重写 Profile。

预检读回：

| 对象 | 结果 |
|---|---:|
| `0x6041 Statusword` | 初始 `0x0240`；NMT Operational 后为 `0x0433` |
| `0x603F Error Code` | `0` |
| `0x6061 Mode Display` | `1` |
| `0x6064 Position Actual` | 首次预检 `0 counts`；正式 Run 起点 `29 counts` |
| `0x6081 Profile Velocity` | `6900` raw |
| `0x6083 Profile Acceleration` | `30000` raw |
| `0x6084 Profile Deceleration` | `30000` raw |
| `0x6085 Quick Stop Deceleration` | `100000` raw |
| `0x6086 Motion Profile Type` | `0` raw |
| `0x60A4:01 Jerk` | `30000` raw |

以上数值均按驱动器原始对象值保存；本记录不把它们换算成正式 RPM、mA 或长度单位。

## 3. 实际操作与现场迭代

### 3.1 首次使能尝试

程序先按 CiA 402 写入 `0x0006 -> 0x0007 -> 0x000F`。在未发送 NMT Operational 时，驱动器停在 `0x0033`，没有写入目标位置，也没有发生计划运动。程序执行 Halt/Disable 退出。

原始会话：

`motion_acquisition\data\20260801\2026\08\20260801_232113_run-388f39a8`

该结果用于确认 DS-BVM 在此链路下需要主站发送 NMT Start Remote Node。

### 3.2 NMT 后首次实际运动

加入 NMT Operational 后，驱动器进入有效 CiA 402 状态。正向 `+5000 counts` 实际完成，但该驱动器没有在规定时间内稳定给出 Statusword bit12 Setpoint Acknowledge。旧判据因此超时，程序按异常路径 Halt 后 Disable；没有自动重试或 Fault Reset。

原始会话：

`motion_acquisition\data\20260801\2026\08\20260801_233253_run-5b435911`

该次结果证明：对当前 DS-BVM，PPM 新目标已经由 Statusword bit10 Target Reached 清零和实际位置变化接受，bit12 不能作为唯一接受证据。

### 3.3 返回恢复

为恢复到起点附近，执行一次受控相对 `-5000 counts` 返回：

- 起点：`5023 counts`
- 终点：`29 counts`
- 实际变化：`-4994 counts`
- 结束状态：`0x0433`
- Error Code：`0`

恢复记录：

`motion_acquisition\data\20260801\2026\08\20260801_233615_return_recovery`

### 3.4 正式控制—采集 Run

正式会话：

`motion_acquisition\data\20260801\2026\08\20260801_233700_run-3a2bfcc9`

实际阶段完整经过：

```text
preflight
-> enabling
-> forward
-> forward_dwell
-> returning
-> final_capture
-> completed_enabled
```

实际控制写入主序列：

```text
NMT Operational
0x6040 = 0x0006
0x6040 = 0x0007
0x6040 = 0x000F
0x607A = +5000
0x6040 = 0x005F -> 0x004F
0x607A = -5000
0x6040 = 0x005F -> 0x004F
0x6040 = 0x000F
```

第一段同时观察到 bit12 和 bit10 清零；第二段没有观察到 bit12，但观察到 bit10 清零，随后实际到位。程序保留了这一差异，没有将第二段伪记为 Setpoint Acknowledge 成功。

## 4. 正式 Run 结果

| 指标 | 结果 |
|---|---:|
| 会话结果 | `completed` |
| 起始位置 | `29 counts` |
| 最终需求位置 | `29 counts` |
| 最终实际位置 | `30 counts` |
| 最终回零偏差 | `+1 count` |
| 采样周期数 | `316` |
| 对象读取总数 | `2212` |
| 读取失败 | `0` |
| 实际采样率 | `48.22 Hz` |
| deadline miss | `0` |
| 位置需求范围 | `29 ... 5029 counts` |
| 位置实际范围 | `-505 ... 5536 counts` |
| Following Error 峰值绝对值 | `632 counts` |
| Error Code | 全程 `0` |
| Statusword Fault bit3 | 未出现 |
| Internal Limit bit11 | 未出现 |
| Following Error bit13 | 未出现 |

`samples.csv` 保持 V7 原始 16 列格式。会话包含：

- `samples.csv`
- `control_commands.jsonl`
- `manifest.json`
- `config_snapshot.yaml`
- `events.jsonl`
- `acquisition.log`
- `checksums.sha256`

六个受保护文件的 SHA-256 均已校验通过。V7 会话检查结果为：`state=completed`、`checksum_state=verified`、`compatible=true`、`replayable=true`、`issues=[]`。

## 5. 实验后状态

正常运动完成时，界面明确显示 `COMPLETED — SERVO ON`，符合“正常完成保持使能”的计划。随后现场点击“正常完成后 Disable”，再次只读预检确认：

- Statusword=`0x0433`
- CiA 402 状态=`switched_on`
- Error Code=`0`
- Position Actual=`27 counts`

之后关闭独立窗口，PCAN 已释放。该状态为 Disable Operation 后的非转矩状态，不等于驱动器主电源断电。

## 6. 结论边界

本次结论为 **PASS_WITH_LIMITATIONS**。

可以确认：

- Windows 已能通过 `PCAN_USBBUS1 / 1 Mbps / Node 1` 直接完成 DS-BVM 的 NMT、CiA 402 使能、PPM 相对位置控制和同步 SDO 采集；
- `+5000 counts` 和 `-5000 counts` 两段运动均实际完成，最终相对起点偏差 `1 count`；
- 控制写入和采集数据在同一个 CANopen 会话中顺序完成，未启用第二个 Network 或第二个采集线程；
- 正常完成保持使能、人工 Disable、异常 Halt 后 Disable 的路径均在现场出现并得到状态回读；
- 该会话可由现有 V7 数据模型校验和回放。

限制与后续问题：

- 当前 DS-BVM 的 Statusword bit12 并非每个 PPM setpoint 都置位；本模块必须保留“bit12 或 bit10 清零 + 实际到位”的兼容逻辑；
- 实际位置出现 `-505 ... 5536 counts` 的瞬态范围，Following Error 峰值为 `632 counts`，本次只证明可控、可返回、可采集，不据此宣称动态性能已优化；
- 当前 SDO 七对象采集约 48 Hz，不是硬实时同步采样；
- 本次只证明单轴固定 PPM 闭环，不等于已经完全替代 Servo Studio 的 Jog、Home、参数管理、报警管理、多轴和调试功能；
- STM32/CANopen 路线仍是独立待验证路线。本次 Windows 直连成功不自动关闭 STM32 固件、收发器、PDO/SYNC 等缺口。

## 7. 软件验证

现场兼容修改完成后执行全量 Python 测试：

```text
184 passed in 30.33s
```

测试覆盖固定使能序列、PPM 握手、bit10 初始到位防误判、无 bit12 的 DS-BVM 兼容、Halt 后 Disable、正常完成保持使能、16 列 CSV、控制日志、Manifest/checksum，以及旧 V1-V7 只读会话和 STM32 no-write 边界。

## 8. 主文档同步

本记录需要同步到：

- `00_总览.md`：当前阶段与最近一次现场结果；
- `40_逐线验收记录表.md`：Windows-PCAN-DS-BVM 控制—采集事实；
- `46_未闭环逐线缺口清单.md`：PC 直连写控制已验证，但 STM32 G8 仍未关闭；
- `57_共享硬件底座与双总线验证计划.md`：增加 Windows 直连研究路线；
- `58_现场实拍确认事实索引.md`：登记正式会话数据证据，不伪造截图文件名；
- `59_现场检查与测量任务单.md`：本轮固定 PPM 首次闭环任务标记完成。
