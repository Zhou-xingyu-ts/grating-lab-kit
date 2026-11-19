# 遗留固件架构审计

## 1. 总判断

四个 STM32 工程是不同阶段的演示/骨架，不能拼接成一套已验证固件。最接近论文描述的是 `h750_demo`：STM32H750 + FreeRTOS + CANopenNode + TIM6 静态 1 ms。但它实现的是 STM32 自身作为 CANopen 节点的通用协议处理，没有发现 DS-BVM CiA 402 控制对象、三轴调度、编码器采样或 PID/LQR 控制器。

## 2. 工程级矩阵

| 工程 | MCU/总线 | 源码实际入口 | 静态功能 | 明确缺失 | 证据 |
|---|---|---|---|---|---|
| `h750_demo` | STM32H750VBT6；FDCAN1 classic/normal；1 Mbps | `StartCanopenTask()` 初始化 CANopenNode，本地 Node-ID `0x21`；TIM6 回调调用 `canopen_app_interrupt()` | 单 CANopen 节点、FreeRTOS、通用 NMT/心跳、RPDO/TPDO 处理框架 | DS-BVM 目标节点控制、NMT Master、SDO Client、CiA 402、编码器、三轴、控制算法 | E4-C/E4-P；完整路径见下方 FW-B2-001 至 009 |
| `fdcan_demo` | STM32H743IIT6；FDCAN1 external loopback；1 Mbps | 主循环初始化 CANopenNode，本地 Node-ID 24；定义 SDO upload/download helper | 通用节点模板、SDO helper 源码、对象字典编辑文件 | helper 调用点、有效 SDO Client 编译配置、正常总线模式、1 ms 定时、DS-BVM/CiA 402 | E4-C/E4-P |
| `h750_baremetal` | STM32H750VBT6；双 FDCAN classic/normal | 启动 FDCAN1 和 TIM7，定时发送固定 ID `0x12`、固定 8 字节；CANopen 初始化被注释 | 裸机 CAN 发送演示 | CANopen 运行、接收、PDO、SYNC、DS-BVM、闭环 | E4-C/E4-P |
| `fxxkSTM32` | STM32H743IIT6；未启用 FDCAN/TIM/RTOS | 仅 GPIO 初始化和空主循环 | CubeMX 空白骨架 | 全部 CANopen 与控制功能 | E4-C/E4-P |

## 3. `h750_demo`：最接近论文但仍是节点模板

### FW-B2-001：RTOS 任务结构

- **结论**：代码创建 `defaultTask`、`rttDebugTask`、`canopenTask`；`defaultTask` 与 `canopenTask` 都是 Normal 优先级，调试任务是 Low。只有 `canopenTask` 调用通用 `canopen_app_process()`。
- **标签**：E4-C/E4-P。
- **路径与定位**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\freertos.c` 行 50–70、109–181；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\h750_demo.ioc` 行 16–18。
- **证明了什么**：FreeRTOS 和三任务骨架实际存在。
- **没有证明什么**：没有论文所述“最高优先级 PDO / 中优先级 SDO-NMT-心跳 / 低优先级诊断”三层实现；线程数量相同不能视为功能相同。

### FW-B2-002：CANopenNode 初始化身份

- **结论**：`StartCanopenTask()` 将 STM32 配成 Node-ID `0x21`、1 Mbps 的 CANopen 节点，使用 FDCAN1/TIM6；该 Node-ID 是 STM32 本地节点身份，不是 DS-BVM 目标节点配置。
- **标签**：E4-C。
- **路径与定位**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\main.c` 行 66–85。
- **证明了什么**：存在通用节点初始化和前台处理调用。
- **没有证明什么**：不证明其作为 NMT Master 控制 DS-BVM，也不证明初始化成功。

### FW-B2-003：NMT Master 未启用

- **结论**：归档 `CO_config.h` 的 `CO_CONFIG_NMT` 未包含 `CO_CONFIG_NMT_MASTER` 位；应用源码也未发现 `CO_NMT_sendCommand()` 调用。
- **标签**：E4-P/E4-C。
- **路径与定位**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode\301\CO_config.h` 行 136–150；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\CO_app_STM32.c` 行 43–49、146–166。
- **证明了什么**：当前静态配置是节点自启动/节点状态管理，不是简单 NMT Master 功能。
- **没有证明什么**：不排除另一个未归档分支曾启用 Master；本轮未编译确认宏展开。

### FW-B2-004：SDO Client 未启用

- **结论**：`CO_CONFIG_SDO_CLI` 为 0；尽管本地 OD 声明 1 个 SDO Client 参数记录，功能宏并未启用。
- **标签**：E4-P。
- **路径与定位**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode\301\CO_config.h` 行 354–391；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\OD.h` 行 46；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\OD.c` 行 42–47。
- **证明了什么**：不能把 OD 中的 `0x1280` 记录误读为 SDO Client 已可用。
- **没有证明什么**：未运行预处理器或构建；若有外部编译定义覆盖，需构建日志确认。

### FW-B2-005：FDCAN 收发配置存在静态不一致

- **结论**：Cube 配置将 RX FIFO0/FIFO1 元素数设为 0；移植驱动却将全局过滤器导向 FIFO0并激活 FIFO 新消息通知。
- **标签**：E4-P/E4-C，结论等级 H（兼容性待验证）。
- **路径与定位**：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Core\Src\fdcan.c` 行 40–66；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_demo\Lib\CANOpenStack\CANopenNode_STM32\CO_driver_STM32.c` 行 126、160–161、503、588–589。
- **证明了什么**：存在需要构建/台架核查的板级配置风险。
- **没有证明什么**：不直接证明接收必然失败；本轮禁止构建和硬件测试。

### FW-B2-006：无 DS-BVM/CiA 402 应用层

- **结论**：在四个工程的应用源码和自定义 OD 中未发现 `0x6040/0x6041/0x6060/0x6064/0x607A/0x60FF/0x6071`，也未发现 controlword/statusword/目标位置/速度/转矩应用代码。
- **标签**：E4-C/E4-P。
- **路径与定位**：四工程的 `Core/Src/` 或 `Src/`；`h750_demo` 与 `fdcan_demo` 的 `Lib/CANOpenStack/CANopenNode_STM32/OD.c`、`OD.h`；静态精确检索无匹配。
- **证明了什么**：归档应用层没有可定位的 DS-BVM CiA 402 控制实现。
- **没有证明什么**：不能证明硬件从未被其他软件控制；也不能替代对未归档分支的调查。

### FW-B2-007：无编码器与三轴调度

- **结论**：应用入口没有正交编码器/Hall/光栅采集，也没有三轴对象、轴循环或三驱动 PDO 调度。
- **标签**：E4-C。
- **路径与定位**：`h750_demo/Core/Src/main.c` 行 66–85、120–136、250–263；`fdcan_demo/Src/main.c` 行 121–150、213–306；`h750_baremetal/Core/Src/main.c` 行 67–75、122–155；完整工程路径同本文件前述来源。
- **证明了什么**：论文图 2.12 的“读取编码器—计算—三轴 PDO”未落在这些入口中。
- **没有证明什么**：不排除编码器由驱动器内部读取；但那不是 STM32 外环源码证据。

## 4. 另外三个工程的边界

### FW-B2-020：`fdcan_demo`

`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\fdcan_demo\Src\main.c` 行 213–306 确实定义了 SDO upload/download helper；但静态调用检索只命中声明与定义，没有实际调用点。`fdcan_demo.ioc` 行 8、16 和 `Src/fdcan.c` 行 41–43 又把总线设为 1 Mbps external loopback。加上 TIM6 不是 1 ms，故该工程只能作为 API/移植参考，不能作为 DS-BVM 实物 SDO/PDO 证据。

### FW-B2-021：`h750_baremetal`

`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\h750_baremetal\Core\Src\main.c` 行 122–129 将 CANopen 初始化整段注释；行 131–145 启动 FDCAN1，行 67–75 只在 TIM7 回调发送固定帧。因此它不是 CANopen 闭环工程。

### FW-B2-022：`fxxkSTM32`

`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\赖建君毕业归档资料\2022级-赖建君-毕业归档资料\2025级-赖建君-毕业归档资料\05.实验原始数据代码及说明\单片机运动控制\STM32\fxxkSTM32\fxxkSTM32.ioc` 行 12–20 只启用核心、内存、NVIC、RCC、SYS；`Core/Src/main.c` 行 93–100 只有 GPIO 初始化与空循环。它不能补齐其他工程的功能缺口。

## 5. 工程完整性

四个工程均保留了 `.ioc`、启动/链接配置及 HAL Drivers；`h750_demo` 另含 FreeRTOS、CANopenNode、对象字典与 CMake 子模块，源码形态上具备继续审查和重新集成的价值。但：

- 本轮没有任何编译通过证据；
- 没有可执行固件哈希、编译器版本锁定、烧录记录和运行日志；
- `h750_demo` CMake 对 Cortex-M7 使用 `ARM_MATH_CM4`/`ARM_CM4F` 相关定义或 include，兼容性未验证；
- `fdcan_demo` 的 SDO helper 与 `CO_CONFIG_SDO_CLI=0` 存在静态配置疑点；
- 无 CAN 收发器原理图/板号/终端电阻证据；
- 无 DS-BVM 节点参数、PDO 映射、NMT/状态机配置备份。

因此工程完整性结论只能是“保留了可供静态复用的工程骨架”，不能写成“可直接编译烧录并控制现台架”。
