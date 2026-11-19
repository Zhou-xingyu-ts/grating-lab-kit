# 定时器、CANopen、PDO 与 SYNC 审计

## 1. 1 ms 定时器逐工程核查

静态周期按 `.ioc` 给出的 TIM 时钟、Prescaler 和 Period 计算；这是配置推导 `H`，不是示波器测量或运行证据。

| 工程 | TIM 时钟与配置 | 静态周期 | 中断入口/是否启动 | 结论 |
|---|---|---|---|---|
| `h750_demo` | `RCC.Tim2OutputFreq_Value=200000000`；TIM6 Prescaler=`100-1`、Period=`1000-1` | 1 ms | `canopen_app_init()` 行 176–177 启动 TIM6；`main.c` 行 250–263 回调进入 `canopen_app_interrupt()` | 存在静态可达的 1 ms CANopenNode 协议处理路径；未证明实际抖动或总线闭环 |
| `h750_baremetal` | 200 MHz；TIM6 Prescaler=`200-1`、Period=`1000-1` | 1 ms | CANopen 初始化和 TIM6 启动被注释；实际启动 TIM7，回调只发送固定帧 | “有 1 ms 配置”不等于 1 ms CANopen 正在运行 |
| `fdcan_demo` | 200 MHz；TIM6 Prescaler=`10000-1`、Period=`10000-1` | 0.5 s（2 Hz） | CANopenNode 会启动 TIM6；中断处理函数却硬编码 `timeDifference_us=1000` | 配置周期与协议栈时间增量相差 500 倍，不能继承为 1 ms 实现 |
| `fxxkSTM32` | 无 TIM6 | 不适用 | 无 | 无定时控制证据 |

### TMR-B2-001

- **结论**：只有 `h750_demo` 同时满足“TIM6 静态 1 ms + 初始化路径启动 TIM6 + 回调调用 CANopenNode 周期函数”。
- **标签**：E4-P/E4-C；周期值为基于配置的 H。
- **路径**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\h750_demo.ioc` 行 164、200、212–213；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\tim.c` 行 42–45；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\main.c` 行 250–263；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\CO_app_STM32.c` 行 166–177、230–248。
- **证明了什么**：源码结构上存在 1 ms 协议处理定时路径。
- **没有证明什么**：不证明中断每次按期到达、处理在 1 ms 内完成、CAN 帧发送完成、DS-BVM响应或闭环稳定。

## 2. CAN 与 FDCAN

四个工程使用 STM32H7 的 FDCAN 外设，但有效帧格式均为 Classic CAN；论文中的“CAN-FD 最高 5 MHz”是芯片能力描述，不是本工程 CANopen 运行速率。`h750_demo` 和 `h750_baremetal` 配为 Normal，`fdcan_demo` 配为 External Loopback；三者 nominal bitrate 静态配置均为 1 Mbps。

证据路径：

- `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\fdcan.c` 行 40–49；
- `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_baremetal\Core\Src\fdcan.c` 行 41–50、89–98；
- `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\fdcan_demo\Src\fdcan.c` 行 41–49。

这证明了外设配置，不证明物理收发器、终端、布线和实物误码率。

## 3. CANopenNode 来源与处理路径

### CAN-B2-010：协议栈来源

`h750_demo/Lib/CANOpenStack/CANopenNode_STM32/CO_app_STM32.c` 文件头行 2–13 明示其为开源 CANopenNode 移植；`CO_app_STM32.c` 行 120–174 顺序执行 `CO_CANinit`、`CO_CANopenInit`、`CO_CANopenInitPDO`，行 192–193 进入 normal mode。该证据说明通用协议栈被纳入工程，不说明 DS-BVM 设备画像和控制应用已完成。

### CAN-B2-011：前台与定时处理

- `canopen_app_process()` 使用 `HAL_GetTick()` 的实际毫秒差调用 `CO_process()`，负责本地节点 NMT/心跳等前台处理；
- `canopen_app_interrupt()` 每次固定向 `CO_process_SYNC/RPDO/TPDO` 传入 1000 μs；
- 固定 1000 μs 并未从定时器实测得到，所以在 `fdcan_demo` 的 0.5 s 配置中会形成明显错误时间语义。

完整路径：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\CO_app_STM32.c` 行 201–248。

## 4. 对象字典与 PDO 映射

### PDO-B2-001：这是 STM32 本地 PDO，不是 DS-BVM PDO

`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\OD.c` 的默认值为：

- 行 48–70：4 个 RPDO 的 COB-ID 均含失能位，Transmission Type 为 `0xFE`；
- 行 72–115：4 个 RPDO 映射数量均为 0；
- 行 116–122：TPDO1 COB-ID `0x40000180`，Transmission Type `0xFF`，Event Timer 1 ms；
- 行 148–157：TPDO1 只映射本地厂商对象 `0x2001:00`（32 bit）；
- 行 205：本地 `0x2001 app` 初值 25。

因此，源码存在 `CO_process_RPDO/TPDO()` 并不等于已配置面向 DS-BVM 的 PDO。默认 RPDO 没有目标对象，唯一启用的 TPDO 发送 STM32 自身的本地变量。

### PDO-B2-002：无 CiA 402 映射

本地 OD 与应用源码均未发现 DS-BVM 的 `0x6040/0x6041/0x6060/0x607A/0x60FF/0x6071`。故没有证据表明 Controlword、Statusword、Modes of operation、目标位置/速度/转矩已映射或发送。

### PDO-B2-003：`STM32Master.xdd` 名称不构成功能证据

`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\fdcan_demo\STM32Master.xdd` 行 18–22 将产品命名为 `STM32Master`，行 2308–2309 声明 4 RxPDO、4 TxPDO 并保留空的 `<CANopenMasterFeatures />`。它生成的是本地 DS301 对象字典，不能证明 NMT Master、DS-BVM PDO master 或三轴主站已经实现。

## 5. SYNC 与 NMT

### SYNC-B2-001

`OD.c` 行 25–27 将 `0x1005` 设为 `0x80`、`0x1006` 设为 0。该默认值没有启用周期性 SYNC 生产；`CO_process_SYNC()` 只证明协议栈具备处理同步对象的通用代码路径。没有应用调用或配置证明 STM32 正在以 1 kHz 生产 SYNC。

### NMT-B2-001

`CO_config.h` 行 147 的 NMT 配置不含 `CO_CONFIG_NMT_MASTER`；应用也未调用 `CO_NMT_sendCommand()`。`CO_app_STM32.c` 行 44 的 `CO_NMT_STARTUP_TO_OPERATIONAL` 控制的是 STM32 本地节点的自启动行为，不是把 DS-BVM 切到 Operational。

## 6. SDO helper 边界

`fdcan_demo/Src/main.c` 行 213–306 包含 `read_SDO()` 和 `write_SDO()` 的完整 helper 函数定义，但：

1. 仅发现声明和定义，未发现调用；
2. 同归档 `CO_config.h` 行 372 将 SDO Client 配为 0；
3. `fdcan_demo` 运行模式是 external loopback；
4. 没有 DS-BVM 对象索引、节点状态或实验记录。

故该 helper 只能列为“可参考源码”，不能写成“已对 DS-BVM 完成 SDO 读写”。尤其不能继承其 download helper 后直接运行。

## 7. 论文 333 μs/1 ms 的证据边界

论文 PDF 页 40 / 印刷页 22 明确在“忽略数据处理与算法执行时间”的条件下计算 `111 μs × 3 = 333 μs`。源码没有三轴队列、三驱动 COB-ID、响应帧调度、错误恢复或测时记录。因此：

- `333 μs` 是理想化设计计算（E5-D）；
- `h750_demo` 的 1 ms 是静态定时配置（E4-P + H）；
- 两者均不是“CANopen 三轴闭环稳定 1 kHz”的实验验证；
- 阶段 B1 又未找到 DS-BVM 官方 1 kHz 周期保证，后续架构不得把 1 kHz 当成已确认参数。
