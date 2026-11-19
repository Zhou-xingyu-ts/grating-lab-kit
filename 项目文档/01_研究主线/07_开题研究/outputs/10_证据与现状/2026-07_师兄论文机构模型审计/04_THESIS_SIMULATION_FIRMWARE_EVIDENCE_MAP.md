# 论文—仿真—源码—固件—实物证据分层图

## 0. 状态定义

- YES：该层有直接静态证据；
- PARTIAL：只有部分结构或同类能力，尚未闭合到该项；
- NO：已看到的实现与该项明确不相符；
- NOT_FOUND：在本次有边界的相关目录核查中未找到；
- N/A：该层不适用。

六个证据标签的含义：

- THESIS_FORMULATION：论文/PPT提出；
- SIMULATION_IMPLEMENTED：仿真模型实际实现；
- SOURCE_CODE_STATIC：源码中有可定位的算法/调用结构；
- FIRMWARE_IMPLEMENTED：已成为目标 MCU 的可调用固件功能；
- HARDWARE_VALIDATED：有与固件、参数、总线和实物对应的运行证据；
- CURRENT_PLATFORM_APPLICABLE：在当前权限、接口和可观测性下可用。

## 1. 总表

| 项目 | THESIS_FORMULATION | SIMULATION_IMPLEMENTED | SOURCE_CODE_STATIC | FIRMWARE_IMPLEMENTED | HARDWARE_VALIDATED | CURRENT_PLATFORM_APPLICABLE |
|---|---|---|---|---|---|---|
| 双质量模型 | YES：图 2.15/PPT | NO：mechanical_system.slx 为单二阶块 | PARTIAL：只有图稿，未见双质量求解代码 | NOT_FOUND | NOT_FOUND | PARTIAL：仅作柔性/共振概念和后续辨识候选 |
| 二阶简化模型 | YES：式 2.10—2.14 | YES：分母 [M B K] | YES：Simulink 参数、LQR.m、sys.m | NOT_FOUND | NOT_FOUND | PARTIAL：模型结构可参考，数值不能继承；当前优先闭环黑箱 |
| 库仑/黏性摩擦 | YES：式 2.9 | YES：Simulink Coulomb & Viscous Friction | YES：模型参数 \(F_c,F_v\) | NOT_FOUND | NOT_FOUND | PARTIAL：物理概念适用，形式和参数需重辨识 |
| PID | YES：三环叙述和图 2.18/2.19 | YES：离散 PID.slx | YES：模型块参数可读 | NOT_FOUND：未见 MCU PID 调用 | NOT_FOUND | NO：当前不能绕过 DS-BVM 内部位置/速度/电流环部署论文 PID |
| 前馈 | YES：式 2.15、图 2.20 | PARTIAL：sys.m 实现恒定目标平衡前馈 | YES：F=[k/m,(Fv+b)/m] | NOT_FOUND | NOT_FOUND | PARTIAL：仅概念；当前内置轨迹/前馈边界不同 |
| LQR | YES：式 2.17、图 2.20/2.21 | YES：LQR.m + Tracking.m/sys.m | YES：Q/R 和增益存在 | NOT_FOUND | NOT_FOUND | NO：当前无状态/力接口和写入权限 |
| 1 ms 离散控制 | YES：论文/PPT主张 | PARTIAL：PID 为 1 ms，LQR 是连续 ode45 | PARTIAL：不同 MCU 工程有不一致定时器痕迹 | NOT_FOUND：未闭合到实际控制任务 | NOT_FOUND | NO：当前 SDO 观察约 50/200 Hz，且非同步 |
| CANopen PDO 控制 | YES：PPT/论文结构 | NO：机械控制仿真不含总线 | PARTIAL：有协议栈/PDO结构，但无 DS-BVM CiA 402 映射 | NOT_FOUND：未见有效主站状态机/SYNC生产者 | NOT_FOUND | NO：当前只开放 SDO upload |
| 三轴控制 | YES：三轴运动台/系统主张 | NO：相关控制仿真是单轴 | NOT_FOUND：未见三轴调度与同步控制算法 | NOT_FOUND | NOT_FOUND：没有控制链匹配证据 | NO：当前目标是现有 Z 轴单轴最小闭环 |

## 2. 双质量模型

### THESIS_FORMULATION

- 图 2.14 给出电机—联轴器—丝杠—工作台—光栅的机械链；
- 图 2.15 标出 \(m_1,m_2,b_1,b_2,c,k,x_1,x_2,\tau,d_1,d_2\)；
- PPT称其为“直线—直线等效双质量模型”，并说忽略丝杠转动惯量。

### SIMULATION_IMPLEMENTED / SOURCE_CODE_STATIC

mechanical_system.slx 只有单个二阶 Transfer Fcn，未见双质量状态、两个位移或四阶分母。因此：

- SIMULATION_IMPLEMENTED = NO；
- SOURCE_CODE_STATIC = PARTIAL 仅指图稿，不指求解实现。

### HARDWARE_VALIDATED

未见模态测试、频响、共振/反共振拟合或两个质量参数，因此不能写“历史双质量模型经实物验证”。

## 3. 二阶模型与摩擦

mechanical_system.slx 静态结构为：

期望力矩（名称） → Driver Gain=1 → 与摩擦相减 → ±100 饱和 → \(1/(Ms^2+Bs+K)\) → 位移。

位移经微分得到速度，进入 Coulomb & Viscous Friction 块，再反馈到净力求和点。

这证明：

- 二阶模型和摩擦确实 SIMULATION_IMPLEMENTED；
- “期望力矩”只是端口名称；没有丝杠折算时，块内部的量纲实际按力处理；
- 仿真实现不等于硬件实现；
- 参数冲突使其也不能称为同一名义对象的完整复现。

## 4. PID、前馈与 LQR

### PID

PID.slx 的静态事实：

- 一自由度 Ideal PID；
- 采样时间 0.001 s；
- Forward Euler 积分/滤波；
- \(K_p=44.2156,K_i=0.289666,K_d=0.647116,N=376.138\)；
- 误差求和为参考减实际；
- 对象包含 \(F_c=10\) 和 ±100 限幅。

所以 PID 是 SIMULATION_IMPLEMENTED，但本次相关固件证据中没有 PID 算法调用。

### 前馈

sys.m 有：

\[
F=[k/m,(F_v+b)/m],\quad
e=z_d-z,\quad
u=Fz_d+K_ee
\]

目标状态的速度分量恒为 0，所以实际只使用了静态刚度平衡项。它是 SOURCE_CODE_STATIC 和连续数值仿真实现，不能升级为硬件前馈。

### LQR

LQR.m 用 \(Q={\rm diag}(100,1),R=1\) 得到反馈增益；Tracking.m 用 ode45 调用 sys.m。该链证明 LQR 的静态仿真实现，但没有：

- MCU 算法；
- 1 ms 离散 LQR；
- 状态估计器；
- 输出/电流限幅；
- CANopen 对象映射；
- 历史硬件日志。

## 5. 1 ms 与 CANopen PDO

论文/PPT称：

- 运动控制周期 1 ms；
- STM32H7；
- CANopen PDO；
- 对比 PLC 713 μs、MCU 333 μs、53.3% 等。

但工作区证据溯源总账对历史 MCU 工程的静态审计显示：

1. 不同 H743/H750 工程的定时器周期并不一致，有约 1 ms、约 0.5 ms、约 0.5 s 等不同变体；
2. CANopen 应用多把 MCU 初始化为节点，而非完整 NMT 主站；
3. SDO 帮助函数存在但未找到有效调用链；
4. 通信周期对象为 0，未见活动 SYNC 生产者；
5. RPDO 映射计数为 0，TPDO 只见本地示例对象；
6. 未见 DS-BVM 的 0x6040、0x6041、0x6064、0x607A 等 CiA 402 控制映射；
7. 未见 Servo On/状态机/故障处理/三轴调度；
8. 编译成功日志只能证明某工程产出 ELF，不能证明烧录、总线或运动。

因此：

- THESIS_FORMULATION = YES；
- SOURCE_CODE_STATIC = PARTIAL；
- FIRMWARE_IMPLEMENTED = NOT_FOUND；
- HARDWARE_VALIDATED = NOT_FOUND。

## 6. 三轴控制

论文第 2.2 节描述三轴运动台，PPT和总结也给出三轴系统主张。但控制仿真目录中的对象、PID 和 LQR 均为单轴；相关 MCU 工程未见三轴状态机、同步调度或三个 DS-BVM 轴的对象映射。

历史平台可能曾有三轴机械结构和其他控制设备，这不等于“本次发现的 STM32/CANopen/PID/LQR 文件已闭合三轴实物控制”。必须把机械存在、控制设想、仿真和运行证据分开。

## 7. 当前平台证据层

截至当前执行文件：

- HARDWARE_VALIDATED：Ri0040 进入 DS-BVM 的单轴内部位置闭环已经可以通过 Servo Studio 做相对位置运动；
- HARDWARE_VALIDATED：CANopen Node-ID=1、1 Mbit/s 的 SDO upload 只读链已建立；
- HARDWARE_VALIDATED：7 对象动态记录约 50 Hz，3 核心对象约 200 Hz；
- LIMITATION：每轮对象顺序读取，共用时间戳不等于同步快照；
- LIMITATION：尚未正式外部位移标定；
- LIMITATION：CANopen SDO/PDO 写入、Servo On 和运动命令没有放行；
- IDENTIFIABILITY：只能先观察外部目标/实际位置等闭环输入输出，不能拆出 DS-BVM 内部三环。

这里的“当前实物已动”和“论文控制器已验证”是两件不同的事。

## 8. 证据文件

历史论文/仿真：

- 90_原始资料/赖建君毕业归档资料/.../thuthesis.pdf
- 90_原始资料/赖建君毕业归档资料/.../正式答辩.pptx
- 历史运动控制仿真目录中的 mechanical_system.slx、PID.slx、test_step_response.slx、Misc.mlx、ref/*.m、python/plot.ipynb

历史固件与全工作区反证：

- 01_研究主线/07_开题研究/outputs/20_路线与方法演进/2026-07_阶段FR_路线反证与定型修订/workspace_evidence_provenance/02_KEY_CLAIM_PROVENANCE_LEDGER.md
- 01_研究主线/07_开题研究/outputs/20_路线与方法演进/2026-07_阶段FR_路线反证与定型修订/workspace_evidence_provenance/WORKSPACE_PROVENANCE_FINAL_HANDOFF.md

当前平台：

- 02_实验与现场/01_当前执行/00_总览.md
- 02_实验与现场/01_当前执行/58_现场实拍确认事实索引.md
- 02_实验与现场/01_当前执行/60_实验室推进记录/2026-07-12_USB控制_CANopen只读_50Hz动态采集.md
- 01_研究主线/07_开题研究/outputs/20_路线与方法演进/2026-07_阶段D1_模型技术校正/
- 01_研究主线/07_开题研究/outputs/2026-07_阶段E_辨识边界与最小数据设计/

