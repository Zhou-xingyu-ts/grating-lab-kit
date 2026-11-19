# Servo Studio 中英文界面地图

> 用途：Servo Studio 是全英文界面。本文件把每个功能区的英文术语对应到中文含义和操作要点，让你打开软件时不用猜。
> 来源：DINGS Servo Studio Manual Ver 1.1（DS-BVS/BVM-Series）
> 状态：基于手册整理，待上机逐项验证

---

## 0. 软件总览

打开 Servo Studio 后，你会看到三个区域：

| 英文 | 中文 | 位置 | 干什么的 |
|------|------|------|----------|
| **Ribbon Bar** | 功能区/工具栏 | 顶部横条 | 所有主要操作的按钮都在这里：连接、使能、调参、Scope、保存 |
| **Tool Pane** | 工具面板 | 左侧竖栏 | 选择你要操作的大类：Drive Setup / Tuning / Motion / Object Dictionary / I/O / Motion List |
| **Target Pane** | 工作区 | 中间大块 | 根据你在 Tool Pane 选的项目，显示对应的参数页面 |
| **Device List** | 设备列表 | 右侧或底部 | 显示已连接的驱动器 |
| **Log Message** | 日志窗口 | 底部 | 显示连接状态、报错信息 |

---

## 1. Ribbon Bar（顶部工具栏）— 最常用

这是你最常打交道的地方，按从左到右的顺序：

| 英文按钮 | 中文 | 功能 | 什么时候用 |
|----------|------|------|------------|
| **Connect** | 连接 | 弹出连接窗口，选 USB(COM) 或 CAN，连驱动器 | 每次开软件第一步 |
| **Disconnect** | 断开 | 断开与驱动器的连接 | 关软件前或换连接方式时 |
| **Enable** | 伺服使能 | 给驱动器上电（按钮变橙色=已使能） | 要让电机动之前**必须**点这个 |
| **Disable** | 伺服关闭 | 断开伺服使能（按钮变黑色=已关闭） | 停止运动、改参数前 |
| **Save** | 保存/写Flash | 把当前参数写入驱动器 Flash 记忆 | **每次改完参数必须点**，否则断电丢失 |
| **Restore** | 恢复默认 | 把驱动器参数恢复出厂设置 | 调乱了想重来时 |
| **Import** | 导入参数 | 从电脑导入 `.wspf` 参数文件 | 恢复之前备份的参数 |
| **Export** | 导出参数 | 把当前参数导出为 `.wspf` 文件 | 备份参数 |
| **Scope** | 示波器/曲线监控 | 打开或隐藏 Scope 窗口 | 看实时曲线、导出 CSV 数据 |
| **Firmware** | 固件升级 | 更新驱动器固件 | 一般不用碰 |
| **Object Dictionary** | 对象字典 | 直接读写驱动器的底层参数 | 改 CAN ID、改波特率、高级调试 |

---

## 2. Tool Pane（左侧工具面板）

点击左侧的选项，右侧工作区会切换页面。

| 英文 | 中文 | 干什么的 |
|------|------|----------|
| **Drive Setup** | 驱动器设置 | 设置电机参数、编码器/反馈类型、单位 |
| **Tuning** | 调参/整定 | 电流环、速度环、位置环的自动/手动调参 |
| **Parameter** | 参数列表 | 查看/导入/导出全部参数 |
| **Motion** | 运动控制 | 发运动指令：绝对位置、速度模式、回零 |
| **Object Dictionary** | 对象字典 | 底层参数读写（高级功能） |
| **I/O** | 输入输出 | 配置数字输入输出口 |
| **Motion List** | 运动序列 | 编写多段连续运动程序 |

---

## 3. Drive Setup — 驱动器设置（接线后第一步）

### 3.1 Motor 选项卡

| 英文 | 中文 | 怎么填 |
|------|------|--------|
| Motor Type | 电机类型 | 你的空心杯电机选 **Sinusoidal PM BL Motor**（正弦波无刷） |
| Maximum Current | 最大电流 | 填电机峰值电流（mA） |
| Rated Current | 额定电流 | 填电机额定电流（mA） |
| Torque Constant | 转矩常数 | 知道就填，不知道取消勾选让软件自动算 |
| Back-EMF Constant | 反电动势常数 | 通常 = Torque Constant |
| Pole Pair | 极对数 | 电机极数 ÷ 2。不知道问厂家或师兄 |
| Resistance | 电阻 | 不用填，Current Auto Tuning 会自动测 |
| Inductance | 电感 | 不用填，Current Auto Tuning 会自动测 |
| **Apply** | 应用 | 填完点这个，参数才下发到驱动器 |
| **Save** | 保存 | 点完 Apply 再点 Ribbon Bar 的 Save |

### 3.2 Feedback 选项卡（光栅尺配置在这里！）

| 英文 | 中文 | 怎么填 |
|------|------|--------|
| Single Feedback | 单反馈 | 只用一路编码器 |
| Dual Feedback | 双反馈 | 用两路反馈（Feedback1 做速度环，Feedback2 做位置环）— **光栅尺闭环可能需要这个** |
| Feedback Type | 反馈类型 | 光栅尺是增量式 → 选 **Incremental Encoder** |
| Resolution (CPR) | 分辨率 | 对于直线电机/光栅尺：填 **count/m** 或 **count/μm**，如 1μm 分辨率 = 1,000,000 counts/m |
| Direction Invert | 方向反转 | 如果运动方向和读数方向反了，改成 true |
| **Apply** | 应用 | 填完点这个 |

> ⚠️ **重要**：Feedback2 用于位置环闭环。如果你想把光栅尺用作位置反馈来闭环控制平台位置（而不是电机轴位置），需要用 Dual Feedback 模式，把光栅尺配置为 Feedback2。

---

## 4. Tuning — 调参（三环控制的核心）

### 4.1 操作顺序（必须从内环到外环！）

```
Phase Auto Tuning → Current Tuning → Velocity Tuning → Position Tuning
    (相序自学习)      (电流环)         (速度环)          (位置环)
```

### 4.2 Phase Auto Tuning（相序自学习）

| 英文 | 中文 | 操作 |
|------|------|------|
| Phase Auto Tuning | 相序自学习 | 点一下，弹窗问是否开始 → 点 Yes。电机会自动动一下来找 UVW 相序。**第一次用新电机必须做** |

### 4.3 Current Tuning（电流环）

| 英文 | 中文 | 操作 |
|------|------|------|
| Auto Current Tuning | 自动电流环调参 | 选这个模式 → 点 Current Tuning Start → 等它自动完成 |
| Manual Current Tuning | 手动电流环调参 | 拖动 Current Amplitude 滑条 → 点 Apply → 点 Current Tuning Start |

### 4.4 Velocity Tuning（速度环）

| 英文 | 中文 | 操作 |
|------|------|------|
| Auto Velocity Tuning | 自动速度环调参 | 输入 Target Position / Velocity / Acceleration → 点 Apply → 点 Start |
| Manual Velocity Tuning | 手动速度环调参 | 手动调 Velocity Loop P/I 参数 |

关键参数：

| 英文 | 中文 | 含义 |
|------|------|------|
| Velocity Loop P | 速度环比例增益 | 越大响应越快，太大振荡 |
| Velocity Loop I | 速度环积分增益 | 消除稳态误差，太大超调 |
| Target Position | 目标位置 | 调参时用的测试行程 |
| Velocity | 速度 | 调参时用的测试速度 |
| Acceleration | 加速度 | 调参时用的测试加速度 |

---

## 5. Motion — 让电机动起来

| 英文 | 中文 | 什么时候用 |
|------|------|------------|
| **Profile Position** | 位置模式 | 让平台走到指定位置 |
| **Profile Velocity** | 速度模式 | 让电机以固定速度转（Jog 点动） |
| **Current/Torque/Voltage** | 电流/转矩/电压模式 | 调试时用 |
| **Homing** | 回零 | 让平台回到机械零点 |

### Profile Position 操作步骤：
1. 在 Tool Pane 点 Motion
2. Operation Mode 选 **Profile Position**
3. 输入 Absolute Position（绝对位置）或 Relative Position（相对位置）
4. 点 Ribbon Bar 的 **Enable**（伺服使能）
5. 点 `[>]` 开始运动
6. 点 `[0]` 把当前位置设为零点

### Profile Velocity（Jog 点动）操作步骤：
1. Operation Mode 选 **Profile Velocity**
2. 输入速度值
3. 勾选 **Jog**（点动模式）
4. 点 Enable
5. 按住 `[>]` 正向转，按住 `[<]` 反向转，松手即停

---

## 6. Scope — 示波器/曲线监控（最重要！导师能看到的就是这个的截图和 CSV）

### 6.1 打开 Scope

点 Ribbon Bar 的 **Scope** 按钮 → 出现曲线窗口

### 6.2 Scope 工具栏（Scope 窗口自己的 Ribbon Bar）

| 英文 | 中文 | 怎么用 |
|------|------|--------|
| **Save** | 保存CSV | **这就是导出数据的地方！** 点一下，曲线数据保存为 `.csv` 文件 |
| **Channel** | 通道选择 | 点一下弹出窗口，选你要监控哪些变量（位置、速度、电流…） |
| **Interval Time** | 采样间隔 | 每隔多少毫秒采一个点。越小越密，但 Buffer 很快就满了 |
| **Buffer Size** | 缓冲区大小 | 总共采多少个点。Recording Time = Interval Time × Buffer Size |
| **Track Line** | 追踪线 | 勾选后在曲线上显示竖线，可以读某时刻的具体数值 |
| **Synchronize** | 同步 | 多张图时，追踪线时间同步 |

### 6.3 Channel 窗口 — 选你要看的变量

| 常见可选变量（英文） | 中文 | 你什么时候选它 |
|-----------------------|------|----------------|
| Actual Position | 实际位置 | 每次必选 |
| Target Position | 目标位置 | 看跟踪误差 |
| Position Error | 位置误差 | 看精度，最关键 |
| Actual Velocity | 实际速度 | 看速度环表现 |
| Target Velocity | 目标速度 | 对比实际速度 |
| Actual Current | 实际电流 | 看电流环表现 |
| Encoder Count | 编码器计数值 | 看原始反馈信号 |
| Status Word | 状态字 | 诊断故障 |

### 6.4 按钮操作

| 英文 | 中文 | 区别 |
|------|------|------|
| **Start** | 开始（非实时） | 周期性从驱动器读数据显示曲线，**不是实时** |
| **Read buffer** | 单次读取缓冲区 | 把驱动器中按 Interval Time × Buffer Size 存的数据一次读出来显示。**这是你导出 CSV 前要点的** |
| **Read buffer repeatedly** | 重复读取缓冲区 | 不断重复读取。Buffer 大的时候更新很慢 |

### 6.5 导出 CSV 的标准流程（照做就行）

```
1. 点 Ribbon Bar 的 Scope → 出现曲线窗口
2. 点 Scope 窗口里的 Channel → 勾选 Actual Position、Target Position、Actual Velocity、Actual Current
3. 设置 Interval Time = 1 (ms)，Buffer Size = 2000（这样录 2 秒）
4. 点 Enable 使能
5. 发运动指令（Jog 或 Position）
6. 运动过程中点 Read buffer
7. 曲线显示出来后，点 Save → 选保存位置 → 得到 CSV
8. 截图 Scope 曲线窗口 → 留作记录
```

---

## 7. 常见英文报错/状态速查

| 英文 | 含义 | 怎么办 |
|------|------|--------|
| Drive not connected | 驱动器未连接 | 检查 USB 线，点 Connect 重连 |
| Servo Off | 伺服关闭 | 点 Enable |
| Communication Error | 通信错误 | 检查线、COM 口、波特率 |
| Over Current | 过流 | 检查电机参数设置，减小电流 |
| Over Voltage | 过压 | 检查供电 |
| Following Error | 跟随误差过大 | 减小加速度或增大位置环增益 |
| Hall Sensor Error | 霍尔传感器错误 | 检查电机编码器接线 |

---

## 8. 你现在打开 Servo Studio 后，按这个顺序点

```
第1步：点 Connect → 选 COM → 选对应端口 → OK
第2步：看 Device List 里有没有出现你的驱动器
第3步：点左侧 Drive Setup → 截图 Motor 页、Feedback 页（这是你的配置证据）
第4步：点左侧 Motion → Operation Mode 选 Profile Velocity → 勾 Jog
第5步：点 Enable → 按住 [>] 或 [<] 看电机动不动
第6步：点 Scope → Channel → 勾 Actual Position → OK → Read buffer → Save
```

这六步走通，你就有了第一份真实的实验 CSV 和截图。
