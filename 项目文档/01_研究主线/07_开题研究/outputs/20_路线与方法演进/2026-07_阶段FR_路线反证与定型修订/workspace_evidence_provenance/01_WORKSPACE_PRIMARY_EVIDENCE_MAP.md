# 工作区原始证据地图

## 1. 使用规则

本图只回答“原始证据在哪里、适用于哪套台架、能证明到哪一步”。标签严格采用本任务给定的来源类型。多份派生文档引用同一材料时，仍只计一个独立证据族。

路径缩写：`ROOT` = `D:\清华\清华-科研横向\光栅尺运动控制调研\`；`LEGACY_ARCHIVE` = `ROOT\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\`；`LEGACY_DATA` = `LEGACY_ARCHIVE\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\`。

## 2. 当前商业单轴台架

| 证据 ID | 来源类型 | 精确路径与定位 | 时间/台架 | 能证明什么 | 不能证明什么 |
|---|---|---|---|---|---|
| `E-CUR-01` | `WORKSPACE_PRIMARY_NOTE` | `02_实验与现场/01_当前执行/60_实验室推进记录/2026-06-18_光栅尺单反馈闭环_Scope数据采集_10Hz限制确认.md`，5、17、22、35、55—57 行 | 2026-06-18，当前 Z 轴 | Ri0040 经数字编码器口作为当次唯一反馈；Servo Studio 运动；Scope CSV 约 10 Hz | 精确 DS 型号、长期配置、正式精度、同步采样 |
| `E-CUR-02` | `WORKSPACE_PRIMARY_RUNTIME` | `02_实验与现场/01_当前执行/62_Scope双总线数据采集/scope_usb_2026-06-18.csv`，时间列；`scope_can622_2026-06-22.csv`，时间列 | 2026-06-18/22，当前 Z 轴 | 两次低频目标/实际/误差序列，时间间隔约 0.1 s | 文件名中的 `622` 不是合法 Node-ID 证据；不是高频动态证明 |
| `E-CUR-03` | `WORKSPACE_PRIMARY_NOTE` | `02_实验与现场/01_当前执行/60_实验室推进记录/2026-07-12_USB控制_CANopen只读_50Hz动态采集.md`，4—5、15—18、24—28、42—59、127—137、141—203 行 | 2026-07-12，当前 Z 轴 | USB/COM 发运动；CAN 侧仅 SDO upload；Node 1、1 Mbps；50/100/200 Hz 口径；往返、参数阶梯和边界 | CANopen 写控制、PDO/SYNC、正式标定、最终定位/重复精度 |
| `E-CUR-04` | `WORKSPACE_PRIMARY_RUNTIME` | `motion_acquisition/logs/motion_sdo_20260712_195923.csv` + `.manifest.json`；CSV 字段 `sample_seq/object_key/parsed_value/read_status`，manifest `acquisition/statistics/safety_boundary` | 2026-07-12 19:59，当前 Z 轴 | 1,500 轮、10,500 次对象读取、0 失败；请求 20 ms；7 对象；只读边界；往返值 | `estimated_rate_hz=95.53` 不是实际 50 Hz 轮次；同轮对象不是同时捕获 |
| `E-CUR-05` | `WORKSPACE_PRIMARY_RUNTIME` | `motion_acquisition/logs/motion_sdo_20260712_203414.csv` + `.manifest.json`；同上字段 | 2026-07-12 20:34，当前 Z 轴 | 三核心对象、请求 5 ms、约 199.75 Hz 实际轮次；11,986 轮；35 次处理时间超过目标周期；三组轨迹中的补采段 | 不含状态字/错误码/电流同速采集；不是驱动内部控制周期 |
| `E-CUR-06` | `WORKSPACE_PRIMARY_CONFIG` | `motion_acquisition/configs/canopen_readonly.local.yaml`，1—16 行 | 2026-07-12 现场确认后配置 | `PCAN_USBBUS1`、1 Mbps、Node 1、默认 20 ms | 配置文件本身不能证明设备响应，需与 manifest/CSV 联用 |
| `E-CUR-07` | `WORKSPACE_PRIMARY_VISUAL` | `90_原始资料/2026-06-01_聊天记录补充资料/` 原图；语义索引 `02_实验与现场/01_当前执行/58_现场实拍确认事实索引.md`，25—36 行 | 2026-06，当前台架 | H743IIT6 芯片、DINGS 接口板 ECAT/CAN/BiSS&COM/Digital I/O、Ri/电机/台架可见 | 精确 DS chassis 型号、端到端电气连续性、固件身份、运动安全 |
| `E-CUR-08` | `WORKSPACE_PRIMARY_NOTE` | `02_实验与现场/01_当前执行/46_未闭环逐线缺口清单.md`，12—15、29、35—48、60、74—78 行 | 当前执行层 | 已关闭/未关闭项、安全边界、单编码器接口约束、Hall/STO/限位/刹车缺口 | 空白验收格不能替代测量；部分旧“622”文字已被 07-12 证据替代 |

## 3. 当前采集源码与时间语义

| 证据 ID | 来源类型 | 精确路径与定位 | 能证明什么 | 不能证明什么 |
|---|---|---|---|---|
| `E-SRC-01` | `SOURCE_CODE_STATIC` | `motion_acquisition/src/motion_acquisition/canopen_client.py`，24、48、73、98—104 行 | 实物调用只有 `node.sdo.upload(...)`；RemoteNode 无 EDS 也可构造 | 不证明包的每次运行都成功；由 CSV/manifest 补强 |
| `E-SRC-02` | `SOURCE_CODE_STATIC` | `motion_acquisition/tests/test_no_write_api.py`，16—26 行 | 静态守卫禁止 `.sdo.download`、Operational NMT、PDO/周期发送和控制字/目标位置写入字样 | 本轮未重跑测试；测试文件存在不等于运行过 |
| `E-SRC-03` | `SOURCE_CODE_STATIC` | `motion_acquisition/src/motion_acquisition/manifest.py`，58—83 行 | manifest 写入 bitrate、node、对象、周期及只读安全声明 | 声明需要与源码扫描和运行文件交叉核对 |
| `E-SRC-04` | `SOURCE_CODE_STATIC` | `motion_acquisition/src/motion_acquisition/sampling_stats.py`，30—42 行 | `estimated_rate_hz = 1 / mean_cycle_s`，其中 cycle 是一轮对象读取的处理耗时 | 不是 `sample_seq` 的实际输出频率，也不是设备控制周期 |
| `E-SRC-05` | `SOURCE_CODE_STATIC` | `motion_acquisition/src/motion_acquisition/acquisition.py`，25—47、50—84 行 | 每轮开始前只取一次 `wall/mono/elapsed`，所有对象共用该轮时间戳；对象随后依次 SDO 读取；按 `next_deadline` 调度 | 同轮不同对象不是原子快照；CSV 无各对象请求/响应完成时间 |
| `E-SRC-06` | `SOURCE_CODE_RUNTIME` | `motion_acquisition/logs/*.csv` 与同名 `.manifest.json` 共 14 对 | 当前源码输出模式与 2026-07-12 实物数据配对；各行 Node 1 且 `read_status=ok` | manifest `git_commit=null`，无法精确到源代码提交 |

## 4. 官方手册、ESI 与当前设备匹配边界

| 证据 ID | 来源类型 | 精确路径与定位 | 能证明什么 | 不能证明什么 |
|---|---|---|---|---|
| `E-MAN-01` | `MANUAL_CAPABILITY` | `90_原始资料/官方手册/` 中 DINGS DS-BVM-FETC/FCAO 系列硬件/CANopen资料 | 系列接口、J8/J9、对象和 CiA 402 能力 | 当前 chassis 精确后缀、固件、PDO 当前值、现场放行 |
| `E-MAN-02` | `WORKSPACE_PRIMARY_CONFIG` + `MANUAL_CAPABILITY` | `90_原始资料/官方软件/DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网/WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，19、41、3891—4185、4519—4807、10278—10527、11529—12043 行 | 工作区确有产品码 `0x1`、修订 `0x20200525`、Profile 402、固定 Rx/TxPDO、DC-Synchron/FreeRun 的系列 ESI | 当前实物是否就是该 ProductCode/Revision；是否已导入、扫描、进入 OP 或运行 PDO/DC |
| `E-MAN-03` | `MANUAL_CAPABILITY` | Renishaw ATOM Data Sheet/Installation Manual；摘要定位 `02_实验与现场/01_当前执行/58_现场实拍确认事实索引.md` 31 行 | Ri0040 命名为 40× 插补；对 20 μm 栅距理论 0.5 μm/count | 当前安装尺度的外部计量溯源、方向、非线性、装配误差 |

## 5. 遗留 STM32 资产

下表中的 `LEGACY_STM32_ROOT` 指：

`90_原始资料/赖建君毕业归档资料/2022级-赖建君-毕业归档资料/2025级-赖建君-毕业归档资料/05.实验原始数据代码及说明/单片机运动控制/STM32/`

| 证据 ID | 来源类型 | 精确路径与定位 | 时间/变体 | 能证明什么 | 不能证明什么 |
|---|---|---|---|---|---|
| `E-STM-01` | `SOURCE_CODE_STATIC` | `LEGACY_STM32_ROOT/fdcan_demo/fdcan_demo.ioc`，16—20、28、37—39、206—208 行；`Src/fdcan.c` 42—50 行 | 2025-04，H743IIT6 | H743 工程、FDCAN1、1 Mbps 配置线索；但为 External Loopback；TIM6 参数对应约 0.5 s 而非 1 ms | 正常总线、主站、1 ms 周期、当前实物运行 |
| `E-STM-02` | `SOURCE_CODE_STATIC` | `LEGACY_STM32_ROOT/fdcan_demo/Src/main.c`，29、58—62、213—306 行 | H743 工程 | CANopenNode 初始化及 SDO upload/download 辅助函数存在 | 函数被调用、DS-BVM 访问、现场写入、主站完整性 |
| `E-STM-03` | `SOURCE_CODE_STATIC` | `LEGACY_STM32_ROOT/fdcan_demo/Lib/CANOpenStack/CANopenNode_STM32/OD.c`，23—50、72—150 行 | H743 工程对象字典 | `0x1006=0`；SDO client COB-ID 禁用；RPDO 0x1600—1603 映射数为 0；TPDO1 仅映射本地 0x2001 | 有效 SYNC producer、DS-BVM 6040/607A 映射、现场 PDO |
| `E-STM-04` | `SOURCE_CODE_STATIC` | `LEGACY_STM32_ROOT/h750_baremetal/h750_baremetal.ioc`，13—24、31、45、258—264 行；`Core/Src/main.c` 31、127 行 | 2025-04，H750VBT6 | Normal FDCAN、TIM6 配置可对应 1 ms；CANopen include/init 被注释 | 活跃 CANopen 应用、当前 H743 固件、运行周期 |
| `E-STM-05` | `SOURCE_CODE_STATIC` | `LEGACY_STM32_ROOT/h750_demo/h750_demo.ioc`，18、26、38、211—213 行；`Core/Src/main.c`，66—85、261 行；`Core/Src/freertos.c`，51—79、111—117 行 | 2025-04/05，H750VBT6 | FreeRTOS 三任务；CANopen node ID `0x21`、1 Mbps；任务循环调用 stack；TIM6 约 0.5 ms、任务注释约 1 ms | NMT Master、SYNC producer、DS-BVM PDO/CiA402、真实周期/抖动 |
| `E-STM-06` | `SOURCE_CODE_RUNTIME` | `LEGACY_STM32_ROOT/.metadata/.plugins/org.eclipse.cdt.ui/CANOpenNode_CubeIDE.build.log`，1—17 行 | 2025-04-12，历史 H743 | 生成 `CANopenNode_CubeIDE.elf`，0 errors/0 warnings | ELF 已不在归档；不证明烧录或运行 |
| `E-STM-07` | `SOURCE_CODE_RUNTIME` | `LEGACY_STM32_ROOT/.metadata/.plugins/org.eclipse.debug.ui/launchConfigurationHistory.xml`，5、19 行 | 2025-04，历史 IDE | Debug launch 配置被使用过 | 未给出 programmer 成功、Flash 校验、串口/RTT/CAN 输出，故仅部分支持“烧录尝试” |

## 6. 历史论文、PPT、运动台和 GM1

| 证据 ID | 来源类型 | 精确路径与定位 | 能证明什么 | 不能证明什么 |
|---|---|---|---|---|
| `E-TH-01` | `THESIS_CLAIM` | `90_原始资料/赖建君毕业归档资料/2022级-赖建君-毕业归档资料/01.学位论文相关/1.1 学位论文PDF最终版本/thuthesis.pdf`，PDF 40—41、48、50、70—74、82 页 | 作者关于三轴、CANopen/STM32、1 ms、333 μs、53.3%、LQR/PID 仿真、±0.27 μm 的陈述 | 这些陈述不自动构成当前台架或固件运行事实；333 μs/53.3% 是报文长度算术模型 |
| `E-TH-02` | `THESIS_CLAIM` | 同级 `1.2 学位论文答辩PPT/正式答辩.pptx`，第 8、9、15、19 页 | 复述同一研究中的 1 ms/333 μs/53.3%、LQR、±0.27 μm、三轴 | 与论文同源，不算独立复现实验 |
| `E-TH-03` | `WORKSPACE_PRIMARY_RUNTIME`（历史） | `LEGACY_DATA\实验数据\运动台\全行程定位0.csv`、`1.csv`、`2.csv`、`plot.ipynb` | 旧平台三个全行程定位数据集及13个激光干涉仪重复测量值 | 缺仪器原始导出、校准证书、完整环境/配置；不适用于当前Z轴 |
| `E-TH-04` | `WORKSPACE_PRIMARY_VISUAL`（历史） | `LEGACY_ARCHIVE\01.学位论文相关\1.3 学位论文中图片汇总\z设计图.png`、`xy堆叠设计图.png`、`xy堆叠爆炸图.png`、`xy堆叠实物图.jpg`、`运动台验证实验平台.jpg` | 历史三轴机械概念图、爆炸图和实物/实验照片 | 无可编辑CAD、BOM、装配验收，也不是当前台架三轴证据 |
| `E-GM1-01` | `SOURCE_CODE_STATIC` | `LEGACY_DATA\上位机\GM1\Sync\GM1_Modbus.py`，3—4、20、31—39、78—129、155—159行；`LEGACY_DATA\上位机\GM1\Async\GM1_Modbus.py`同类 | 历史上位机通过Modbus TCP访问GM1，含reset ECAT、2D目标/使能寄存器 | 不是EtherCAT Master工程；无当前扫描/PDO/DC/DS-BVM运行日志 |
| `E-GM1-02` | `SOURCE_CODE_STATIC` | `LEGACY_DATA\上位机\GM1\Sync\GM1Widget.py`，47—49、107—109、250—252行 | GM1 IP `192.168.2.5`、端口502与复位界面静态逻辑 | 不证明GM1当前可借、可上电或与当前DS相连 |

## 7. 派生与离线材料的正确定位

| 证据层 | 代表路径 | 正确用途 | 禁止升级 |
|---|---|---|---|
| `WORKSPACE_DERIVED` | `01_研究主线/07_开题研究/outputs/2026-07_阶段A_*` 至阶段 FR；`02_实验与现场/01_当前执行/00_总览.md` | 入口、状态整合、缺口和路线判断 | 不以 HANDOFF 数量代替原始日志/配置 |
| `OFFLINE_METHOD` | 阶段 D、D1、E 的模型、合成数据、仿真与协议 | 方法设计、风险和未来验收标准 | 不证明 CANopen/EtherCAT/三轴/外位置环已实现 |
| `USER_REPORTED` | `5 mm≈10000 counts`、部分标签/配置口述 | 明确记录待验证信息 | 不作为正式标定或精确型号 |
| `EXTERNAL_CONFIRMATION` | 自研尺规格/交付、厂家型号/安全接口、可借控制器 | 建立外部问题清单 | 在未获得材料前不得写成事实 |

## 8. 路线证据链摘要

```text
当前可直接复核：
现场图/记录 -> USB Servo Studio 运动 -> SDO upload 源码/配置
-> 14 对 CSV+manifest -> Node 1 / 1 Mbps / 10-200 Hz 不同口径

仅历史静态/声明：
师兄论文/PPT + H743/H750 工程 + 编译日志 + GM1 上位机源码
-> 不能跨越为当前 STM32 主站、当前 EtherCAT 或三轴运行

仅能力：
DINGS 手册/ESI -> 产品系列可有 CANopen/EtherCAT/CiA402/PDO/DC
-> 不能跨越为当前精确型号、配置或现场可用
```

因此，FR 的最近期证据主线仍以当前商业单轴为基座，但理由由“阶段 A 摘要”升级为可追到原始运行、配置和源码的证据链；STM32 与 EtherCAT 两分支都保留，且各自的缺口更精确。
