# 2026-06-22 实验日志：CANopen 数据采集链路与 USB-CAN 实测

## 实验对象

DINGS DS-BVM 驱动 CAN 口 (J8/J9) + XCAN-USB 适配器 (UTC-T01, VID_0C72&PID_000C) + Servo Studio 2.01.0008，通过 Peak-System CAN 类型连接 DS-BVM CAN node 622。

## 实验目标

1. 打通 USB-CAN → DS-BVM CAN 口的物理链路。
2. 通过 Servo Studio CAN 连接确认 CAN 总线工作状态。
3. 采集 CAN 总线下的 Scope CSV 数据，确认采样率是否与 USB 直连有差异。
4. 为后续 CANopen SDO 直接读取做准备。

## 实验前状态

- 06-18 已确认 USB 直连 Scope CSV 约 10 Hz。
- USB-CAN 适配器 (UTC-T01) 已定位但驱动未安装（设备管理器显示 Code 28）。
- DS-BVM CAN 口 (J8/J9) pinout 已知（Pin1=HIGH, Pin2=LOW, Pin3=GND）。
- `ds_bvm_canopen_logger` Python 工程已就绪但尚未对硬件执行。

## 实际操作

### 1. USB-CAN 驱动排查
- Windows 设备管理器中 XCAN-USB 显示"该设备的驱动程序未安装。（代码 28）"。
- 记录硬件 ID：`USB\VID_0C72&PID_000C`。
- 根据硬件 ID 判断适配器更可能走 **PEAK-System PCAN-USB / 兼容设备** 路线。
- 安装 PEAK 驱动后设备识别正常。

### 2. CAN 物理层连接
接线关系确认：
```
XCAN-USB CANH → DS-BVM J8/J9 Pin1 HIGH
XCAN-USB CANL → DS-BVM J8/J9 Pin2 LOW
XCAN-USB GND  → DS-BVM J8/J9 Pin3 GND
```

### 3. Servo Studio CAN 连接
- Servo Studio 中 CAN 连接类型选择 **Peak-System CAN**（非 Kvaser/Zhcxgd）。
- 连接成功后设备树显示 Device 1 (ID:1)，CAN node 622 在线确认。
- 状态栏显示 Connected。

### 4. Scope CSV 采集
- 在 Servo Studio Scope 页面添加 Channel（Position Demand/Actual、Following Error、Average Current）。
- 执行 PPM 多段往复运动后 Read Buffer → Save 导出 `scope-can622.csv`。

### 5. 初步数据分析
- 数据点数：349（去头后 348 有效点）。
- 总时长：~34.71 s。
- 平均采样间隔：~0.10003 s，等效采样率约 **9.997 Hz**。
- 即使用 USB-CAN 连接，CSV 导出仍约 10 Hz，与 USB 直连无差异。

## 现场观察

### 运动轨迹
```
第一段：-10 → +990 → -10 count
第二段：-10 → -1010 → -10 count（中间穿插多段短行程测试）
```

### 闭环状态
- Position Actual 能跟随 Position Demand，方向一致 → 光栅尺单反馈位置闭环已跑通。
- 第二段到达 −1010 count 后，实际位置最终收敛到目标附近。

### 回零振荡（关键异常）
- 第二段回到 −10 count 附近后，Position Demand 已基本固定，但 **Position Actual 在约 −42 ~ +30 count 范围内反复摆动**。
- 振荡期间 Average Current 也出现正负方向大幅摆荡，驱动器仍在来回补偿位置误差。
- 可能原因（按优先级排列）：
  1. Profile Velocity / Acceleration 设置偏激，到位后残余动态大；
  2. 位置环或速度环阻尼不足，导致到位后 hunting；
  3. 丝杠/导轨/联轴器存在摩擦死区或间隙，回程时表现更明显；
  4. 光栅尺反馈噪声、分辨率或滤波参数导致目标附近来回修正。

### 第一段运动特征
- 正向 +990 count：最大正向跟随误差约 +80 count。
- 反向回程：最大负向跟随误差约 −95 count。
- 回程误差大于去程，与 06-18 USB 直连观察一致。

## 可支持结论

1. **CAN 物理层已打通**。XCAN-USB (PEAK-System 兼容) → DS-BVM CAN J8/J9 → CAN node 622 在线，链路可用。
2. **Servo Studio Scope CSV 约 10 Hz 与通信总线无关**。无论是 USB 直连（CP2102N）还是 USB-CAN（XCAN-USB→CAN bus），GUI Save CSV 的采样率都在 ~10 Hz，说明瓶颈在 Servo Studio 的 CSV 导出机制本身。
3. **光栅尺单反馈位置闭环已实际跑通**。两段运动均能跟随目标位置并最终收敛。
4. **回程存在低频振荡/伺服 hunting**。这比 10 Hz 采样率更紧迫——即使将来采样率提高，如果伺服参数不优化，到位后的持续振荡仍是问题。

## 不能推出的结论

- 不能用 10 Hz 数据精确量化振荡频率或峰值幅值（振荡周期可能远短于 100 ms 采样间隔）。
- 不能将回零振荡直接归因于单一原因（需要小步进低速实验 + 高频采集 + 参数调节来区分）。
- 不能认为 CANopen SDO 能达到多高频率（尚未执行 SDO 只读实测）。

## 遗留问题

- [ ] 回零振荡的根因（需 100/200/500 count 小步进低速实验 + Servo On/Off 静止抖动对比）。
- [ ] SDO 只读的实际可达到采样率（需 Python + XCAN-USB 实测）。
- [ ] CAN 终端电阻是否已正确配置（理论 120Ω 两端，待万用表实测）。
- [ ] CAN 静态电压 (CANH/CANL recessive) 是否正常。
- [ ] CANopen node ID 是否确认为 622（Servo Studio 显示 node=622，需与 DS-BVM 参数交叉验证）。

## 下一步

- [ ] 做 100 / 200 / 500 count 小步进实验，速度和加速度降到当前的一半或更低。
- [ ] 分别记录正向、反向、回零过程的 Position Demand / Actual / Following Error / Average Current。
- [ ] 观察 Servo Off 静止、Servo On 静止、运动后到位三种状态下 Position Actual 是否抖动。
- [ ] 通过 PCAN-View 监听 CAN 总线报文（boot-up/heartbeat/EMCY）。
- [ ] 使用 `ds_bvm_canopen_logger` Python 工程执行 SDO 只读（先读 0x1000 设备类型验证通道，再读 0x6064）。
- [ ] SDO 链路稳定后，评估 PDO 周期上传的可行性。

## 应同步到

- `00_总览.md`：CAN 物理层已验证，USB-CAN 提频探索中
- `46_未闭环逐线缺口清单.md`：G8 CANopen 补充进展（CAN 物理层已通）
- `57_共享硬件底座与双总线验证计划.md`：阶段 1（物理层）部分完成
- `58_现场实拍确认事实索引.md`：XCAN-USB 已确认可用
- `59_现场检查与测量任务单.md`：CAN 接线任务完成，新增 SDO 只读任务
- `62_Scope双总线数据采集/`：scope_can622_2026-06-22.csv 已归档
- `ds_bvm_canopen_logger/README.md`：CAN 物理层已通过 XCAN-USB 验证
