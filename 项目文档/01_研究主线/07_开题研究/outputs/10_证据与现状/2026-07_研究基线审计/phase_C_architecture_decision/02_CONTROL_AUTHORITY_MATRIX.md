# 控制权与信号职责矩阵

## 1. 控制权总表

| 控制/安全职责 | A：DS内部闭环 | B：STM32位置+DS CSV | C：STM32位置/速度+DS CST | D：STM32轨迹+DS CSP |
|---|---|---|---|---|
| 轨迹生成 | DS（PPM）或上位机给低频目标 | STM32 | STM32 | STM32 |
| 位置环 | DS | STM32 | STM32 | DS |
| 速度环 | DS | DS | STM32 | DS |
| 电流/转矩环 | DS | DS | DS | DS |
| 光栅计数 | DS | STM32 | STM32 | DS |
| 速度估计 | DS内部，来源待确认 | DS电机侧反馈；STM32也可估位置导数作外环诊断 | STM32必须可靠估计/观测；DS仍需电角度 | DS内部 |
| 换相/电角度 | DS，来源待确认 | DS独立电机反馈/Hall | DS独立电机反馈/Hall | DS，来源待确认 |
| 位置目标 | Servo Studio/上位机/低频主站 | STM32内部轨迹 | STM32内部轨迹 | STM32周期607A |
| 速度目标 | DS内部生成 | STM32周期60FF | STM32内部速度环 | DS内部位置环生成 |
| 转矩目标 | DS内部级联 | DS内部速度环生成 | STM32周期6071 | DS内部级联 |
| CiA 402状态机 | 若仍用Servo Studio则由现软件；CANopen化后由主站 | STM32主站 | STM32主站 | STM32主站 |
| PDO/SYNC | PPM无需；CSP分支需要 | 必须 | 必须 | 必须 |
| 通信失联处置 | DS/Servo Studio现有机制，仍需核查 | STM32+DS双层 | STM32+DS双层，最严格 | STM32+DS双层 |
| 软件限位/目标限幅 | DS参数+上层 | STM32主责，DS保底 | STM32主责，DS保底 | STM32轨迹限制+DS保底 |
| Quick Stop/STO/制动 | DS硬件安全链 | 不能由外位置环替代 | 不能由外控制器替代 | 不能由轨迹主站替代 |

## 2. 反馈信号职责

| 信号 | A | B | C | D | 仍需证据 |
|---|---|---|---|---|---|
| 负载位置 | DS读取光栅，反映于6064的具体绑定仍需确认 | STM32读取光栅；DS的6064来自其电机侧反馈 | STM32读取光栅；DS的6064为自身反馈 | DS读取光栅 | 自研尺接口、方向/尺度/零点、逐周期时间戳 |
| 电机速度 | DS由所选反馈估算 | DS必须从独立电机侧反馈得到 | STM32需估计控制速度；DS还需保护/换相所需反馈 | DS内部估算 | 当前电机反馈类型、606C来源、滤波与延迟 |
| 电角度/换相 | DS当前配置未知 | DS电机侧反馈/Hall | DS电机侧反馈/Hall | DS当前配置未知 | Hall接线、极对数、初始电角度、传感器类型 |
| Index/基准 | 光栅可选I进入DS | 光栅I进入STM32或由外部回零 | 同B | 光栅I进入DS | 自研尺是否提供I、回零策略 |
| 位置误差 | DS内部；可观测6062/6064/60F4但60F4公式未公开 | STM32显式计算外位置误差；DS内部速度误差不可完全见 | STM32显式计算位置/速度误差 | DS内部位置误差 | 统一单位、采样对齐、对象更新时间 |

## 3. 模式证据与控制权依据

官方控制权依据来自：

- `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\outputs\2026-07_研究基线审计\phase_B1_dsbvm_manual\03_OPERATION_MODE_BOUNDARIES.md`：`MODE-B1-PPM/PVM/CSP/CSV/CST`；
- `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\outputs\2026-07_研究基线审计\phase_B1_dsbvm_manual\01_FEEDBACK_INTERFACE_AUDIT.md`：`FB-B1-009`–`FB-B1-014`；
- `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\outputs\2026-07_研究基线审计\phase_B1_dsbvm_manual\04_PDO_SYNC_AND_CIA402_AUDIT.md`：状态机、映射和NMT前置条件。

当前实现边界来自：

- `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\outputs\2026-07_研究基线审计\phase_A_current_system\01_CURRENT_SYSTEM_FACTS.md`：`FACT-A001`、`FACT-A003`、`FACT-A006`–`FACT-A009`；
- `D:\清华\清华-科研横向\光栅尺运动控制调研\01_研究主线\07_开题研究\outputs\2026-07_研究基线审计\phase_B2_legacy_thesis_firmware\02_FIRMWARE_ARCHITECTURE_AUDIT.md`：`FW-B2-001`–`FW-B2-007`。

## 4. 不能跨越的边界

1. STM32接管位置/速度环，不等于可以绕过DS-BVM的电机换相和硬件安全链。
2. 光栅只进STM32时，DS-BVM的6064不会自动变成光栅位置。
3. 选择CSV/CST/CSP不自动配置PDO、SYNC、NMT Operational或CiA 402状态机。
4. 现有SDO只读采集能力不能转换成周期控制能力。
5. `h750_demo`中的1 ms协议处理函数不能替代外环控制器、帧映射和deadline证据。

