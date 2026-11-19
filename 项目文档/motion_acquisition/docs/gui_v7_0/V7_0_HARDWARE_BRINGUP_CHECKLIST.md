# V7.0 未来 H743/三驱动器上板清单

此清单不是当前执行授权。V7 包未烧录，未连接任何硬件。

## 1. 板卡身份与供电

- [ ] 记录 H743 核心板/底板完整型号、原理图版本、MCU 标识；
- [ ] 确认 HSE/LSE、供电、电压、启动脚、调试器和复位；
- [ ] 复核 linker memory 与实际器件；
- [ ] 决定 cache/MPU/DMA 策略；
- [ ] 先以无外设固件验证 reset、SysTick、fault diagnostics；
- [ ] 确认硬件 IWDG/WWDG 方案后再实现 service port。

## 2. PC 物理链路

- [ ] 选择 UART/USB CDC/Ethernet；
- [ ] 确认引脚、电平、隔离、波特率/端点、驱动和缓冲；
- [ ] 将 ByteStreamPort 绑定为非阻塞 partial read/write；
- [ ] 验证 fragmentation、backpressure、断线、重连、CRC/错误策略；
- [ ] 保持 protocol 4.0，不绕过 capability/endpoint policy。

## 3. FDCAN/CANopen

- [ ] 确认 FDCAN instance、引脚复用、收发器、CANH/CANL/GND；
- [ ] 确认 1 Mbps 是否仍为最终总线值、采样点和时钟；
- [ ] 确认终端、电缆、隔离和 bus-off 恢复；
- [ ] 为 ISR/mainline 绑定 SPSC queue，先做 loopback/分析仪；
- [ ] 逐台读取 boot-up/heartbeat、0x1000/1001/1018；
- [ ] 记录三个实际 Node-ID，禁止沿用测试 11/22/33；
- [ ] 获得对应型号/固件 EDS 或厂商对象清单；
- [ ] 读取 0x6041/6061/6064/6502 并与 Servo Studio 对照；
- [ ] 单独评审后才能放开 NMT、SDO download、PDO/SYNC。

## 4. 驱动器与反馈

- [ ] 确认每轴唯一权威反馈源和接入位置；
- [ ] 确认 counts、方向、溢出、有效位、时间戳和失联行为；
- [ ] 方案 A：确认驱动内部位置闭环和目标位置命令接口；
- [ ] 方案 B：确认 STM32 直读反馈接口和速度/转矩命令能力；
- [ ] 为每个 drive profile 固化 identity、mode、对象、PDO、limits；
- [ ] raw counts 始终保留，物理单位需正式标定。

## 5. 安全

- [ ] 独立确认 E-stop、STO、J13、制动器、限位和电源切断路径；
- [ ] 记录极性、有效电平、断线失效状态、响应时间和复位条件；
- [ ] PC protective-stop 不得代替 E-stop/STO；
- [ ] 软件 watchdog 不得标为 safety-rated；
- [ ] 任何 enable/motion 前完成独立安全评审和低能量单轴放行。

## 6. 分级验收

1. 无总线启动与 debugger diagnostics；
2. PC 物理协议 hello/poll/disconnect；
3. CAN 只接分析仪/loopback；
4. 单驱动只读 discovery；
5. 三驱动只读 heartbeat/EMCY；
6. 经审批的 NMT/SDO/PDO；
7. 单轴低能量；
8. 三轴同步；
9. 性能、异常、断线与安全；
10. 第二台 Windows 部署。

