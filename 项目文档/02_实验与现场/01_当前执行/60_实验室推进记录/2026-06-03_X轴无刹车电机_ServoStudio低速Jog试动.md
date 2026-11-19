# X 轴无刹车电机 + DS-BVM + Servo Studio 低速 Jog 主动试动

## 1. 基本信息

- 日期：2026-06-03
- 时间：待补充
- 地点：实验室现场
- 操作者：用户
- 记录人：Codex 根据用户现场事实整理
- 实验/推进对象：X 轴无刹车电机，不是原 Z 轴带刹车平台
- 控制方式：电脑 USB/COM + DINGS Servo Studio
- 软件/工具：`DINGS' Servo Studio - 2.01.0008`
- 相关设备：DS-BVM 驱动器、X 轴无刹车电机、电脑 USB/COM 连接
- 相关文件：截图文件名待补充
- 记录状态：草稿

## 2. 本次目标

记录 X 轴无刹车电机在 `DS-BVM + Servo Studio + USB/COM` 条件下的低速短时 Jog 主动试动事实，明确它能支持的结论和不能外推到 Z 轴带刹车平台、光栅尺反馈或控制器路线的结论。

## 3. 实验前状态

### 3.1 硬件状态

- 电源：底部状态栏显示电压约 `24.1 V`
- 驱动器：DS-BVM，左侧 COM 下显示 `Device 1 (ID:1)`
- 电机：X 轴无刹车电机
- 编码器：本记录未单独确认编码器线序和分辨率
- 控制器：本次未使用 GM1/EtherCAT 或 STM32/CANopen 控制器下发运动
- 通信：电脑 USB/COM + Servo Studio 与 DS-BVM 连接
- 机械状态：X 轴无刹车测试对象，非原 Z 轴带刹车平台

### 3.2 软件状态

- 软件名称：`DINGS' Servo Studio - 2.01.0008`
- 连接状态：底部显示 `Connected`
- 设备树：COM 下 `Device 1 (ID:1)`
- 驱动状态：顶部/状态区显示 `Servo On`
- 页面提示：`Target Reached`
- 模式：`Operation Mode = Profile Velocity`
- 日志状态：`Log Message` 中可见 `Device Connected`
- 操作项：`Jog` 已勾选

### 3.3 安全边界

本次允许：

- 记录 X 轴无刹车电机在 Servo Studio 下的低速短时 Jog 主动试动事实。
- 记录 Servo Studio 页面上可见的状态、数值和日志。
- 将本次结果限定为 X 轴无刹车测试对象的阶段性过程记录。

本次禁止：

- 将 X 轴无刹车电机试动结果直接迁移到 Z 轴带刹车平台。
- 写成 Z 轴运动已放行。
- 写成 Renishaw 光栅尺 / Ri0040 已接入或光栅尺反馈已验证。
- 写成 GM1/EtherCAT 或 STM32/CANopen 已控制 DS-BVM。
- 写成可以进行 Home、Position Move、Auto Tune、Motor Test 或正式运动。

## 4. 实际操作过程

1. 通过电脑 USB/COM 与 DINGS Servo Studio 连接 DS-BVM。
2. Servo Studio 左侧 COM 下识别到 `Device 1 (ID:1)`，底部显示 `Connected`。
3. 在 Servo Studio 中进入 Motion 相关页面。
4. 驱动器进入 `Servo On` 状态。
5. `Operation Mode` 显示为 `Profile Velocity`。
6. 勾选 `Jog`。
7. 设置或显示 `Jog Velocity = 22 RPM`、`Acceleration = 300 RPM/s`、`Deceleration = 300 RPM/s`、`Jerk = 300 RPM/s^2`。
8. 用户现场确认 X 轴无刹车电机已经动起来。
9. 截图时页面显示 `Target Reached`，速度回到 `0 RPM`。

## 5. 现场观察与数据

### 5.1 数值记录

| 项目 | 数值 | 备注 |
|---|---:|---|
| Operation Mode | `Profile Velocity` | Servo Studio Motion 页面可见 |
| Status word | `0x0437` | 截图时状态 |
| Position | `4 counts` | 截图时数值 |
| Velocity | `0 RPM` | 截图时速度已回到 0 |
| Current | `76 mA` | 截图时数值 |
| Voltage | `约 24.1 V` | 底部状态栏 |
| Temperature | `约 35°C` | 底部状态栏 |
| PositionError | `0 counts` | 截图时数值 |
| Jog Velocity | `22 RPM` | Jog 参数 |
| Acceleration | `300 RPM/s` | Jog 参数 |
| Deceleration | `300 RPM/s` | Jog 参数 |
| Jerk | `300 RPM/s^2` | Jog 参数 |

### 5.2 截图/照片

| 文件名 | 内容 | 用途 | 边界 |
|---|---|---|---|
| 待补充截图文件名 | Servo Studio Motion 页面，显示 `Connected`、`Device 1 (ID:1)`、`Servo On`、`Target Reached`、`Profile Velocity`、Jog 参数和状态数值 | 证明本次 X 轴无刹车电机低速 Jog 试动的现场软件状态 | 不证明 Z 轴带刹车平台可运动；不证明光栅尺反馈或控制器通信链路已验证 |

### 5.3 异常现象

- 是否报警：截图界面未见明显报警；仍需补充 Alarm / Error 页面复核。
- 是否异响：待补充。
- 是否抖动：待补充。
- 是否失控：用户现场确认电机已经动起来，但未记录失控现象；仍需补充停止状态和方向记录。
- 是否通信异常：`Log Message` 中可见 `Device Connected`，底部显示 `Connected`。
- 其他：截图时显示 `Target Reached`。

## 6. 本次可支持的结论

- X 轴无刹车电机在 `DS-BVM + Servo Studio + USB/COM` 下完成低速短时 Jog 主动试动。
- Servo Studio 可在 `Profile Velocity` 模式下使能驱动器并执行低速 Jog。
- 截图时速度回到 `0 RPM`，`PositionError` 为 `0 counts`，界面未见明显报警。
- 本次说明 DS-BVM 驱动 X 轴无刹车测试对象的基础主动试动链路初步打通。

## 7. 本次不能推出的结论

- 不代表 Z 轴带刹车平台可以运动。
- 不代表 Z 轴运动已放行。
- 不代表 Renishaw 光栅尺已接入。
- 不代表 Ri0040 已接入。
- 不代表光栅尺反馈已验证。
- 不代表全闭环反馈已验证。
- 不代表 GM1 / EtherCAT 已控制 DS-BVM。
- 不代表 STM32 / CANopen 已控制 DS-BVM。
- 不代表控制器通信链路已打通。
- 不代表可以进行 Home。
- 不代表可以进行 Position Move。
- 不代表可以 Auto Tune。
- 不代表可以 Motor Test。
- 不代表可以正式运动。
- 不代表原 Z 轴项目安全链、刹车、STO、限位、Home、Index、零点已经闭环。
- 不代表 X 轴试动结果可直接迁移到 Z 轴平台。

## 8. 遗留问题

- 需补充试动后 `Servo Off` 停止状态截图。
- 需补充 Jog+ / Jog- 方向记录。
- 需补充 Position 变化量记录。
- 需补充是否有报警的复核记录。
- 需补充截图文件名。
- 需继续区分 X 轴无刹车试动和 Z 轴带刹车平台验证。
- Z 轴带刹车平台仍需独立完成反馈、刹车、STO、限位、Home、光栅尺和运动放行闭环。

## 9. 建议同步到主文档

- `40_逐线验收记录表.md`：建议新增 X 轴无刹车电机 Servo Studio 低速 Jog 试动记录。
- `46_未闭环逐线缺口清单.md`：建议标记该进展仅限 X 轴无刹车测试对象，Z 轴与光栅尺闭环仍未闭环。
- `58_现场实拍确认事实索引.md`：建议登记 Servo Studio Motion 页面截图。
- `59_现场检查与测量任务单.md`：建议更新 X 轴低速试动任务，保留待补证据项。

## 10. 下一步建议

- 整理并命名本次 Servo Studio Motion 页面截图。
- 补充试动后 `Servo Off` 停止状态截图。
- 补充 Jog+ / Jog- 方向记录和 Position 变化量记录。
- 打开 Alarm / Error 页面，复核是否存在报警。
- 若后续要推进 Z 轴带刹车平台，必须单独按 `39/40/46/58/59` 关闭刹车、STO、限位、Home、Index、零点、光栅尺反馈和运动放行证据。
