# 核心论文证据账本

## 1. 口径与结论先行

- 核心论文：27 篇；均已核验题名、年份、出版源和 DOI。
- 出版状态：27 篇均为 `FORMAL`；其中 2026 年两篇已有正式卷期页码，不按预印本计。
- 可见性：`FULLTEXT` 20 篇，`ABSTRACT_ONLY` 7 篇。后者只填写题名、摘要或出版页能证明的字段。
- 验证分层：21 篇明确含实验或实物对比；4 篇以仿真/数值示例或方法论为主要证据；2 篇为综述。
- 对当前平台最关键的否定性发现：几乎所有主动控制论文都拥有当前位置链不具备的控制写权限、高频同步数据、双编码器或内部控制器信号。

## 2. 逐篇证据

### K01 Huang et al. (2022), Survey

- 文献：*A survey of modeling and control in ball screw feed-drive system*, IJAMT 121, 2923–2946. DOI: [10.1007/s00170-022-09506-4](https://doi.org/10.1007/s00170-022-09506-4)。
- 状态：FORMAL；ABSTRACT_ONLY；综述。
- 可确认：问题簇包括辨识、LPV、热误差、非线性、鲁棒、学习与数据驱动控制；不能从摘要反推各引用实验细节。
- 用途：证明算法簇已成熟；不作为任何性能数字或当前可实施性证据。

### K02 Huang et al. (2022), LPV experimental modeling

- 文献：滚珠丝杠传动系统线性变参数动力学模型试验建模方法，《机械工程学报》58(9). DOI: [10.3901/JME.2022.09.098](https://doi.org/10.3901/JME.2022.09.098)。
- 状态：FORMAL；FULLTEXT；实验建模。
- 对象/方法：把位置、负载等工况变化映射到局部动态参数，再组织为 LPV；依赖专门激励、频响/辨识数据。
- 当前适用性：`CONCEPT_ONLY`。当前 SDO 只读轮询不足以形成同等级闭环频域数据。

### K03 Zhang et al. (2023), LPV gain scheduling

- 文献：*Gain scheduling control of ball screw feed drives based on linear parameter varying model*, IJAMT 124, 4493–4510. DOI: [10.1007/s00170-022-10205-3](https://doi.org/10.1007/s00170-022-10205-3)。
- 状态：FORMAL；FULLTEXT；实验。
- 模型：刚体传递函数、Stribeck 摩擦、弹性体传递函数均由试验辨识；位置相关 LPV。
- 控制/验证：输出反馈增益调度与 PID 对比，目标是在全行程保持一致精度。
- 前提：可激励、可辨识、可部署外环控制器；当前均不满足，不能直接复现。

### K04 Precision Engineering (2023), GS robust control

- 文献：*Gain-scheduling robust control with guaranteed stability for ball screw drives with uncertain load mass and varying resonant modes*, 80, 198–207. DOI: [10.1016/j.precisioneng.2022.12.006](https://doi.org/10.1016/j.precisioneng.2022.12.006)。
- 状态：FORMAL；FULLTEXT；实验。
- 输入/输出：控制电压 (u)；输出为台面直线位置 (l) 与丝杠转角 θ；旋转/直线编码器双反馈。
- 模型：刚体二阶环节乘第一轴向柔性模态；台面位置改变共振，负载质量作为不确定量。
- 当前差异：DS-BVM 只接受现有 GUI 位置目标且内部环黑箱；无控制电压入口、同步双反馈和在线调度权限。

### K05 Huang et al. (2024), polytopic LPV H-infinity

- 文献：*Polytopic LPV modeling and gain scheduling H-infinity control of ball screw with position- and load-dependent variable dynamics*, Precision Engineering 87, 1–10. DOI: [10.1016/j.precisioneng.2024.01.004](https://doi.org/10.1016/j.precisioneng.2024.01.004)。
- 状态：FORMAL；FULLTEXT；辨识与轨迹实验。
- 方法：闭环频域辨识 + LM，位置和负载为调度变量，3 个仿射参数、8 个顶点；混合灵敏度 GS (H_∞)。
- 当前适用性：现象假设“位置/负载依赖”可检验；控制器和参数不能迁移。当前不能获取相同频域数据和外环状态反馈。

### K06 Deng et al. (2024), LPV interpolation + modal pole placement

- 文献：*LPV interpolation modeling and modal-based pole placement control for ball screw drive with dynamic variations*, ISA Transactions 152, 477–486. DOI: [10.1016/j.isatra.2024.07.009](https://doi.org/10.1016/j.isatra.2024.07.009)。
- 状态：FORMAL；FULLTEXT；实验。
- 方法：位置/负载为调度变量，局部闭环子空间辨识后插值；相似变换得到模态状态，LPV 状态观测器与积分误差状态用于反馈。
- 前提：局部高质量闭环试验、可写状态反馈控制器、可测调度变量；当前只满足位置可知这一小部分。

### K07 Li et al. (2023), flexible model + ZVD

- 文献：*Vibration suppression of ball-screw drive system based on flexible dynamics model*, Engineering Applications of Artificial Intelligence 117, 105506. DOI: [10.1016/j.engappai.2022.105506](https://doi.org/10.1016/j.engappai.2022.105506)。
- 状态：FORMAL；FULLTEXT；辨识与实物振动实验。
- 方法：Lagrange 柔性模型化成只含电机转角的参数辨识式；RLS-PSO；耦合 ZVD 输入整形抑制高速停机残振。
- 边界：针对高速停机共振；并非低速小行程的首要证据。当前无高频残振测量和命令整形接口。

### K08 Zhang et al. (2023), variable-coefficient physical model

- 文献：*Variable-Coefficient Dynamic Modeling Method for a Ball Screw Feed System in the No-Extra-Load Running State*, Processes 11, 793. DOI: [10.3390/pr11030793](https://doi.org/10.3390/pr11030793)。
- 状态：FORMAL；FULLTEXT；实验模态校核。
- 模型：Timoshenko 丝杠单元、轴承/螺母结合部弹簧、移动台质量；质量矩阵随螺母位置变，刚度还随速度/加速度及接触状态变。
- 删除项：只研究固有频率时明确忽略阻尼；因此不能直接成为跟踪误差时域对象。
- 当前用途：解释“工况改变对象”的物理来源；参数量远超当前可辨识能力。

### K09 McPherson et al. (2025), in-process closed-loop identification

- 文献：*In-process identification of feed drive dynamics considering machining forces*, Precision Engineering 95, 468–483. DOI: [10.1016/j.precisioneng.2025.04.027](https://doi.org/10.1016/j.precisioneng.2025.04.027)。
- 状态：FORMAL；FULLTEXT；两台 CNC 铣削实验。
- 方法：PL-ARX 的线性部分表示伺服+刚体动态，核回归表示未知切削力；使用内部控制器信号，非侵入、在线预测。
- 最近性：与“商业闭环不拆内部环”最接近，但它拥有高频内部控制器信号和速度数据；当前 SDO 外部轮询不具备。

### K10 Mehrabi & Ahmadi (2025), reconfigurable model clusters

- 文献：*Reconfigurable model clusters for scalable modelling of feed drive dynamics*, Scientific Reports 15, 23988. DOI: [10.1038/s41598-025-90325-4](https://doi.org/10.1038/s41598-025-90325-4)。
- 状态：FORMAL；FULLTEXT；数值案例，不是实物控制验证。
- 方法：电气、机械、控制组件形成候选模型簇；以内部 CNC/控制器信号做贝叶斯模型选择；机械子模型含刚体、一模态、两模态层级。
- 重要限制：模型库需预标定且专家仍决定结构；当前无内部信号，不能照搬，但“先做可证伪模型阶次选择”可继承。

### K11 Zhu et al. (2023), parallel LQR + gain scheduling

- 文献：*Tracking and Vibration Control with a Parallel Structure Controller Based on a Flexible Ball Screw Drive System*, Actuators 12, 330. DOI: [10.3390/act12080330](https://doi.org/10.3390/act12080330)。
- 状态：FORMAL；FULLTEXT；实验。
- 结构：LQR 负责跟踪，插值增益调度控制器抑制振动；台面/电机两侧位置反馈，频响试验覆盖柔性模态。
- 当前边界：LQR 不是可直接套用的候选；其状态、双反馈、模型和控制写通道当前均缺失。

### K12 Wang & Hsiao (2024), multivariable ILC

- 文献：*Multivariable Iterative Learning Control Design for Precision Control of Flexible Feed Drives*, Sensors 24, 3536. DOI: [10.3390/s24113536](https://doi.org/10.3390/s24113536)。
- 状态：FORMAL；FULLTEXT；工业双轴 CNC 实验。
- 方法：同时利用电机侧和台面侧误差，经范数优化向参考轨迹和控制命令两个通道注入补偿。
- 当前边界：需要重复任务、同步双侧误差和两个可写通道；当前最多只具备“可能重复相对位置任务”这一条件。

### K13 Zhu et al. (2023), GESO-ISMC

- 文献：*The Design of a Tracking Controller for Flexible Ball Screw Feed System Based on Integral Sliding Mode Control with a Generalized Extended State Observer*, Actuators 12, 387. DOI: [10.3390/act12100387](https://doi.org/10.3390/act12100387)。
- 状态：FORMAL；FULLTEXT；仿真与对比实验。
- 模型：刚体加第一轴向模态；观测匹配/非匹配扰动和不可测状态；外环滑模加振动补偿。
- 当前边界：依赖已知状态空间结构、高率状态估计和控制输入权限；50/200 Hz 非同步观测不足。

### K14 Wu et al. (2024), GNFTSMC + ESO

- 文献：*Research on Global Nonsingular Fast Terminal Sliding Mode Control Strategy ...*, Actuators 13, 423. DOI: [10.3390/act13100423](https://doi.org/10.3390/act13100423)。
- 状态：FORMAL；FULLTEXT；公开正文展示的主要性能证据为仿真。
- 注意：论文报告约 14 μm 量级的平均位移误差改进，但不能据此推出精密实物台性能；比较依赖给定扰动和模型。
- 当前用途：只作为算法前提/反例，不列为近期首选方向。

### K15 Wei et al. (2025), dynamic SMC + disturbance observer

- 文献：*Dynamic sliding mode control for ball screw drive systems under a disturbance observer scheme*, Journal of the Franklin Institute 362, 107517. DOI: [10.1016/j.jfranklin.2025.107517](https://doi.org/10.1016/j.jfranklin.2025.107517)。
- 状态：FORMAL；FULLTEXT；实验。
- 输入/状态：电机侧电压/等效转矩通道，线性与旋转轨迹误差；统一状态/扰动观测器处理匹配与非匹配扰动。
- 当前边界：控制接口和状态数据均不匹配；其“不可把所有误差揉成一个匹配扰动”的提醒可继承。

### K16 Peng et al. (2024), rigid-flexible stage + ADRC

- 文献：*A novel ball-screw-driven rigid-flexible coupling stage with active disturbance rejection control to compensate for friction dead zone*, MSSP 208, 110963. DOI: [10.1016/j.ymssp.2023.110963](https://doi.org/10.1016/j.ymssp.2023.110963)。
- 状态：FORMAL；FULLTEXT；实物。
- 对象：通过柔性铰链改变机构，使摩擦死区转化为弹性变形；ADRC 在 DS1202 上实现位置/速度环，驱动器内电流 PI。
- 裁决：因机械结构和控制权限都被改变，不能作为当前既有商业单轴平台的直接路线。

### K17 Shirvani et al. (2021), adaptive feedforward cancellation

- 文献：*Suppression of harmonic positioning errors in ball-screw drives using Adaptive Feedforward Cancellation*, Precision Engineering 68, 235–255. DOI: [10.1016/j.precisioneng.2020.11.010](https://doi.org/10.1016/j.precisioneng.2020.11.010)。
- 状态：FORMAL；FULLTEXT；机床实验。
- 输入/反馈：控制量是电机电流命令；旋转与直线位置双反馈。针对时间周期切削力和空间周期导程/装调误差。
- 结果：摘要可核验的改善为切削力工况误差标准差 50%–78%，位置周期误差通常 10%–20%，且恒速段最有效。
- 当前用途：可用于提出“空间周期误差”假设，但当前无电流命令写权和同步双反馈。

### K18 Dumanli & Sencer (2021), data-driven trajectory shaping

- 文献：*Data-Driven Iterative Trajectory Shaping for Precision Control of Flexible Feed Drives*, IEEE/ASME T-Mech 26, 2735–2746. DOI: [10.1109/TMECH.2020.3045444](https://doi.org/10.1109/TMECH.2020.3045444)。
- 状态：FORMAL；FULLTEXT；多轴进给实物。
- 方法：迭代修改参考位置、速度和加速度轮廓，补偿闭环伺服动态、结构振动和预滑摩擦；不等于无条件模型自由。
- 当前适配：概念上接近位置目标入口；实现仍要求精确重复、可编程轨迹、周期同步高率误差和安全迭代。

### K19 Chou et al. (2021), linear hybrid servo-error model

- 文献：*A linear hybrid model for enhanced servo error pre-compensation of feed drives with unmodeled nonlinear dynamics*, CIRP Annals 70, 301–304. DOI: [10.1016/j.cirp.2021.04.070](https://doi.org/10.1016/j.cirp.2021.04.070)。
- 状态：FORMAL；FULLTEXT；实验。
- 方法：物理线性模型预测主要伺服误差，数据驱动线性残差模型吸收未建模非线性，用于预补偿。
- 当前适配：研究哲学高度相符；但补偿部署与高率训练信号尚不可用。当前可先做“只诊断、不补偿”的外部等效版本。

### K20 Sato (2022), NCTF + learned FF

- 文献：*Practical high-precision motion control system based on nominal characteristic trajectory following control and simple feedforward element design methods*, Precision Engineering 75. DOI: [10.1016/j.precisioneng.2021.11.017](https://doi.org/10.1016/j.precisioneng.2021.11.017)。
- 状态：FORMAL；FULLTEXT；滚珠丝杠实物。
- 方法：NCTF 反馈不依赖精确机械模型；四次实验确定时不变前馈。试验台有高分辨率线性编码器和亚毫秒级采样。
- 当前边界：证明“少模型+少量实验”可行，不证明 50/200 Hz 外部 SDO 数据足够，也不赋予当前控制器写权。

### K21 Hayashi et al. (2022), quadrant glitch

- 文献：*Negative quadrant glitch suppression control ... by friction compensation and initial value compensation*, Electrical Engineering in Japan 215(3). DOI: [10.1002/eej.23402](https://doi.org/10.1002/eej.23402)。
- 状态：FORMAL；ABSTRACT_ONLY；摘要明确仿真与双轴实验。
- 可确认：滚动摩擦导致换向 quadrant glitch；单纯模型摩擦补偿可能产生反方向误差，加入初值补偿。
- 不可确认：采样、控制输入细节和全部量化结果；当前只能用于建立换向假设。

### K22 雷美荣等 (2023), command-trajectory conditions

- 文献：不同指令轨迹条件下滚珠丝杠进给系统运动精度特性研究，《制造技术与机床》(1), 109–113. DOI: [10.19287/j.mtmt.1005-2402.2023.01.017](https://doi.org/10.19287/j.mtmt.1005-2402.2023.01.017)。
- 状态：FORMAL；FULLTEXT；仅 Simulink/Simscape 半物理机电耦合仿真。
- 可确认：位置、速度、加速度等指令参数影响仿真运动精度。
- 裁决：可支持工况分层设计，但不能写成实物规律或当前平台证据。

### K23 Liu et al. (2025), prediction-model dynamic-error compensation

- 文献：*Dynamic Error Compensation for Ball Screw Feed Drive Systems Based on Prediction Model*, Machines 13, 433. DOI: [10.3390/machines13050433](https://doi.org/10.3390/machines13050433)。
- 状态：FORMAL；FULLTEXT；仿真与实验。
- 方法：位置相关机电模型经拉丁超立方仿真生成单控制周期样本，训练级联前馈神经网络，再做前馈补偿。
- 边界：训练数据主要源于模型而非当前平台采集；若模型不可信会继承偏差。需要可写前馈通道。

### K24 Fu (2025), frequency-domain normalized ILC

- 文献：*Frequency-domain-based nonlinear normalized iterative learning control for three-dimensional ball screw drive systems*, ISA Transactions 157, 224–232. DOI: [10.1016/j.isatra.2024.12.030](https://doi.org/10.1016/j.isatra.2024.12.030)。
- 状态：FORMAL；FULLTEXT；正文称“两例”验证，未从可见证据确认真实硬件，故不计硬件验证。
- 方法：输入输出比的频域归一化，面向变迭代轨迹；声称无需名义模型。
- 当前边界：仍需离散、长度一致、高率输入输出序列和可迭代更新控制输入；不能由非同步 SDO 轮询直接支持。

### K25 Liu et al. (2026), jerk-segmented adaptive ILC

- 文献：*Jerk-segmented adaptive iterative learning control for ball screw feed systems*, Precision Engineering 99, 57–77. DOI: [10.1016/j.precisioneng.2026.01.004](https://doi.org/10.1016/j.precisioneng.2026.01.004)。
- 状态：FORMAL（2026-05 卷期）；FULLTEXT；仿真与实验。
- 方法：按 jerk 分段，零相位 Butterworth 提取低频误差趋势，非线性 PID 型学习律，误差能量自适应参数；10 kHz。
- 结果：摘要报告相对基线 RMSE 最多降 57.40%、峰值误差最多降 38.19%。
- 当前边界：10 kHz 与当前 50/200 Hz 相差两个数量级以上；且需速度命令和迭代写入，不可直接迁移。

### K26 李邦林等 (2026), Gaussian-kernel feedforward

- 文献：滚珠丝杠驱动系统高斯核前馈补偿方法，《制造技术与机床》(2). DOI: [10.19287/j.mtmt.1005-2402.2026.02.023](https://doi.org/10.19287/j.mtmt.1005-2402.2026.02.023)。
- 状态：FORMAL；FULLTEXT；实验。
- 方法：参考轨迹与实际输出构成训练数据，滑动窗离线重构，高斯回归得到非因果 FIR 逆模型前馈。
- 结果：最大跟踪误差 9.4 μm 降至 5.9 μm；不同轨迹最大误差下降超过 20%。
- 最近性：数据接口最接近当前位置目标/实际位置，但最终仍要写入前馈补偿且要求时序可靠、训练域覆盖和稳定性审查。

### K27 Liu et al. (2025), modified data-driven DOB for quadrant glitch

- 文献：*Modified Data-Driven Disturbance Observer With Its Application in Quadrant Glitch Compensation of Ball Screw Feed-Drive System*, IEEE Transactions on Industrial Electronics 72(1), 167–176. DOI: [10.1109/TIE.2024.3406853](https://doi.org/10.1109/TIE.2024.3406853)。
- 状态：FORMAL；ABSTRACT_ONLY；摘要明确实验对比。
- 方法：数据驱动三自由度控制结构加改进 DOB，针对带迟滞、难完整建模的 quadrant glitch；与反馈、传统 2-DOF 和传统 DOB 对比。
- 边界：可确认优于所列基线，但摘要不足以核验采样、信号权限和量化结果。其存在使“数据驱动摩擦补偿”本身不再是可信新意。

## 3. 核心集统计

| 维度 | 数量 | 说明 |
|---|---:|---|
| 2021 | 3 | K17–K19 |
| 2022 | 4 | K01、K02、K20、K21 |
| 2023 | 7 | K03、K04、K07、K08、K11、K13、K22 |
| 2024 | 5 | K05、K06、K12、K14、K16 |
| 2025 | 6 | K09、K10、K15、K23、K24、K27 |
| 2026 | 2 | K25、K26 |
| 合计 | 27 | 与逐篇账本一致 |

精确年份计数为：2021=3，2022=4，2023=7，2024=5，2025=6，2026=2，共 27。

## 4. 证据强度裁决

1. **强证据**：K04–K07、K09、K12、K17–K20、K25–K26，因对象、输入/反馈和实验边界较清楚。
2. **中证据**：K02–K03、K08、K10–K11、K13、K15–K16、K23，方法清楚但与当前接口存在本质差异或验证层级有限。
3. **仅问题定义/反例**：K01、K14、K21、K22、K24；不能据此直接立控制路线。

## 5. 不能从核心集推出的结论

- 不能推出 DS-BVM 可被外部 LPV/LQR/SMC/DOB 直接接管。
- 不能推出 50/200 Hz SDO 轮询能识别机械共振、内环带宽或 1 ms 动态。
- 不能把“数据驱动”理解为不需要同步、重复性、激励、输入权限或独立验证。
- 不能把其他滚珠丝杠台上的 μm 级改善数字迁移为当前平台预期。
