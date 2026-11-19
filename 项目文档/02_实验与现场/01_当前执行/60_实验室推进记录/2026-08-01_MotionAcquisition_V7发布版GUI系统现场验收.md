# 2026-08-01 Motion Acquisition V7 发布版 GUI 系统现场验收

> 状态：`目标已纠偏：转入研究补采主线，尚未连接本轮硬件`  
> 本轮对象：Windows `Motion Acquisition V7.0.0` 完整 onedir 发布版  
> 准确命名：V7 发布版 GUI、完整会话目录、校验和、回放分析和异常收尾链的首次系统真实现场验收  
> 不是：Motion Acquisition/USB-CAN 第一次真实读取；2026-07-12 的 V0.2 CLI 已有真实静止与运动 SDO upload 证据

> 2026-08-01 现场纠偏：本文件早先的全面 V7 验收矩阵不再作为当前执行清单。取消主动断线、故障注入、耐久和大规模生命周期验收；只做一个短静止采集排除阻断性回归，随后立即采集能补齐研究证据的数据。文件名保留用于追溯本轮目标变化。

## 当前研究补采主线（覆盖后文初始全面验收方案）

历史 `5000 counts / 50 RPM` 数据中，Accel/Decel/Jerk=`300` 只有一次高频往返，正向/返回稳态 RMS 为 `18.45 / 3.59 counts`；`150` 条件已有三次重复，正向 `3.16–5.58 counts`、返回 `3.69–4.00 counts`。当前最有价值的补采不是增加任意样本，而是补齐 300 条件的重复数并检查方向差异是否跨日稳定。

第一正式批次冻结为：

- Relative Position：`+5000/-5000 counts`；
- Velocity：`50 RPM`；
- Acceleration / Deceleration / Jerk：`300 / 300 / 300`；
- 三次完整往返；
- Motion Acquisition 只读对象：`position_demand / position_actual / following_error`；
- 请求周期：`5 ms`；
- 先保留短静止段，随后每次运动前后打主机侧事件；
- 任何运动仍由用户在 Servo Studio 发起。

2026-08-01 当前准备状态：V7 已从冻结 onedir 启动，保持 `Demo — no hardware / idle`；已预填短静止会话 `V7_GUI_REAL_STATIC_PRE_A300_01`，Z 轴、raw counts、三核心对象、5 ms、15 s，数据根目录为新建的空目录 `motion_acquisition\data\20260801`。尚未连接 PCAN、尚未生成新会话、尚未进行本轮运动。R-0 现场确认连续三轮未获得回复，当前安全停在 Demo/idle；恢复时从 R-0 继续，不重做历史调研或发布基线。

本批通过快速门：会话文件完整、运动范围正确、三对象覆盖完整、采样连续、失败读取无异常、元数据完整。数据有效即进入下一批；下一批优先是最小匹配的 150 条件或用于抵消先后顺序的重复，不预先扩张成大矩阵。

## 1. 目标与结论口径

在不改变现有接线、不接入 STM32、不让 Motion Acquisition 承担运动控制的前提下，依次验证：

1. 未修改 V7 onedir 能在当前 Windows 环境启动；
2. `XCAN-USB / UTC-T01` 经 PEAK 兼容驱动，以 `pcan / PCAN_USBBUS1 / 1 Mbps / Node 1` 完成真实 SDO upload；
3. 静止和用户通过 Servo Studio 发起的低速、小行程运动中，V7 能连续记录原始值；
4. `completed / stopped / error` 结果、六文件会话、校验和、回放、分析、退出和重连行为可追溯；
5. 任何暴露出的真实软件缺陷按失败证据—根因—最小修复—永久回归—受影响复测处理。

最终只使用 `PASS / PASS_WITH_LIMITATIONS / PARTIALLY_VERIFIED / BLOCKED_BY_HARDWARE / FAIL`。软件证据不得改写为安全、精度、同步或现场控制证明。

## 2. 固定职责与禁止边界

```text
运动路径：
Windows -> DS-BVM M3 USB/COM -> Servo Studio -> DS-BVM 内部单轴闭环
        -> 运动命令只能由用户确认并执行

只读采集路径：
Windows -> XCAN-USB / UTC-T01 -> DS-BVM J8/J9 -> Node 1 / 1 Mbps
        -> Motion Acquisition -> CANopen SDO upload
```

- STM32、V8 固件、三轴控制（SIL）页和任何固件工具均不进入本轮。
- 禁止 SDO download、Controlword、目标值/模式写入、PDO 配置、SYNC、NMT 状态写入、原始 CAN 运动帧及 Servo Studio 自动化。
- “只读”指产品调用链只有 SDO upload；总线上仍会发送 SDO 读请求，不能表述为物理零发送。
- Motion Acquisition 的“停止采集”不停止电机；运动停止和 Servo Off 始终由用户在 Servo Studio/现场安全手段完成。
- 不改历史会话，不回写新会话源文件。`samples.csv` 的原始 16 列与 raw counts 始终是权威数据。
- `0x606C` 与 `0x6078` 只记驱动器 raw values，不擅自写成 RPM 或 mA；counts 到长度的换算仍不是外部计量标定。

## 3. 开始基线

### 3.1 发布物

- 目录：`motion_acquisition/release/MotionAcquisition/`
- 入口：完整 onedir 内的 `MotionAcquisition.exe`，不得单独复制 EXE。
- 主 EXE SHA-256：`4E3489E9604EFD1645FBF39A93078D1B54E9C701E778D60293E19D9E9A5518F5`
- 2026-08-01 本机复核：493 个文件，177,426,736 bytes，发布清单 5/5 匹配。
- 详细基线：`motion_acquisition/build/Field_Verification_20260801/BASELINE_RELEASE_VERIFICATION.md`

### 3.2 计划数据与派生证据落点

- 新原始会话根目录：`motion_acquisition\data\20260801`（启动前确认可写和空间；不使用 284 字符长路径）。
- 派生审计证据：`motion_acquisition/build/Field_Verification_20260801/`。
- 实验名统一以 `V7_GUI_REAL_` 开头，与 legacy、Demo、SIL 和 synthetic 明确分开。
- 原始会话目录创建后不移动、不改名、不修改；分析输出放在派生证据目录，并记录源会话绝对路径与 SHA-256。

## 4. 现场环境待填

| 项目 | 本轮实测/截图 | 状态 |
|---|---|---|
| Windows 机器与 OS |  | `PENDING` |
| XCAN-USB / UTC-T01 标签与硬件 ID |  | `PENDING` |
| PEAK 兼容驱动版本、设备状态 |  | `PENDING` |
| python-can backend/channel | 计划 `pcan / PCAN_USBBUS1` | `PENDING` |
| CAN bitrate / DS-BVM Node-ID | 计划 `1 Mbps / 1` | `PENDING` |
| Servo Studio 版本、COM、Connected/Servo Off | 历史版本 2.01.0008 | `PENDING` |
| DS-BVM 型号/固件/当前报警 |  | `PENDING` |
| 当前反馈源 | 历史结论为 Ri0040 单反馈；需本轮确认 | `PENDING` |
| 电机铭牌 | `30ZWWC65TEK5B-1` 与旧记录拼写冲突待照片收口 | `PENDING` |
| 机械支撑、行程余量、立即停止手段 |  | `PENDING` |
| STM32 未接入 |  | `PENDING` |

## 5. 对象与采样配置

V7 产品对象字典定义 10 个只读对象，但当前采集 GUI 实际暴露 8 个复选项：历史默认 7 对象加 `error_register`；`device_type` 和 `identity_vendor_id` 不在本轮 GUI 采集选项中。

主采集沿用历史可比的默认 7 对象：

```text
position_demand
position_actual
following_error
velocity_actual
current_actual
statusword
error_code
```

- 请求周期：20 ms。
- 静止与运动基线时长：各 30 s。
- 历史参考：约 1,500 轮、约 10,500 行、49.999 Hz、零失败读取；这是比较基线，不是驱动器内部控制周期。
- 本轮研究补采不扩展对象识别；正式批次只用历史已实测的三核心对象。若另选 `error_register` 时失败，记录 SDO abort 原文，不自动否定三核心动态链。

## 6. 初始全面验收方案（已压缩，不作为当前执行清单）

每一阶段只在上一阶段通过后进入；涉及用户的物理动作时一次只请求一个短动作。

### P0 未改线现状与安全确认

用户动作：保持轴静止和 `Servo Off/Disable`，不拔线、不换线、不重新接线；确认机械支撑、行程余量和立即 Disable/停止手段；确认 STM32 未接入；拍摄能同时识别 M3 USB、J8/J9 CAN、XCAN-USB 标签/线缆走向及电机铭牌的现状证据。

- 预期：USB/COM 与 USB-CAN 两条独立链存在，STM32 不在链路中，无报警、无运动。
- 中止：接线与历史不同、Z 轴无可靠支撑、无法确认立即停止手段、出现 STO/反馈/刹车/过流/编码器报警，或无法识别 USB-CAN 路径。
- 当前状态：`PENDING`。

### P1 V7 启动、驱动识别与静止只读通信测试

在 P0 通过后，先启动完整 V7 onedir，确认默认 Demo 不自动连接 CAN；随后手工逐项核对 `pcan / PCAN_USBBUS1 / 1000000 / Node 1`，再连接真实源并对所选对象执行“只读通信测试”。不得信任历史 GUI 保存设置。

- 预期：Servo Studio USB/COM 仍在线；V7 能连接 PCAN 通道；默认 7 对象全部 upload 成功；`0x603F` 预期为 0；轴仍静止。
- 中止：通道/Node-ID 与计划不一致、Servo Studio 抢占 PCAN、持续 SDO abort、驱动报警、非零错误码或任何意外运动。
- 当前状态：`PENDING`。

### P2 30 s 静止 completed 会话

实验名：`V7_GUI_REAL_STATIONARY_01`。使用 7 对象、20 ms、30 s，自然完成。

- 功能通过门：outcome=`completed`；六文件齐全；checksum verified；16 列不变；7 对象覆盖完整；failed reads=0；sample_seq 无倒退、重复或缺口；源文件哈希在分析前后不变。
- 时间参考：CSV 主时间轴重算约 48–52 Hz、约 1,500 轮。偏离时保留实际 p50/p95/p99、deadline misses 与 CPU/驱动环境，不为过门修改黄金结果。
- 预期语义：位置保持稳定；velocity raw 接近静止基线；Error Code=0；statusword 与 Servo Off 状态可解释。
- 当前状态：`PENDING`。

### P3 用户 Servo Studio 单次低速小行程往返

在 P2 数据审核通过且轴重新确认静止后，先开始 `V7_GUI_REAL_MOTION_01` 采集并打“运动前”事件；再单独请求用户在 Servo Studio 执行一次已确认的低速、小行程正向—停稳—返回动作。历史参考为 `+1000/-1000 counts`，本轮数值必须由用户结合现场行程确认。

- 预期：Demand 先变化，Actual 同向跟随；Following Error 运动中变化、停稳后回落；velocity raw 正反方向符号合理；采集不断；Error Code 保持 0。
- 中止：异常摩擦/撞击/线缆牵拉、接近限位、方向与预期不符、持续误差不收敛、报警或 SDO 连续失败。用户先停止运动/Disable，随后再停止采集。
- 当前状态：`PENDING`。

### P4 重复启停与事件

仅在 P3 通过后，用户用相同安全参数做 2–3 次往返；每次前后由 V7 写主机侧人工事件。

- 预期：运动/静止段可分辨，位置方向与误差回落可重复，采集无累计失败或缺轮。
- 限制：事件是 Windows 主机时间，`drive_synchronized=false`，不能证明驱动器同步时序。
- 当前状态：`PENDING`。

### P5 stopped、退出、重启、回放与分析

轴静止、Servo Off 后新建一轮较长静止会话，运行至少 5 s 后点击“停止采集”；检查 outcome=`stopped` 与六文件。随后正常退出并重启 V7，扫描、打开、回放 P2/P3/P5 会话并运行完整性/基础数据质量分析。

- 预期：正常退出不挂起；stopped 会话完整；回放/分析行数与原 CSV 一致；原会话哈希不变。
- 当前状态：`PENDING`。

### P6 软件断开/重连

保持静止、Servo Off；使用 V7 的软件 Disconnect，再 Connect、重新执行只读通信测试，并新建一个短静止 completed 会话。

- 预期：无残留活动 recorder；重连后新 run_id；旧会话不变；新会话正常完成。
- 当前状态：`PENDING`。

### P7 受控意外断线与恢复（独立放行）

仅在 P0–P6 均通过并再次获得用户明确确认后执行。轴必须静止、Servo Off、无运动命令；只允许用户断开 PC 侧 USB-CAN 的 USB 插头，绝不在运动中拔线，也不动 CANH/CANL 端子。

- 预期：当前会话以 `error` 明确收尾而非伪装 completed；已有 CSV 可读；重新插入后按 Disconnect/Connect/只读通信测试流程建立全新会话。
- 若拔插会影响现场安全、设备供电或线缆状态，则标为 `BLOCKED_BY_HARDWARE`，不为覆盖率强行执行。
- 当前状态：`PENDING`。

## 7. 每轮数据审核清单

- 会话绝对路径、run_id、开始/结束时间、outcome。
- 六文件是否齐全；`checksums.sha256` 结果。
- `samples.csv` 表头精确 16 列；行数、有效轮次、对象覆盖。
- failed reads、failed cycles、最长连续失败、attempts/错误文本。
- CSV 重算实际轮次率、interval min/mean/p50/p95/p99/max、deadline misses。
- missing/reversed/duplicate sequence 与 gap。
- Position Demand/Actual/Following Error 的 raw-count 方向、峰值、停稳回落；velocity/current 只用 raw 口径。
- statusword、error code 与 Servo Studio 截图/事件的近似对应。
- 回放/分析行数与源 CSV 一致；分析前后源 SHA-256 不变。

## 8. 缺陷与裁决规则

- 任何失败先冻结原会话、截图/日志、输入配置和复现步骤；不删除失败证据。
- 只有能在软件路径中稳定复现并定位根因的问题才登记为软件缺陷；驱动/接线/报警/安全/硬件不可用分别保留证据边界。
- 不删除测试、不降低断言、不吞异常、不增加无依据 skip、不修改黄金数据。
- 产品源码或打包内容若变化，必须新增永久回归、重跑受影响及必要全量测试，并重新生成可追溯发布物后才可复测现场。

## 9. 当前结论

- 发布与源码身份：`PASS`（仅文件系统基线）。
- V7 GUI 真实 PCAN 连接、静止、运动、六文件、回放与恢复：`PARTIALLY_VERIFIED`，尚待本轮现场步骤。
- STM32、V8 固件、三驱动器 CANopen、同步、真实周期、安全和精度：`BLOCKED_BY_HARDWARE` / 本轮排除。
- 下一动作：执行 P0，未获用户现场确认前不启动硬件连接。

## 10. 后续同步

本记录先保存完整过程。实际证据产生后再最小提炼：验收事实进 `40`，未解决问题进 `46`，照片/截图/会话哈希索引进 `58`，下一现场动作进 `59`，总览只更新简短状态；不得预填不存在的证据文件名。
