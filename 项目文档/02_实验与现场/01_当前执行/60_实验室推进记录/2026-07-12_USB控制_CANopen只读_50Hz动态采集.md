# USB 控制 + CANopen 只读 50 Hz 动态采集

> 日期：2026-07-12  
> 对象：现有 Z 轴平台、DS-BVM、Ri0040 光栅反馈、DINGS Servo Studio、XCAN-USB  
> 边界：运动命令仅由已允许的 USB 直连 Servo Studio 路径执行；CANopen 侧仅做 SDO upload，不写对象、不发 NMT Operational、不配置 PDO。

## 1. 实验目标

1. 绕开 Servo Studio Scope CSV 约 10 Hz 限制。
2. 验证 Servo Studio 走 M3 USB/COM 控制时，Python 能否同时通过 USB-CAN 只读采集。
3. 对低速、小行程正反向运动采集位置给定、实际位置、跟随误差、速度、电流和状态证据。

## 2. 实验前状态

- Servo Studio 2.01.0008 通过 M3 USB/COM 连接 `Device 1 (ID:1)`。
- USB-CAN 为 Peak-System 兼容设备，通道 `PCAN_USBBUS1`，波特率 `1 Mbps`。
- CANopen 实际 Node-ID 经 Servo Studio 扫描和 SDO 返回确认是 `1`；既有记录中的“622”不是可填入 `1..127` 配置字段的 Node-ID。
- 运动参数：Velocity=`100 RPM`，Acceleration/Deceleration=`300 RPM/s`，Jerk=`300 RPM/s^2`，Relative Position=`1000 counts`。
- 初始 Servo Off；错误码为 0。

## 3. 实际操作

1. 在根目录 `motion_acquisition/` 建立独立 `.venv`，安装工程声明的 `python-can`、`canopen` 等依赖。
2. 使用 `PCAN_USBBUS1 / 1 Mbps / Node 1` 单次 SDO upload 读取 `0x6064:00`，一次返回 `13 counts`。
3. 完成 10 秒静止采集：500 轮、4 对象、实际约 50 Hz、零读取失败。
4. 将 Servo Studio 改为 M3 USB/COM 连接，同时保留 USB-CAN；在 Servo Off 下并行读取 `0x6064/0x6041/0x603F`，一次成功。
5. 第一次 30 秒记录采集使能状态变化；Enable 后状态字约 3 秒从 `0x0240 -> 0x0021 -> 0x0033 -> 0x0437`，最终进入 Operation Enabled。该轮未下发位移。
6. 第二次启动 30 秒、20 ms 周期 SDO 只读记录；通过 Servo Studio 执行 `+1000 counts`，到位后执行 `-1000 counts` 返回，随后 Disable。

## 4. 原始数据与结果

动态原始数据：

- `motion_acquisition/logs/motion_sdo_20260712_195923.csv`
- `motion_acquisition/logs/motion_sdo_20260712_195923.csv.manifest.json`

动态采集结果：

| 指标 | 结果 |
|---|---:|
| 时长 | 29.980485 s |
| 采样轮次 | 1500 |
| 每轮对象数 | 7 |
| 读取失败 | 0 |
| 实际轮次频率 | 49.999 Hz |
| 周期均值 / 范围 | 20.000 ms / 19.328--20.458 ms |
| Position Demand | 20--1020 counts |
| Position Actual | 18--1021 counts |
| Following Error | -123--+134 counts |
| Velocity Actual | -82--+110 RPM |
| Current Actual | -289--+380 mA |
| Error Code | 全程 0 |

界面到位观察：正向到约 `1021 counts`，返回后约 `18--20 counts`；到位静态误差约 `-1--2 counts`。动态峰值跟随误差明显大于到位静态误差，不能用到位截图替代动态指标。

## 5. 可支持结论

1. DS-BVM 可在 Servo Studio 通过 USB/COM 控制的同时，由 USB-CAN/Python 进行 CANopen SDO upload 只读采集。
2. 当前 7 对象 SDO 轮询可稳定实现约 50 Hz，显著高于 Servo Studio CSV 的约 10 Hz，30 秒内无读取失败。
3. 本次低速 `+1000/-1000 counts` 往返完成并返回起点附近，错误码全程为 0。
4. 当前主要动态证据是峰值跟随误差约 `-123/+134 counts`，需要结合 count 标定、方向、机械振动和控制参数进一步解释。

## 6. 不能推出的结论

- 不能据此放行 CANopen 写控制、PDO 配置、STM32 主站、正式运动或更大行程/速度。
- 不能把 50 Hz SDO 当作驱动器内部控制周期或高带宽动态测量能力。
- 未完成 counts 到 um 标定前，不能把误差直接换算为微米精度。
- 单次往返不能证明重复定位精度、长期稳定性、限位、Home、STO 或刹车策略已经闭环。

## 7. 遗留问题与下一步

1. 重复采集不少于 5--10 组同参数往返，统计峰值误差、稳态误差、回程残差与重复性。
2. 补齐 counts/um、方向极性和实际行程标定。
3. 对比不同 Velocity/Acceleration 组合，保持小行程和可停止边界。
4. SDO 50 Hz 已够当前趋势和低速动态初测；若研究目标需要更高带宽，再单独评估被动 TPDO 或经放行的 PDO 配置。

## 8. 应同步主文档

- `00_总览.md`：更新为 USB 控制 + CANopen SDO 只读 50 Hz 已验证。
- `46_未闭环逐线缺口清单.md`：关闭 SDO 只读待执行项，保留 TPDO、终端电阻、STM32 等缺口。
- `59_现场检查与测量任务单.md`：将下一现场任务调整为重复动态试验、标定和误差解释。

## 9. 同参数 5 组重复往返补充试验

随后在相同参数下执行 5 组 `+1000/-1000 counts` 往返：Velocity=`100 RPM`，Acceleration/Deceleration=`300 RPM/s`，Jerk=`300 RPM/s^2`。采集保持 20 ms 周期、7 对象 SDO upload，只读边界不变；结束后已 Disable，Servo Off，无非零错误码。

原始数据：

- `motion_acquisition/logs/motion_sdo_20260712_200718.csv`
- `motion_acquisition/logs/motion_sdo_20260712_200718.csv.manifest.json`

采集共 2250 轮，实际 `50.000 Hz`，周期范围 `19.168--20.904 ms`，零读取失败。按给定轨迹变化段提取结果：

| 周期 | 正向峰值 Following Error | 返回峰值 Following Error | 正向到位约 1 s 后实际位置 | 返回到位约 1 s 后实际位置 |
|---:|---:|---:|---:|---:|
| 1 | +71 counts | -201 counts | 1021 | 17 |
| 2 | +129 counts | -181 counts | 1021 | 17 |
| 3 | +121 counts | -194 counts | 1020 | 11 |
| 4 | +78 counts | -189 counts | 1021 | -20 |
| 5 | +77 counts | -146 counts | 1023 | 9 |

结果显示明显方向不对称：正向到位重复性较好；返回方向动态峰值误差更大，返回给定为 20 counts 时的 1 s 残差依次约为 `3、3、9、40、11 counts`。第 4 组出现约 40 counts 残差，说明此前“回程低频振荡/不稳定”并非仅由 10 Hz Scope 采样造成，但单凭本数据尚不能区分摩擦/刹车、机械间隙、控制参数、反馈方向/标定或负载方向效应。

因此本轮不继续增加无差别重复次数。下一步应保持小行程，分别降低速度或加速度做单变量对比，并同步观察返回段电流、峰值误差和稳定时间；在 counts/um、方向和机械状态未确认前，不提高速度和行程。

## 10. 50 RPM 单变量对比

用户现场确认前述往返无明显异响，但存在轻微摩擦声；未确认润滑剂型号和加注点，因此本轮不加油。保持 Relative Position=`1000 counts`、Acceleration/Deceleration=`300 RPM/s`、Jerk=`300 RPM/s^2`，仅将 Velocity 从 `100 RPM` 降为 `50 RPM`，执行 3 组往返。

原始数据：

- `motion_acquisition/logs/motion_sdo_20260712_201224.csv`
- `motion_acquisition/logs/motion_sdo_20260712_201224.csv.manifest.json`

采集为 1750 轮、实际 `50.000 Hz`、零读取失败、Error Code 全程 0。提取结果：

| 周期 | 正向峰值 Following Error | 返回峰值 Following Error | 正向到位约 1 s 后误差 | 返回到位约 1 s 后误差 |
|---:|---:|---:|---:|---:|
| 1 | +80 counts | -59 counts | -8 counts | +3 counts |
| 2 | +129 counts | -32 counts | -15 counts | +5 counts |
| 3 | +109 counts | -24 counts | -13 counts | +5 counts |

与 100 RPM 的 5 组数据相比，返回方向峰值误差从约 `-146~-201 counts` 降至 `-24~-59 counts`，返回到位残差也收敛至约 `3--5 counts`，说明回程问题具有明显速度相关性。正向峰值并未同幅下降，仍需结合对象单位/比例、控制参数、摩擦与反馈标定解释。`0x606C` 的数值不能在未核实 DS-BVM 对象单位和缩放前直接等同于 Servo Studio 显示 RPM。

下一步不直接跳到 5000 counts。若现场确认当前方向机械余量充分，先做 `2000 counts @ 50 RPM` 中间阶梯；同时保持急停/Disable 可达，发现摩擦声加重、线缆牵拉或回程残差放大立即停止。润滑处理必须先确认导轨/丝杠类型、厂家规定润滑剂和加注点，避免污染光栅或使用不兼容油脂。

## 11. SDO 100 Hz / 200 Hz 上限测试

在 Servo Off、无运动状态下，对同一 `PCAN_USBBUS1 / 1 Mbps / Node 1` 链路进行采样周期上限测试：

| 请求周期 | 对象数 | 实际频率 | 周期范围 | 读取失败 | 原始数据 |
|---:|---:|---:|---:|---:|---|
| 10 ms | 7 | 98.62 Hz | 9.48--10.83 ms | 0 | `motion_sdo_20260712_201548.csv` |
| 5 ms | 7 | 101.62 Hz | 9.28--10.91 ms | 0 | `motion_sdo_20260712_201745.csv` |
| 5 ms | 3 | 199.90 Hz | 4.24--5.91 ms | 0 | `motion_sdo_20260712_201831.csv` |

三对象为 `position_demand / position_actual / following_error`。结果表明：当前 SDO 实现下，7 对象完整轮询可稳定运行在约 100 Hz；继续把请求周期从 10 ms 压到 5 ms 不会得到 200 Hz。精简为三个动态核心对象后可达到约 200 Hz。若要求状态字、错误码、电流等所有对象同时达到 200 Hz，应评估分频采集架构或 TPDO，而不是继续压缩 SDO 周期。

现场已确认 5000 counts 机械余量足够，但 Servo Studio 的 Relative Position 自动化输入未能生效，界面仍为 1000 counts，因此没有启动 5000 counts 运动。待用户手动改为 5000 并再次核对后再执行。

## 12. 5000 counts / 50 RPM / 200 Hz 核心轨迹

用户手动将 Relative Position 改为 `5000 counts`，确认机械余量充足。保持 Velocity=`50 RPM`、Acceleration/Deceleration=`300 RPM/s`、Jerk=`300 RPM/s^2`，执行一次相对 `+5000/-5000 counts` 往返。Python 以 5 ms 请求周期读取 `position_demand / position_actual / following_error`；Servo Studio 通过 USB/COM 负责状态监视与 Disable。结束后 Servo Off，界面无报警。

原始数据：

- `motion_acquisition/logs/motion_sdo_20260712_202323.csv`
- `motion_acquisition/logs/motion_sdo_20260712_202323.csv.manifest.json`

结果：

| 指标 | 正向 | 返回 |
|---|---:|---:|
| 给定 | 14 -> 5014 counts | 5014 -> 14 counts |
| 动态 Following Error | -40--+76 counts | -43--+41 counts |
| 到位约 1 s 后 Following Error | +37 counts | +9 counts |
| 稳态观察窗误差范围 | -23--+42 counts | 0--+9 counts |
| 稳态观察窗 RMS | 18.45 counts | 3.59 counts |
| 稳态观察窗落在 +/-5 counts 比例 | 36.3% | 82.7% |

采集共 5980 轮，实际 `199.325 Hz`，平均周期 `5.017 ms`，周期范围 `4.230--8.314 ms`，零读取失败；实际位置范围 `5--5051 counts`。

5000 counts 长行程数据表明：在 50 RPM 下，返回动态误差已较 100 RPM 明显改善，但正向到位后的低频波动/收敛仍明显，且方向不对称。该结果不能由“缺少润滑油”单独解释；后续需把位置环参数、顺逆向负载/重力、机械摩擦与间隙、线缆牵引、反馈方向和 counts/um 标定分别检查。未确认润滑规范前仍不加油。

## 13. 5000 counts / 50 RPM / 150 加减速度三组重复

为减少单次偶然性，保持 Relative Position=`5000 counts`、Velocity=`50 RPM`，仅将 Acceleration/Deceleration/Jerk 从 `300` 降至 `150`，使用三核心对象约 200 Hz 采集三组完整往返。前两组完整轨迹位于 `motion_sdo_20260712_202749.csv`；原40秒窗口未完整覆盖第3组返回，因此不把缺失段计入统计，另用60秒窗口补采一组完整往返，文件为 `motion_sdo_20260712_203414.csv`。两次采集均有对应 manifest。

| 周期 | 方向 | 动态峰值 Following Error | 稳态窗误差范围 | 稳态窗 RMS | 稳态窗 +/-5 counts 比例 |
|---:|---|---:|---:|---:|---:|
| 1 | 正向 | -20--+72 | -4---3 | 3.16 | 100.0% |
| 1 | 返回 | -94--+37 | +4 | 4.00 | 100.0% |
| 2 | 正向 | -37--+83 | -9---1 | 5.58 | 60.8% |
| 2 | 返回 | -34--+29 | +4 | 4.00 | 100.0% |
| 3 | 正向 | -29--+52 | -8--0 | 4.42 | 73.1% |
| 3 | 返回 | -40--+25 | +1--+5 | 3.69 | 100.0% |

补采日志实际频率 `199.752 Hz`，11986轮。三组结果显示：Accel/Decel/Jerk 从300降到150后，正向稳态 RMS 从先前单次的18.45 counts显著下降到3.16--5.58 counts，且返回稳态 RMS 维持3.69--4.00 counts。正反向到位稳定性趋于接近，改善在重复试验中可复现。

当前证据更支持“加减速激励/位置环收敛是正向低频波动的重要因素”，而不是把现象直接归因于缺润滑。动态瞬间仍可见约 -94--+83 counts 峰值，后续若继续优化，应在150附近做更细参数阶梯，并优先完成counts/um标定；未确认润滑规范前不加油。

## 14. 临时 counts/位移换算（2026-07-13）

用户现场给出 `5 mm ≈ 10000 counts`，同意现阶段先按该比例进行工程换算。由此暂取：

- `2000 counts/mm`
- `1 count ≈ 0.5 μm`
- 本轮 `5000 counts` 相对行程约为 `2.5 mm`

该比例与当前缺口清单中依据 `Ri0040=40×插补、RCLC 20 μm 栅距` 得到的理论 `500 nm/count` 一致，但尚未用千分表、位移计或其他外部长度基准完成现场实测，因此只能标记为“暂定工程换算”，不能作为最终计量标定或精度溯源依据。

按 `1 count ≈ 0.5 μm` 重算第 13 节三组完整往返：

| 周期 | 方向 | 动态峰值 Following Error | 稳态窗误差范围 | 稳态窗 RMS |
|---:|---|---:|---:|---:|
| 1 | 正向 | -10--+36 μm | -2---1.5 μm | 1.58 μm |
| 1 | 返回 | -47--+18.5 μm | +2 μm | 2.00 μm |
| 2 | 正向 | -18.5--+41.5 μm | -4.5---0.5 μm | 2.79 μm |
| 2 | 返回 | -17--+14.5 μm | +2 μm | 2.00 μm |
| 3 | 正向 | -14.5--+26 μm | -4--0 μm | 2.21 μm |
| 3 | 返回 | -20--+12.5 μm | +0.5--+2.5 μm | 1.85 μm |

其中 `±5 counts` 等价于暂定的 `±2.5 μm`。三组稳态 RMS 的工程换算范围为：正向 `1.58--2.79 μm`，返回 `1.85--2.00 μm`；动态瞬间全组三组最坏峰值约为 `-47--+41.5 μm`。这组结果描述的是当前参数、当前负载和当前暂定比例下的跟随误差，不等同于平台的最终定位精度、重复定位精度或光栅尺计量精度。
