# EtherCAT 主站导入与 PDO 联调作业指导：GM1 -> DS-BVM

> 更新时间：2026-06-01  
> 资料定位：这是 GM1 + EtherCAT + DS-BVM 候选路线的通信验证作业指导，不是当前现场已放行接线方案。  
> 用途：把“GM1 能否扫到 DS-BVM”推进到“GM1 能否读写最小运动对象，并在安全条件闭合后发一个受控小步 PPM 命令”。  
> 边界：这不是让电机马上动。先完成 XML/ESI 导入、从站扫描、PDO 核对、对象读写、状态机检查，再允许首次小步运动。

## 这一步在闭环链路里负责什么

候选路线中的物理线关系是：

```text
GM1 EtherCAT 口 -> 屏蔽 Cat5e/Cat6 -> DS-BVM EtherCAT Input
```

现在要讲的是这根线“通了以后”怎么用。EtherCAT 不是普通网线插上就能运动，它至少要过四关：

1. GM1 主站认识 DS-BVM 这个从站，靠 XML/ESI 文件；
2. GM1 和 DS-BVM 进入正确通信状态，至少能稳定扫描，后续进 OP；
3. PDO 映射里有最小读写对象；
4. GM1 按 CiA 402 状态机给 DS-BVM 发使能和位置命令。

新人最容易犯的错是：扫描到从站以后，直接写目标位置，结果轴不动，就开始怀疑 U/V/W。实际上，这时问题可能还在 `Controlword`、`Modes of Operation`、PDO 映射、状态机、STO/限位报警或软件限位。

> ⚠️ 本作业的安全底线  
> EtherCAT 配置阶段可以不上 U/V/W，可以 Servo Off，可以只读对象。首次写 `Controlword` 前，Z 轴必须支撑，刹车策略清楚，软件限位和机械限位都在，现场有人守断电。

## 官方资料入口

> 📎 Panasonic GM1 控制器官方资料入口：[Panasonic GM1 EtherCAT-compatible Controller](https://industry.panasonic.com/ap/en/products/fasys/plc/mc/gm1)  
> 请重点看 GM1 EtherCAT 控制器资料、编程工具说明和 EtherCAT 端口规格。手册里提到 GM1 编程工具、PANATERM Lite for GM，以及 EtherCAT 控制器通信周期范围；具体软件界面以现场安装版本为准。

> 📎 DINGS DS-BVM-FETC 的 EtherCAT XML/ESI 文件：[DINGS 英文官网下载 DS-BVS-FETC DS-BVM-FETC XML File](https://www.dingsmotion.com/downloads/downloads.php?idx=125)  
> 请重点看下载得到的 XML/ESI 文件。项目已归档到 `../../../../90_原始资料/官方软件/DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网/WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`。

> 📎 DINGS DS-BVS/BVM 官方参考手册：[DINGS DS-BVS/BVM Series Reference Manual](https://www.dingsmotion.cn/uploadfile/2022/0509/20220509031529520.pdf)  
> 请重点看 Profile Position Mode、`0x6040 Controlword`、`0x6041 Statusword`、`0x6060/0x6061 Modes of Operation`、`0x607A Target Position`、`0x6064 Position Actual Value`、`0x607D Software Position Limit`。

## 开始前的现场状态

| 项目 | 必须状态 | 为什么 |
|---|---|---|
| GM1 | 已单独上电稳定 | 主站先活着 |
| DS-BVM | 24 V 上电稳定，USB/Servo Studio 可连接更好 | 证明驱动器本体不是死机 |
| EtherCAT 线 | GM1 -> DS-BVM Input，优先成品屏蔽线 | 接 Output 或自制线序错会扫不到 |
| U/V/W | 可以暂不接，或已接但 Servo Off | 通信调试不需要电机出力 |
| 编码器 | J201 pin 1-8 已接并手转有反馈 | 后续 `0x6064` 才有意义 |
| 刹车 | 独立 24 V 已测，Z 轴有支撑 | 首次使能不允许掉轴 |
| Hall/J13 | 未确认线仍贴标绝缘 | 不把未闭合项混进通信排错 |

## 第一步：导入 XML/ESI，让 GM1 认识 DS-BVM

先把文件放在 GM1 工程软件能导入的位置：

```text
../../../../90_原始资料/官方软件/DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网/
  WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml
```

操作逻辑：

1. 打开 GM1 工程软件。
2. 找到 EtherCAT 从站设备库或 ESI/XML 导入入口。
3. 导入 `WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`。
4. 新建或打开当前 Z 轴工程。
5. 扫描 EtherCAT 网络，只保留一台 DS-BVM 从站。
6. 记录从站名称、站号、供应商/产品识别信息、扫描时间。

为什么要导入 XML/ESI？  
因为主站要知道这个从站有哪些对象、PDO 可以怎么映射、输入输出数据多长。如果不导入，主站可能只能看到一个未知设备，或者能扫到但不能进 OP。

如果导入失败：

| 现象 | 先查什么 |
|---|---|
| 软件不接受 XML | 文件是否解压、是否选错 zip、软件是否要求放到指定 ESI 目录 |
| 导入后设备库没有 DS-BVM | 重启工程软件或刷新设备库 |
| 扫描到未知设备 | XML 与实物固件/产品码不匹配，或主站没使用新库 |
| 扫不到任何设备 | 先查物理链路、电源、Input/Output，不要先改 PDO |

## 第二步：扫描从站，只证明“主站能说话”

允许状态：

| 子系统 | 允许动作 |
|---|---|
| GM1 | 扫描网络、读取从站信息 |
| DS-BVM | 保持 Servo Off，读状态 |
| 电机 | 不运动 |
| 刹车 | 保持支撑，不自动联动 |

扫描通过后记录：

| 项目 | 记录 |
|---|---|
| ESI/XML 文件名 | |
| 从站显示名称 | |
| 站号/节点号 | |
| EtherCAT 状态 | Init / Pre-Op / Safe-Op / OP |
| Link 灯 | |
| DS-BVM 报警 | |
| Servo Studio 同时是否可连接 | |

判断：

| 现象 | 说明 | 下一步 |
|---|---|---|
| 扫不到从站 | 物理线、Input/Output、供电、ESI 都可能 | 先换成品网线、确认接 Input |
| 扫到但不进 OP | PDO、报警、安全状态或对象配置问题 | 先读错误，不运动 |
| OP 稳定但对象读写失败 | PDO/SDO 配置或变量绑定问题 | 回到对象映射 |
| USB 能连但 EtherCAT 不通 | 驱动器本体有电，问题更集中在 EtherCAT 侧 | 查主站、ESI、线缆 |

## 第三步：先选最小 PDO，不要一开始塞满

项目 ESI/XML 中已经能看到多组 PDO。对第一轮 PPM，小而清楚比多而复杂更好。

最小 RxPDO，主站写给驱动器：

| 对象 | 名称 | 位长 | 为什么需要 |
|---|---|---:|---|
| `0x6040` | Controlword | 16 | 使能、停机、触发新目标 |
| `0x607A` | Target Position | 32 | PPM 目标位置 |
| `0x6060` | Modes of Operation | 8 | 选择 PPM，若不在 PDO 中，可先用 SDO 写 |
| `0x60FE:01` | Physical Outputs | 32 | 后续 GPO/刹车输出用，第一轮可读写但不接刹车 |

最小 TxPDO，驱动器回给主站：

| 对象 | 名称 | 位长 | 为什么需要 |
|---|---|---:|---|
| `0x6041` | Statusword | 16 | 判断驱动状态机 |
| `0x6064` | Position Actual Value | 32 | 电机编码器实际位置 |
| `0x6061` | Modes of Operation Display | 8 | 确认驱动器实际进入 PPM |
| `0x60FD` | Digital Inputs | 32 | 看限位/STO/GPI 状态，排查安全输入 |

ESI 里可见的典型 PDO：

| PDO | ESI 中的内容 | 用途 |
|---|---|---|
| `RxPDO 5` / `0x1604` | `0x6040`、`0x607A`、`0x60FE:01` | PPM 第一轮很适合起步 |
| `TxPDO 5` / `0x1A04` | `0x6041`、`0x6064`、`0x60FD` | 第一轮状态反馈够用 |

如果 GM1 软件允许自定义 PDO，先用上面这组。若它默认选择了更复杂 PDO，也不要急着删改，先确认关键对象有没有被映射出来。

> ⚠️ 不要第一次就启用自动刹车 GPO  
> 即使 PDO 里有 `0x60FE Physical Outputs`，也只代表主站能写数字输出对象。J13 GPO 仍不能直接接刹车白线，自动刹车必须走 `../../../../02_实验与现场/01_当前执行/80_阶段参考资料/51_未闭合接线项验证作业指导_厂家问询版.md` 的假负载和外部驱动流程。

## 第四步：对象读写静态检查，不使能

在 Servo Off 状态下，先做这些检查：

| 检查 | 操作 | 预期 |
|---|---|---|
| 读 `0x6041` | 看 Statusword | 有稳定值，不乱变 |
| 读 `0x6064` | 手转电机轴 | 数值连续变化 |
| 写/读 `0x6060` | 选择 PPM | `0x6061` 显示对应模式 |
| 读 `0x60FD` | 切换 GPI0 假测试或观察默认输入 | 输入位变化可解释 |
| 写 `0x607D` | 设置软件位置限位 | 范围很小，在机械安全行程内 |
| 写 `0x6081/0x6083` | 设置速度/加速度 | 小速度、小加速度 |

记录模板：

| 对象 | 写入值 | 读回值 | 现象 | 结论 |
|---|---:|---:|---|---|
| `0x6041` | 不写 | | 状态稳定/不稳定 | |
| `0x6064` | 不写 | | 手转增加/减少 | |
| `0x6060` | | `0x6061 = ____` | 模式一致/不一致 | |
| `0x607D` | | | 限位已生效/未确认 | |
| `0x6081` | | | 速度限制 | |
| `0x6083` | | | 加速度限制 | |

如果 `0x6064` 手转不变，先回到编码器链路，不要继续写 `Controlword`。驱动器不知道电机实际位置，运动命令就没有基础。

## 第五步：进入 PPM 前的放行

PPM 首次小步前，必须全部满足：

| 项目 | 放行要求 |
|---|---|
| Z 轴支撑 | 支撑块/夹具/限位已就位 |
| 刹车 | 独立 24 V 释放/抱闸逻辑已记录 |
| 编码器 | `0x6064` 手转连续 |
| 光栅尺 | GM1 高速计数静止不乱跳，移动有方向 |
| EtherCAT | 从站稳定，PDO 关键对象能读写 |
| 软件限位 | `0x607D` 或 GM1 程序限位已小范围设置 |
| 速度/加速度 | `0x6081/0x6083` 很小 |
| 人员 | 一人操作软件，一人守机构/断电 |
| 未闭合项 | Hall 不接，J13 不乱接，GPO 不接刹车 |

不满足任意一项，就不能进入首次小步。

## 第六步：Controlword 状态机只做“可控使能”

DINGS 参考手册说明 `0x6040 Controlword` 触发驱动状态转换，`0x6041 Statusword` 反馈当前状态。实际位模式以 DINGS 手册和 GM1 软件解码为准。新人不要死背十六进制，先学会每一步都看 Statusword。

常见 CiA 402 使能顺序通常是：

```text
Shutdown -> Switch on -> Enable operation
```

现场记录时按“命令含义”写，不要只写一串数：

| 步骤 | 主站动作 | 必须观察 |
|---|---|---|
| 1 | 清故障/复位故障，若有报警 | 报警是否清除 |
| 2 | Shutdown | Statusword 是否进入 Ready to switch on |
| 3 | Switch on | Statusword 是否进入 Switched on |
| 4 | Enable operation | Statusword 是否进入 Operation enabled |

如果 Statusword 不按预期变化：

| 卡住位置 | 优先查 |
|---|---|
| Switch on disabled | 故障、STO、安全输入、驱动参数 |
| Ready to switch on 进不去 | Controlword 顺序、报警未清 |
| Operation enabled 进不去 | STO/限位/刹车/反馈/驱动内部状态 |
| 一使能就报警 | U/V/W、编码器方向、参数、刹车、极对数 |

> ⚠️ Enable operation 不是“开始运动”按钮  
> 它只是允许驱动器出力。Z 轴可能因为刹车释放、重力或参数错误发生动作，所以使能前必须有人守断电。

## 第七步：第一次 PPM 小步

第一次 PPM 不追求距离精度，只验证命令链路和方向关系。

建议动作：

1. 设置 `0x6060` 为 PPM，并确认 `0x6061` 显示一致。
2. 设置很小的 `0x6081 Profile Velocity` 和 `0x6083 Profile Acceleration`。
3. 设置 `0x607D Software Position Limit`，范围只覆盖安全小行程。
4. 读取当前 `0x6064` 作为起点。
5. `0x607A Target Position = 当前 0x6064 + 小增量`。
6. 用 `0x6040` 触发新目标。
7. 观察 `0x6041`、`0x6064`、GM1 光栅尺计数和平台方向。
8. 到位后立即记录，不连续发第二个目标。

记录模板：

| 项目 | 数值/现象 |
|---|---|
| PPM 模式写入 `0x6060` | |
| 模式显示 `0x6061` | |
| 起点 `0x6064` | |
| 目标 `0x607A` | |
| 速度 `0x6081` | |
| 加速度 `0x6083` | |
| 软件限位 `0x607D` | |
| 平台实际方向 | |
| 光栅尺计数增/减 | |
| DS-BVM `0x6064` 增/减 | |
| Statusword 到位/异常 | |

判断：

| 现象 | 判断 | 下一步 |
|---|---|---|
| `0x6064` 变，光栅尺也变，方向关系可解释 | 主命令链路初步通过 | 建立比例和方向表 |
| `0x6064` 变，光栅尺不变 | 电机到平台机械传动断链或刹车/联轴器问题 | 停止，查机械 |
| 光栅尺变，`0x6064` 不变 | 数据读取或外力移动异常 | 停止，查反馈 |
| 一触发就报警 | 状态机、限位、U/V/W、编码器、参数 | 停止，读报警 |
| 方向反 | 不连续运动，回到三本账方向表 | 一次只改一层 |

## 第八步：把 EtherCAT 结果回填到闭环记录

必须回填这些文件：

| 文件 | 回填内容 |
|---|---|
| `../../../../02_实验与现场/01_当前执行/40_逐线验收记录表.md` | EtherCAT 扫描、PDO 对象读写、首次小步记录 |
| `49_零基础闭环控制接线项目手册.md` | 若现场对象名、PDO、状态机有差异，补充项目实测说明 |
| `../../../../02_实验与现场/01_当前执行/50_一页上电放行卡_零基础版.md` | 首次运动前的 EtherCAT、软件限位、三本账 |
| `52_EtherCAT主站导入与PDO联调作业指导_GM1_DS-BVM.md` | 如 GM1 软件界面名称和导入路径已确认，补现场截图编号 |

完成本文件后，你应该能回答：

```text
GM1 为什么要导入 DS-BVM XML/ESI？
最小 RxPDO 和 TxPDO 里有哪些对象？
为什么扫描到从站不等于可以运动？
为什么第一次用 PPM，不直接用 CSP？
第一次小步时，命令位置、电机编码器位置、光栅尺位置分别怎么看？
```

能答出来，说明 EtherCAT 这条“命令链路”不再是黑箱。
