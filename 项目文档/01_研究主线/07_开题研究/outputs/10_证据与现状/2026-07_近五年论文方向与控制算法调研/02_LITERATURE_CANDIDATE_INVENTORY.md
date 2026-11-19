# 文献候选清单

> 口径：2021-01-01—2026-07-15；按 DOI 去重。`FULLTEXT` 表示本轮可访问正文或出版页给出足够正文段落，`ABSTRACT_ONLY` 表示只使用摘要/元数据。候选清单不等于核心证据账本。

## A. 建模、辨识、柔性与位置相关动力学

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C01 | 2022 | Huang et al., *A survey of modeling and control in ball screw feed-drive system* | 10.1007/s00170-022-09506-4 | ABSTRACT_ONLY | 综述/主题骨架 |
| C02 | 2022 | Huang et al., 滚珠丝杠传动系统线性变参数动力学模型试验建模方法 | 10.3901/JME.2022.09.098 | FULLTEXT | 位置/负载 LPV 试验建模 |
| C03 | 2023 | Zhang et al., *Gain scheduling control ... based on LPV model* | 10.1007/s00170-022-10205-3 | FULLTEXT | LPV+Stribeck+柔性实验 |
| C04 | 2023 | *Gain-scheduling robust control ... uncertain load mass and varying resonant modes* | 10.1016/j.precisioneng.2022.12.006 | FULLTEXT | 双反馈、位置相关共振 |
| C05 | 2024 | Huang et al., *Polytopic LPV modeling and gain scheduling H-infinity control...* | 10.1016/j.precisioneng.2024.01.004 | FULLTEXT | 闭环频域辨识+LPV控制 |
| C06 | 2024 | Deng et al., *LPV interpolation modeling and modal-based pole placement control...* | 10.1016/j.isatra.2024.07.009 | FULLTEXT | 局部闭环子空间辨识 |
| C07 | 2023 | Li et al., *Vibration suppression ... based on flexible dynamics model* | 10.1016/j.engappai.2022.105506 | FULLTEXT | 电机角单测量辨识+ZVD |
| C08 | 2024 | Xie et al., *Dynamic vibration modeling ... considering travel deviation* | 10.1016/j.precisioneng.2024.03.008 | ABSTRACT_ONLY | 行程偏差与振动 |
| C09 | 2023 | Zhang et al., *Variable-Coefficient Dynamic Modeling ... No-Extra-Load Running State* | 10.3390/pr11030793 | FULLTEXT | 位置/速度/加速度变系数 |
| C10 | 2021 | Wu et al., *Hybrid model for modal properties of a ball screw vibration system* | 10.1007/s12206-021-0104-4 | ABSTRACT_ONLY | 机理-数据混合模态 |
| C11 | 2023 | Li et al., *A review of static and dynamic analysis of ball screw feed drives...* | 10.1016/j.ijmachtools.2023.104021 | ABSTRACT_ONLY | 机械模型综述 |
| C12 | 2023 | *Influence of nonlinearity of servo system electrical characteristics on motion smoothness...* | 出版页待补 DOI | ABSTRACT_ONLY | 机电非理想与微动平顺性 |
| C13 | 2023 | *Modelization and Identification of Feed Drive Axis ... Two-Mass Model and PSO* | IEEE ICIT 2023 稳定入口待补 | ABSTRACT_ONLY | 两质量辨识 |
| C14 | 2025 | McPherson et al., *In-process identification of feed drive dynamics considering machining forces* | 10.1016/j.precisioneng.2025.04.027 | FULLTEXT | 非侵入闭环 PL-ARX+GP |
| C15 | 2025 | Mehrabi & Ahmadi, *Reconfigurable model clusters for scalable modelling of feed drive dynamics* | 10.1038/s41598-025-90325-4 | FULLTEXT | 内部控制信号+模型选择 |
| C16 | 2025 | *A method for constructing digital twins of CNC machine tools feed systems based on hybrid mechanism-data* | Nature 稳定入口待补 | ABSTRACT_ONLY | 混合数字孪生/在线惯量 |
| C17 | 2025 | *A Fusion Modeling Method for Ball Screw Feed System of Machine Tool* | Springer 章节入口待补 | ABSTRACT_ONLY | 刚体+数据未建模动态 |
| C18 | 2026 | *Dynamic modeling of ball screw feed systems: a review and framework...* | 10.59400/sv4089 | FULLTEXT | 2026综述；期刊层级谨慎 |

## B. 柔性反馈、振动和鲁棒控制

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C19 | 2023 | Zhu et al., *Tracking and Vibration Control with a Parallel Structure Controller...* | 10.3390/act12080330 | FULLTEXT | LQR+增益调度、双侧反馈 |
| C20 | 2024 | Wang & Hsiao, *Multivariable Iterative Learning Control ... Flexible Feed Drives* | 10.3390/s24113536 | FULLTEXT | 电机/台面双误差、实物双轴 |
| C21 | 2023 | Zhu et al., *Integral Sliding Mode Control with a Generalized Extended State Observer* | 10.3390/act12100387 | FULLTEXT | 刚体+一阶轴向模态、实验 |
| C22 | 2024 | Wu et al., *Global Nonsingular Fast Terminal SMC ... Improved Double Power Reaching Law* | 10.3390/act13100423 | FULLTEXT | ESO+滑模；主要仿真 |
| C23 | 2025 | Wei et al., *Dynamic sliding mode control ... disturbance observer scheme* | 10.1016/j.jfranklin.2025.107517 | FULLTEXT | 匹配/非匹配扰动、实验 |
| C24 | 2024 | Peng et al., *A novel ball-screw-driven rigid-flexible coupling stage with ADRC...* | 10.1016/j.ymssp.2023.110963 | FULLTEXT | 机构改造+ADRC，不同对象 |
| C25 | 2025 | 滚珠丝杠进给系统改进预设性能无模型自适应滑模控制 | 10.15938/j.emc.2025.09.016 | FULLTEXT | 无模型自适应滑模 |
| C26 | 2025 | *A composite adaptive robust control approach for ball screw feed system with mismatched uncertainty* | JVC 出版页待补 DOI | ABSTRACT_ONLY | 自适应鲁棒控制 |
| C27 | 2026 | *An observer-based memory feedback control ... ball screw drive systems* | 10.1177/09596518261435692 | ABSTRACT_ONLY | ONLINE FIRST；时滞/记忆控制 |
| C28 | 2026 | *Disturbance Observer-Based Fixed-Time SMC for Electromechanical Actuators* | 10.3390/act15050247 | FULLTEXT | 滚珠丝杠 EMA；对象同构但非机床 |

## C. 前馈、学习、轨迹和误差补偿

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C29 | 2021 | Shirvani et al., *Suppression of harmonic positioning errors ... Adaptive Feedforward Cancellation* | 10.1016/j.precisioneng.2020.11.010 | FULLTEXT | 双反馈、时间/空间谐波实物 |
| C30 | 2021 | Dumanli & Sencer, *Data-Driven Iterative Trajectory Shaping ... Flexible Feed Drives* | 10.1109/TMECH.2020.3045444 | FULLTEXT | 参考轨迹整形、实物多轴 |
| C31 | 2021 | Chou et al., *A linear hybrid model for enhanced servo error pre-compensation...* | 10.1016/j.cirp.2021.04.070 | FULLTEXT | 线性机理+数据残差 |
| C32 | 2022 | Sato, *Practical high-precision motion control ... NCTF and simple feedforward* | 10.1016/j.precisioneng.2021.11.017 | FULLTEXT | 少模型、四次学习实验 |
| C33 | 2022 | Hayashi et al., *Negative quadrant glitch suppression ... friction and initial value compensation* | 10.1002/eej.23402 | ABSTRACT_ONLY | 换向/反向误差实物 |
| C34 | 2023 | 雷美荣等，不同指令轨迹条件下滚珠丝杠进给系统运动精度特性研究 | 10.19287/j.mtmt.1005-2402.2023.01.017 | FULLTEXT | 仅仿真；位置/速度/加速度 |
| C35 | 2024 | *A hybrid model for pre-compensating servo error ... high-bandwidth controller* | CIRP JMST 52 出版页待补 DOI | ABSTRACT_ONLY | 混合误差预测 |
| C36 | 2024 | *Feed Error Prediction and Compensation ... WPSO-BPNN* | 10.3390/electronics13050892 | FULLTEXT | 多轴进给误差数据模型 |
| C37 | 2025 | Liu et al., *Dynamic Error Compensation ... Based on Prediction Model* | 10.3390/machines13050433 | FULLTEXT | 机电模型仿真样本+CFNN |
| C38 | 2025 | Fu, *Frequency-domain-based nonlinear normalized ILC for 3-D ball screw drives* | 10.1016/j.isatra.2024.12.030 | FULLTEXT | 数据驱动、变轨迹示例 |
| C39 | 2026 | Liu et al., *Jerk-segmented adaptive ILC for ball screw feed systems* | 10.1016/j.precisioneng.2026.01.004 | FULLTEXT | 10 kHz、实物、正式卷期 |
| C40 | 2026 | 李邦林等，滚珠丝杠驱动系统高斯核前馈补偿方法 | 10.19287/j.mtmt.1005-2402.2026.02.023 | FULLTEXT | 参考/输出训练、非因果FIR |
| C41 | 2025 | 滚珠丝杠进给系统弹性变形预测方法 | 10.19287/j.mtmt.1005-2402.2025.08.017 | ABSTRACT_ONLY | Bouc-Wen混合摩擦/轨迹因素 |
| C42 | 2021 | Hsieh et al., *Iterative learning control of an inverse novel ball screw transmission system* | 10.3233/JIFS-189627 | ABSTRACT_ONLY | 特殊逆丝杠对象 |

## D. 标定、热、健康和误差源

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C43 | 2024 | Wu et al., *Improved displacement sensor accuracy calibration ... ball screw positioning error compensation* | 10.1109/JSEN.2024.3420427 | ABSTRACT_ONLY | 外部位移传感器标定 |
| C44 | 2025 | NIST, *A Novel Methodology for Incipient Ball Screw Backlash Measurement...* | 10.1109/TIM.2025.3545168 | ABSTRACT_ONLY | 回程测量，不是控制 |
| C45 | 2022 | *Ball Screw Health Monitoring with Inertial Sensors* | 10.1109/TII.2022.3210999 | ABSTRACT_ONLY | 状态监测 |
| C46 | 2022 | *Condition monitoring of ball screw feed drives using CNN* | 10.1016/j.cirp.2022.03.017 | ABSTRACT_ONLY | 健康状态/数据驱动 |
| C47 | 2024 | *Robust feature design for early detection of ball screw degradation* | 10.1016/j.mfglet.2024.09.148 | ABSTRACT_ONLY | NIST退化检测 |
| C48 | 2021 | *Research on Thermal Characteristics ... Considering Nut Movement* | 10.3390/machines9110249 | FULLTEXT | 移动螺母热特性 |
| C49 | 2023 | *Temperature field modeling and vibratory-thermal coupling analysis...* | 10.1016/j.tsep.2023.102105 | ABSTRACT_ONLY | 热-振耦合 |
| C50 | 2024 | *Thermal error prediction ... mechanism and data hybrid-driven model* | 10.1007/s00170-024-13841-z | ABSTRACT_ONLY | 全时序热误差 |
| C51 | 2024 | *XGBoost-based thermal error prediction and compensation of ball screws* | 10.1177/09544054231157110 | FULLTEXT | 数据热误差补偿 |
| C52 | 2026 | *Robust physics-informed thermal error modeling ... in-process data assimilation* | ScienceDirect PII S0888327026005856；可见页未给 DOI | ABSTRACT_ONLY | FORMAL；MSSP 255, 114428, 2026-07-01 |

## E. 通信、多轴与系统层

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C53 | 2024 | Zhang et al., *Design and Real-Time Optimization of an EtherCAT Master for Multi-Axis Motion Control* | 10.3390/electronics13153101 | FULLTEXT | 实时主站，不是控制算法 |
| C54 | 2022 | 基于 EtherCAT 的多轴系统同步控制 | 10.3969/j.issn.1005-2895.2022.03.011 | ABSTRACT_ONLY | 同步总线实现 |
| C55 | 2025 | *Error propagation mechanism and robust synchronization control of industrial dual ball screw...* | 10.1088/1361-6501/ae1fa9 | ABSTRACT_ONLY | 双丝杠同步与切削载荷 |
| C56 | 2025 | *Recent Advances in CNC Technology: Toward Autonomous and Sustainable Manufacturing* | 10.1007/s12541-025-01334-2 | ABSTRACT_ONLY | CNC系统综述 |

## F. 物理机制与边界候选

| ID | 年 | 文献 | DOI/稳定入口 | 可见性 | 初筛角色 |
|---|---:|---|---|---|---|
| C57 | 2022 | *Dynamic analysis and stability prediction of nonlinear feed system coupled with flexible workpiece* | 10.1016/j.jsv.2021.116597 | ABSTRACT_ONLY | 柔性工件耦合 |
| C58 | 2024 | *Modeling ... ball-screw feed drives with bolt looseness* | 10.1016/j.ijnonlinmec.2024.104996 | ABSTRACT_ONLY | 松动非线性 |
| C59 | 2025 | *Reliability-based design optimization of key components...* | 10.1016/j.mechmachtheory.2025.106151 | ABSTRACT_ONLY | 结构可靠性，外围 |
| C60 | 2021 | *Developing ball screw drive system of high speed machine tool considering dynamics* | 10.1109/TIE.2021.3083200 | ABSTRACT_ONLY | 动力学约束机械设计 |
| C61 | 2024 | *Dynamic vibration modeling ... travel deviation* | 10.1016/j.precisioneng.2024.03.008 | ABSTRACT_ONLY | 运行振动/偏差 |
| C62 | 2026 | *Contact-Induced vibration ... moving nut FEM and ball recirculation* | 10.1016/j.ymssp.2025.113610 | ABSTRACT_ONLY | ONLINE FIRST/2026，接触振动 |
| C63 | 2022 | *Rolling resistance model for estimation of friction fluctuation in linear ball guideways* | 10.1016/j.precisioneng.2022.01.008 | ABSTRACT_ONLY | 导轨摩擦波动 |
| C64 | 2024 | *Full-stroke Positioning Accuracy Prediction and Compensation of Ball-screw Pair Precision Motion Platform* | 中文期刊稳定入口待补 | ABSTRACT_ONLY | 多尺度行程定位误差 |
| C65 | 2025 | Liu et al., *Modified Data-Driven Disturbance Observer ... Quadrant Glitch Compensation* | 10.1109/TIE.2024.3406853 | ABSTRACT_ONLY | 数据驱动3-DOF+DOB，换向实物 |

## 去重与排除记录

- 预印本已有正式期刊版者只保留正式版：例如刚柔耦合 ADRC 论文。
- 2020 年以前的经典高带宽、双反馈、扰动响应优化论文只进入引用链背景，不计入 65 篇。
- 纯制造、寿命、润滑、点云、光谱共焦和与运动控制无关的热分析已排除。
- C12、C13、C16、C17、C26、C35、C52、C64 的 DOI/正式入口尚需定向核验；未进入“核心完全核验数量”。

## 第一轮覆盖判断

65 篇覆盖了 11 个规定主题簇。第二轮已补到低速换向的数据驱动 DOB 实物论文，但“商业驱动器仅外部位置输入输出的闭环辨识”和“50—200 Hz 非同步监测下的可辨识性”仍没有比 K09/K10 更直接且满足同等限制的工作。

## 2026 候选出版状态复核

| ID | 状态 | 裁决 |
|---|---|---|
| C18 | FORMAL | 已有 2026 卷期与 DOI；只作补充综述 |
| C27 | ONLINE FIRST | DOI 已核验；未进入核心结论 |
| C28 | FORMAL | Actuators 15(5), 247；对象为 EMA，不是当前机床台 |
| C39 | FORMAL | Precision Engineering 99, 57–77, 2026-05 |
| C40 | FORMAL | 《制造技术与机床》2026(2)，DOI 已核验 |
| C52 | FORMAL | MSSP 255, 114428, 2026-07-01；出版页当前可见元数据未显示 DOI，故记 N/A 而不猜测 |
| C62 | ONLINE FIRST / 2026卷期候选 | DOI 已核验；只作物理机制外围候选 |
