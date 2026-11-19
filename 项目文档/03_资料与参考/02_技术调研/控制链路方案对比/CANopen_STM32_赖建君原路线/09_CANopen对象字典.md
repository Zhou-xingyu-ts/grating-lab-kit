# CANopen对象字典速查

> 资料定位：这是赖建君原系统/归档 STM32 工程的 CANopen 对象字典速查，用于原路线理解和备选研究，不是当前台架默认接线方案。

> 来源：`h750_demo/Lib/CANOpenStack/CANopenNode_STM32/DS301_profile.md`
> 控制器节点ID：**0x21**（由 `h750_demo/main.c:73` 确认）
> 波特率：**1000 kbps**（1 Mbps）

> 现场适用性说明（2026-05-28）：本文件只描述归档 STM32/CANopen 工程中的对象字典和 CAN 配置，不等同于当前台架实际接线。当前照片中更明确的运动主控是 Panasonic `GM1 CSEC16T`，且 DINGS 驱动器与 GM1 都有 EtherCAT 相关接口。因此本文件只能作为“若确认走 STM32/CANopen 时”的备查资料；不能作为当前上电接线或 GM1->DINGS EtherCAT 配置依据。

---

## 1. 节点配置

| 参数 | 值 |
|------|-----|
| Node ID | **0x21** (33) |
| Baudrate | 1000 kbps |
| CAN外设 | FDCAN1 |
| 引脚 | PA11 (RX), PA12 (TX) |
| CAN模式 | Classic CAN (非FD模式) |
| PDO数量 | 4 RPDO / 4 TPDO |
| LSS | Slave = True |
| 定时器 | TIM6, 1ms中断 |
| 主循环 | 1ms `vTaskDelay` |

---

## 2. COB-ID映射表（nodeID=0x21）

### 网络管理

| 对象 | COB-ID公式 | 实际值 | 说明 |
|------|-----------|--------|------|
| NMT | 0x000 | 0x000 | 网络管理命令（所有节点接收） |
| SYNC | 0x080 | 0x080 | 同步报文 |

### 服务数据对象 (SDO)

| 对象 | COB-ID公式 | 实际值 | 方向 |
|------|-----------|--------|------|
| SDO (rx) | 0x600 + $NODEID | **0x621** | PC → 控制器 |
| SDO (tx) | 0x580 + $NODEID | **0x5A1** | 控制器 → PC |

### 过程数据对象 (PDO) — 接收 (RPDO)

| 对象 | COB-ID默认值(nodeID=0x21) | 传输类型 | 用途 |
|------|--------------------------|---------|------|
| RPDO1 | 0x221 (bit31=1 → 默认禁用) | 254 (事件驱动) | 控制指令（目标位置等） |
| RPDO2 | 0x321 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |
| RPDO3 | 0x421 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |
| RPDO4 | 0x521 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |

### 过程数据对象 (PDO) — 发送 (TPDO)

| 对象 | COB-ID默认值(nodeID=0x21) | 传输类型 | 用途 |
|------|--------------------------|---------|------|
| TPDO1 | 0x1A1 (bit31=1 → 默认禁用) | 254 (事件驱动) | 状态反馈（当前位置等） |
| TPDO2 | 0x2A1 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |
| TPDO3 | 0x3A1 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |
| TPDO4 | 0x4A1 (bit31=1 → 默认禁用) | 254 (事件驱动) | 备用 |

### 紧急报文

| 对象 | COB-ID公式 | 实际值 |
|------|-----------|--------|
| EMCY | 0x80 + $NODEID | **0xA1** |

---

## 3. COB-ID位域说明

**PDO COB-ID** (32位)：
- bit 31: 1 = PDO不存在/无效（默认禁用）
- bit 30: RTR允许标志（仅TPDO）
- bit 11-29: 保留（=0）
- bit 0-10: 11位CAN-ID

**SDO COB-ID** (32位)：
- bit 31: 1 = SDO不存在/无效
- bit 30: 1 = 动态分配
- bit 11-29: 保留（=0）
- bit 0-10: 11位CAN-ID

---

## 4. 传输类型配置

所有PDO默认传输类型 = **254**（事件驱动/异步）：

| 值 | 含义 |
|----|------|
| 0 | 同步（非周期，收到SYNC后发送） |
| 1-240 | 同步（周期，每N个SYNC发送一次） |
| 254 | **事件驱动（制造商特定）** ← 当前配置 |
| 255 | 事件驱动（设备子协议特定） |

---

## 5. PDO映射配置

PDO映射对象字典：
- RPDO映射: 0x1600-0x1603
- TPDO映射: 0x1A00-0x1A03

**当前所有PDO映射均为禁用状态**（映射数量=0），需要：
1. 通过SDO写入PDO映射参数（0x1600/0x1A00）
2. 通过SDO启用对应PDO（清除COB-ID的bit31）
3. 配置驱动器侧的对象字典以匹配

---

## 6. 操作流程

### 上电后CANopen初始化顺序
```
1. NMT复位 (0x000, command=0x81→复位所有节点)
2. SDO配置驱动器PDO映射 (向驱动器0x600+driveID写入)
3. SDO配置控制器PDO映射 (向0x621写入0x1600/0x1A00)
4. SDO启用PDO (清除COB-ID bit31)
5. NMT启动 (0x000, command=0x01→进入Operational)
```

### 运动控制数据流
```
PC → 控制器(SDO 0x621/0x5A1): 参数配置
控制器 → 驱动器(RPDO 0x221→驱动器COB-ID): 控制指令(目标位置)
驱动器 → 控制器(驱动器TPDO→0x1A1): 状态反馈(当前位置、速度)
控制器 → PC(TPDO 0x1A1→PC侧监听): 实时数据(可选)
```

---

## 7. 必备对象字典（DS301）

| 索引 | 名称 | 说明 |
|------|------|------|
| 0x1000 | Device type | 设备类型 |
| 0x1001 | Error register | 错误寄存器 |
| 0x1005 | COB-ID SYNC | SYNC报文COB-ID (默认0x80) |
| 0x1006 | Communication cycle period | SYNC周期 (µs, 0=禁用) |
| 0x1014 | COB-ID EMCY | 紧急报文COB-ID (0x80+nodeID) |
| 0x1017 | Producer heartbeat time | 心跳周期 (ms, 0=禁用) |
| 0x1018 | Identity | 设备标识（厂商ID/产品码/版本/序列号） |

---

## 8. FDCAN硬件配置

来源：[`h750_baremetal/Core/Src/fdcan.c`](../../../../90_原始资料/赖建君毕业归档资料/2022级-赖建君-毕业归档资料/2025级-赖建君-毕业归档资料/05.实验原始数据代码及说明/单片机运动控制/STM32/h750_baremetal/Core/Src/fdcan.c)

| 参数 | FDCAN1 | FDCAN2 |
|------|--------|--------|
| 模式 | Normal (Classic CAN) | Normal (Classic CAN) |
| 引脚 | PA11(RX), PA12(TX) | PB12(RX), PB13(TX) |
| AF | GPIO_AF9_FDCAN1 | GPIO_AF9_FDCAN2 |
| Nominal Prescaler | 10 | 10 |
| Time Seg1 | 8 | 8 |
| Time Seg2 | 1 | 1 |
| Sync Jump Width | 1 | 1 |
| TxFifo Queue | 10 elements | 1 element |
| 自动重传 | DISABLE | DISABLE |
| 时钟源 | PLL (HSE→PLL→FDCAN) | PLL |
