# 有限文献与创新性缺口图

## 1. 检索边界与证据规则

本轮于2026-07-15围绕四个规定主题做有限反查，每主题保留6项，共24项。优先同行评议论文与出版社/机构稳定页面；综述仅用于确认领域成熟度；学位论文未作为核心证据；厂商/标准能力不作为算法效果证据；未使用普通博客。实验数字只记录出版页面或论文可核对内容，无法从可访问摘要确认的写“N/A”，不推测。

此图不是系统综述，不能证明“绝无创新”；它只用于检验阶段F四项贡献是否已被明显先例削弱。

## 2. 负载侧光栅/线性编码器反馈的精密位置控制

| 完整题目；作者；年份；来源；DOI/链接 | 研究对象 | 反馈结构 | 控制结构 | 实验指标 | 与本项目相同点 | 仍可能的差异 | 是否削弱阶段F创新 |
|---|---|---|---|---|---|---|---|
| *Machine Tool Feed Drives*；Yusuf Altintas, Alexander Verl, Christian Brecher, Luis Uriarte, Günter Pritschow；2011；CIRP Annals 60(2), 779–796；[DOI](https://doi.org/10.1016/j.cirp.2011.05.010) | 机床进给驱动综述 | 覆盖电机侧、工作台侧及多传感器反馈 | 进给驱动建模、控制、补偿综述 | 综述，无单一可迁移指标 | 滚珠丝杠、伺服环、位置/动态误差 | 本项目可强调证据等级和特定硬件约束 | **是**：负载侧反馈和进给驱动分层本身不是新概念 |
| *An Investigation Into the Relative Accuracy of Ball-screws and Linear Encoders over a Broad Range of Application Configurations and Usage Conditions*；A. J. White, S. R. Postlethwaite, D. G. Ford；1999；WIT Transactions on Engineering Sciences 23；[DOI](https://doi.org/10.2495/LAMDAMAP990311) | CNC滚珠丝杠轴热误差 | 比较旋转编码器反馈与直线编码器反馈 | 工业轴位置测量对照 | 多组热试验；线性编码器误差通常更小、变化更慢，摘要无统一单值 | 商用直线尺、丝杠、位置误差 | 本项目可做低速小行程、方向与自研尺对照 | **是**：线性尺替代旋转反馈的系统价值已有先例 |
| *Robust Compensation of Elastic Deformations in Ball Screw Drives*；Amin Kamalzadeh, Daniel J. Gordon, Kaan Erkorkmaz；2010；International Journal of Machine Tools and Manufacture 50(6), 559–574；[DOI](https://doi.org/10.1016/j.ijmachtools.2010.03.001) | 工业滚珠丝杠弹性变形 | 旋转编码器用于控制，线性编码器监测真实工作台位置 | 弹性变形模型补偿+鲁棒伺服 | 高速跟踪、加工与仿真实验；摘要无统一单值 | 外部负载侧尺度、等效误差与补偿验证 | 它展示了“不把线性尺闭环”也可形成补偿路线；本项目的F1不同 | **是**：阶段F不能把负载侧反馈→补偿写成唯一方法；也支持比较替代结构 |
| *Optimal High-bandwidth Control of Ball-screw Drives with Acceleration and Jerk Feedback*；Arda Dumanli, Burak Sencer；2018；Precision Engineering 54, 254–268；[DOI](https://doi.org/10.1016/j.precisioneng.2018.06.002) | 柔性滚珠丝杠工业级平台 | 工作台线性编码器+加速度计，非共置负载侧反馈 | LQR极点配置、运动学观测器、加速度/jerk反馈 | 实物带宽、模态阻尼和鲁棒性对照；可访问摘要无统一数值 | 负载侧位置、柔性动态、宽带控制 | 本项目当前低速且驱动内部环不可见，目标可能是证据受限的局部模型 | **是**：负载侧多传感器高带宽控制已有成熟实验 |
| *Tracking and Vibration Control with a Parallel Structure Controller Based on a Flexible Ball Screw Drive System*；Muzhi Zhu, Dafei Bao, Mengxin Sun, Yong Liu；2023；Actuators 12(8), 330；[DOI](https://doi.org/10.3390/act12080330) | 位置/负载变化下的柔性丝杠 | 电机旋转编码器+工作台Heidenhain线性编码器 | LQR跟踪+增益调度μ综合振动控制 | 50 kg、560 mm台架比较实验；线性尺插值分辨率0.05 μm，性能值依图表 | 双侧测量、动态随位置/负载变化、实验验证 | 本项目可聚焦低成本黑箱证据和传感器替换，而非高带宽控制 | **是**：分层反馈和跟踪/振动并行控制不是空白 |
| *Practical High-precision Motion Control System Based on Nominal Characteristic Trajectory Following Control and Simple Feedforward Element Design Methods*；Kaiji Sato；2022；Precision Engineering 75, 55–66；[DOI](https://doi.org/10.1016/j.precisioneng.2021.11.017) | 线性编码器测量的滚珠丝杠机构 | 20 nm线性编码器测工作台，0.2 ms采样 | NCTF反馈+学习得到的时不变前馈 | 0.2 mm、1 Hz正弦平均最大误差0.209 μm | 黑箱/少模型、方向反转误差、前馈、真实台架 | 本项目可比较事件时间语义、驱动内部环和自研尺误差 | **是**：少模型+学习前馈+线性尺精密控制已有实验先例 |

**主题判断：**“采用负载侧光栅”“以线性尺形成闭环”“将单轴测量与动态补偿结合”均不能单独构成创新。可能缺口只能来自本项目特有的黑箱驱动证据受限条件、自研尺特性，或一种被实验证明优于现有流程的可追溯方法。

## 3. 滚珠丝杠标定、回程、摩擦和方向误差补偿

| 完整题目；作者；年份；来源；DOI/链接 | 研究对象 | 反馈结构 | 控制结构 | 实验指标 | 与本项目相同点 | 仍可能的差异 | 是否削弱阶段F创新 |
|---|---|---|---|---|---|---|---|
| *Robust Friction Compensation for Submicrometer Positioning and Tracking for a Ball-screw-driven Slide System*；Paul I. Ro, Wonbo Shim, Sanghwa Jeong；2000；Precision Engineering 24(2), 160–173；[DOI](https://doi.org/10.1016/S0141-6359(00)00030-1) | 单轴滚珠丝杠滑台 | 编码器反馈，摩擦从实验估计 | bristle非线性摩擦估计+PD；跟踪加DOB | 亚微米范围稳态误差低于1.5%，含定位/跟踪实物实验 | 丝杠、低速、摩擦/方向相关现象、补偿验证 | 本项目尚不能把方向项归因摩擦，可突出因果否证 | **强烈是**：摩擦建模补偿不是创新 |
| *Practical Control Method for Ultra-precision Positioning Using a Ballscrew Mechanism*；Guilherme Jorge Maeda, Kaiji Sato；2008；Precision Engineering 32(4), 309–318；[DOI](https://doi.org/10.1016/j.precisioneng.2007.10.002) | 常规滚珠丝杠超精密PTP | 位置反馈，具体线性尺见论文装置 | NCTF+PI+anti-windup，弱模型依赖 | 报告5 nm定位分辨率；对摩擦变化鲁棒 | PTP、非线性摩擦、积分与防饱和 | 本项目更强调标定/证据边界，且性能尚未知 | **是**：低模型依赖精密定位与anti-windup已有成熟实验 |
| *Initial Friction Compensation by Disturbance Observer Based on Rolling Friction Model*；Yoshihiro Maeda, Makoto Iwasaki；2010；IEEJ Transactions on Industry Applications 130(2), 228–235；[DOI](https://doi.org/10.1541/ieejias.130.228) | 工业滚珠丝杠工作台 | 位置伺服反馈 | 滚动摩擦模型前馈+DOB初始摩擦补偿 | 工业原型实物验证；可访问摘要无统一单值 | 初始运动、慢收敛、方向相关摩擦 | 本项目可检验方向项是否根本不是摩擦 | **是**：滚动摩擦模型+DOB补偿已有直接先例 |
| *Projection-based Iterative Learning Control for Ball-screw-driven Stage with Consideration of Rolling Friction Compensation*；Takumi Hayashi, Hiroshi Fujimoto, Yoshihiro Isaoka, Yuki Terada；2020；IEEJ Journal of Industry Applications 9(2), 132–139；[DOI](https://doi.org/10.1541/ieejjia.9.132) | 工业滚珠丝杠台 | 位置闭环 | 基于滚动摩擦物理基函数的投影ILC | 仿真+实物实验，验证跨位置参考的补偿；摘要无统一单值 | 重复轨迹、方向/滚动摩擦、留出任务 | 本项目可研究非重复、黑箱和自研尺条件 | **强烈是**：方向/摩擦补偿与学习控制都非新概念 |
| *Improvement of Reverse Motion of an NC Moving Table Based on Vector Control Method by Friction Force Compensation*；Akio Hayashi, Tatsuya Mukai, Yusuke Inomata, Yoshitaka Morimoto；2019；International Journal of Automation Technology 13(5), 610–618；[DOI](https://doi.org/10.20965/ijat.2019.p0610) | NC滚珠丝杠/导轨工作台换向lost motion | 伺服电机电流与位置运动测量 | 基于实测转矩关系的换向电流补偿、FPGA矢量控制 | 实物换向试验，抑制lost motion；摘要无统一单值 | 换向、方向误差、控制电流与补偿 | 本项目驱动内部电流环不可见，需外部等效验证 | **强烈是**：换向误差与摩擦力补偿已有直接成熟工作 |
| *An Improved Displacement Sensor Accuracy Calibration Method for Ball Screw Positioning Error Compensation*；Zhaoqian Wu, Canyang Jiang, Fei Jiang, Shaohui Zhang, Jinglun Liang, Rizwan Ullah；2024；IEEE Sensors Journal 24(20), 32499–32507；[DOI](https://doi.org/10.1109/JSEN.2024.3420427) | 滚珠丝杠定位误差与低成本位移传感器 | 外部位移传感器经高精度校准 | FFE-DRSN传感器校正+定位误差补偿 | 不同工况实验，摘要报告相对未补偿定位精度至少提升78% | 外部基准、传感器校准、丝杠误差补偿 | 本项目可追溯计量和物理不确定度与其数据驱动校正不同 | **是**：标定—补偿链也已有新近直接先例 |

**主题判断：**方向相关误差建模、摩擦辨识、lost motion补偿、DOB、ILC和数据驱动校正均已拥挤。阶段F贡献2只能保留为假设：必须证明新的可重复现象、新的跨工况泛化方法，或严格否定常见摩擦归因后形成独特结论。

## 4. CSP/CSV、CANopen与EtherCAT周期多轴控制

| 完整题目；作者；年份；来源；DOI/链接 | 研究对象 | 反馈结构 | 控制结构 | 实验指标 | 与本项目相同点 | 仍可能的差异 | 是否削弱阶段F创新 |
|---|---|---|---|---|---|---|---|
| *Evaluation of EtherCAT Distributed Clock Performance*；Gianluca Cena, Ivan Cibrario Bertolotti, Stefano Scanzio, Adriano Valenzano, Claudio Zunino；2012；IEEE Transactions on Industrial Informatics 8(1), 20–29；[DOI](https://doi.org/10.1109/TII.2011.2172434) | EtherCAT分布时钟 | 网络时钟/节点，不限定位置环 | DC性能测量与统计 | 测量同步偏差/抖动；具体条件依论文，不能迁移到DS-BVM | 统一时基、跨节点同步证据 | 本项目需把通信时钟继续追到驱动目标生效和传感器锁存 | **是**：DC同步测量是成熟主题；也支持阶段FR强调端到端证据 |
| *A Clock Synchronization Method for EtherCAT Master*；Xin Chen, Di Li, Jiafu Wan, Nan Zhou；2016；Microprocessors and Microsystems 46B, 211–218；[DOI](https://doi.org/10.1016/j.micpro.2016.03.002) | Windows RTX EtherCAT主站与参考时钟 | 网络主从时钟 | 主站时钟跟踪/补偿 | 实验验证可行性；摘要无统一可迁移数值 | 主站周期、丢包、DC和应用时钟 | 本项目可比较Windows/实时控制器，但当前无EtherCAT主站 | **是**：主站—参考时钟同步已有专门方法，协议存在不等于系统已同步 |
| *Design Approach Based on EtherCAT Protocol for a Networked Motion Control System*；Lei Wang, Muguo Li, Junyan Qi, Qun Zhang；2014；International Journal of Distributed Sensor Networks 10(2), 750601；[DOI](https://doi.org/10.1155/2014/750601) | 10从站海洋造波网络运动系统 | 各从站传感与伺服全闭环设计 | PC主站、EtherCAT PDO/DC、周期目标/数据 | 访问延迟和抖动实验；摘要无统一单值 | 多从站、周期目标、端到端性能 | 应用、从站和驱动完全不同，不能证明DS-BVM可用 | **是**：EtherCAT多轴/多节点周期运动已有系统实现 |
| *Multi-Motor Synchronous Control with CANopen*；Chung-Wen Hung, Roger C. L. Lee, Bo-Kai Huang, Shih-Ting Yu；2019；Journal of Robotics, Networking and Artificial Life 5, 236–240；[DOI](https://doi.org/10.2991/jrnal.k.190220.006) | 六台BLDC同步 | 电机各自本地驱动反馈 | TI TMS320F28069主站、CiA301/402、SYNC/PDO | 六电机，500 kbit/s与1 Mbit/s；可访问页面无同步误差单值 | MCU主站、CANopen、SYNC/PDO、多电机 | 不是DS-BVM、不是CSP位置精密台，证据深度有限 | **是**：MCU+CANopen多电机同步不是新颖点；同时说明需完整主站固件 |
| *Synchronous Control of Multi Axis System Based on EtherCAT Real Time Industrial Ethernet*；鲁强施、钧华马；2022；轻工机械 40(3), 66–73；[DOI](https://doi.org/10.3969/j.issn.1005-2895.2022.03.011) | 两轴EtherCAT伺服系统 | 从站本地伺服环 | DC同步信号、LAN9252、TMS320F28379D | 两轴实验验证；公开摘要未给可迁移单值 | 嵌入式从站、DC、多轴同步 | 本项目是现成DS驱动+不同主站，且尚无EtherCAT环境 | **是**：EtherCAT DC多轴同步已有直接工程研究 |
| *The Design and Real-Time Optimization of an EtherCAT Master for Multi-Axis Motion Control*；Jianjun Zhang, Manjiang Xia, Han Li, Shasha Li, Juan Shi；2024；Electronics 13(15), 3101；[DOI](https://doi.org/10.3390/electronics13153101) | 六轴伺服主站 | 各轴驱动本地环 | AM64x+Xenomai+IgH EtherCAT主站、CPU/网卡优化 | 500 μs通信周期下最大抖动20 μs、平均抖动1 μs（论文摘要口径） | 实时主站、六轴、周期与抖动实测 | 特定AM64x/网卡/软件，不能迁移到当前PC或DS-BVM | **是**：实时主站优化和多轴周期控制已有成熟实验；有力反证STM32唯一性 |

**主题判断：**CANopen PDO/SYNC、CSP/CSV模式或EtherCAT DC都只能是实现手段。阶段F贡献3若仅写“用STM32+CSP统一时基并扩展三轴”，新颖性明显不足；还必须回答为什么该平台、该端到端时序或该通信误差产生了现有工作未解决的问题。

## 5. 多轴同步、轮廓误差和交叉耦合

| 完整题目；作者；年份；来源；DOI/链接 | 研究对象 | 反馈结构 | 控制结构 | 实验指标 | 与本项目相同点 | 仍可能的差异 | 是否削弱阶段F创新 |
|---|---|---|---|---|---|---|---|
| *Cross-Coupled Biaxial Computer Control for Manufacturing Systems*；Yoram Koren；1980；Journal of Dynamic Systems, Measurement, and Control 102(4), 265–272；[DOI](https://doi.org/10.1115/1.3149612) | CNC双轴路径 | 每轴位置环+轴间误差耦合 | 原始cross-coupled controller | 数学比较表明轮廓精度改善；可访问摘要无统一单值 | 轴向误差与轮廓误差、双轴协调 | 本项目可能加入现代总线/自研尺，但CCC原理不新 | **决定性削弱**：交叉耦合本身已有45年以上历史 |
| *Variable-Gain Cross-Coupling Controller for Contouring*；Yoram Koren, Chung-Ching Lo；1991；CIRP Annals 40(1), 371–374；[DOI](https://doi.org/10.1016/S0007-8506(07)62009-5) | 非直线轮廓双轴系统 | 每轴伺服+轮廓误差耦合 | 随轮廓变化的CCC增益 | 分析/仿真报告轮廓误差降低约3:1至10:1，依轮廓与分辨率 | 变化路径、轮廓误差分配 | 本项目可做真实低速平台/通信约束，但算法概念成熟 | **决定性削弱**：变量增益CCC也非新方法 |
| *Contouring Control of Machine Tool Feed Drive Systems: A Task Coordinate Frame Approach*；George T.-C. Chiu, Masayoshi Tomizuka；2001；IEEE Transactions on Control Systems Technology 9(1), 130–139；[DOI](https://doi.org/10.1109/87.896754) | 工业加工中心进给轴 | 多轴位置反馈转入切/法向坐标 | 时变PD反馈+轨迹前馈、任务坐标系 | Matsuura MC510V实物，进给速度至6 m/min，轮廓性能显著改善 | 切/法向误差、路径几何、实物多轴 | 本项目更低速且通信/传感器问题突出 | **决定性削弱**：任务坐标系和法向控制已有成熟实验证据 |
| *Estimation of the Contouring Error Vector for the Cross-Coupled Control Design*；Syh-Shiuh Yeh, Pau-Lo Hsu；2002；IEEE/ASME Transactions on Mechatronics 7(1), 44–51；[DOI](https://doi.org/10.1109/3516.990886) | 双轴任意轮廓误差估计 | 轴向位置误差映射到轮廓误差向量 | 实时轮廓向量估计与可变耦合增益 | 双轴实物实验验证；摘要无统一单值 | 与阶段F工作簿的误差向量/法向投影直接重合 | 本项目可研究时间不同步如何污染估计 | **决定性削弱**：轮廓误差向量估计不是新贡献 |
| *Analysis and Design of Integrated Control for Multi-Axis Motion Systems*；Syh-Shiuh Yeh, Pau-Lo Hsu；2003；IEEE Transactions on Control Systems Technology 11(3), 375–382；[DOI](https://doi.org/10.1109/TCST.2003.810372) | 三轴CNC加工中心 | 各轴反馈/前馈+多轴耦合 | CETF、反馈、前馈和CCC集成设计 | DYNA 1007三轴实物，跟踪与轮廓精度均改善；摘要无统一单值 | 三轴、轴向/轮廓分层、集成控制 | 本项目可能把端到端时间证据纳入，但需新实验问题 | **决定性削弱**：阶段F“单轴到三轴统一验证”已有近似系统框架 |
| *Contouring Accuracy Improvement Using Cross-Coupled Control and Position Error Compensator*；Ke-Han Su, Ming-Yang Cheng；2008；International Journal of Machine Tools and Manufacture 48(12–13), 1444–1453；[DOI](https://doi.org/10.1016/j.ijmachtools.2008.04.008) | 双轴自由曲线跟踪 | 轴位置反馈+实时轮廓误差 | PEC+改进CCC+模糊feedrate调节 | 多个自由曲线实物实验，显著降低轮廓误差；摘要无统一单值 | 跟踪/轮廓同时改善、速度规划 | 本项目可考察低速方向误差或传感器差异，而非重复控制器 | **决定性削弱**：组合式轮廓/位置/进给补偿已成熟 |

**主题判断：**多轴同步、轮廓误差估计和CCC是高度成熟领域。除非当前平台暴露出由负载侧传感器、黑箱驱动、通信时序或自研尺造成的、现有方法不能解释的新问题，否则应把它们定位为实验工具/条件扩展，不定位为核心创新。

## 6. 四项阶段F贡献的修订结论

| 阶段F贡献 | 有限反查结果 | 当前可保留的最窄表述 | 升格为创新需要的证据 |
|---|---|---|---|
| 证据约束的分层标定—建模 | 组成要素均成熟；未检出阶段F表述本身的新算法 | 项目方法框架/研究规范候选 | 与最接近方法逐项比较；形式化的新规则；对错误结论或实验效率有可量化改善 |
| 方向误差可证伪补偿 | 摩擦、lost motion、DOB、ILC、数据驱动补偿均拥挤 | 方向现象的因果对照与失效域研究 | 新的稳定现象、传感器特有效应、新模型/补偿，且跨工况留出优于强基线 |
| 单轴到三轴统一验证 | 任务坐标系、CCC、CETF、EtherCAT/CANopen同步均成熟 | 条件性系统集成和评价框架 | 证明通信/传感时序与轮廓误差的独特耦合，提出并实测优于既有方法的处理 |
| 自研/商用光栅替换验证 | 替换对照通常是系统验证，不天然新颖 | 条件性器件接入与工程对照 | 自研尺存在新的测量机制/误差特性，并由同条件实验形成不可由常规尺解释的结论 |

## 7. 待继续人工核查的有限检索式

如果导师要求正式文献综述，下一步应在Web of Science、Scopus、IEEE Xplore、ScienceDirect、ASME Digital Collection和CNKI中使用并记录纳排标准：

- `("load-side encoder" OR "linear scale feedback" OR "full closed loop") AND (ball screw OR feed drive) AND positioning`
- `(ball screw AND (reversal OR lost motion OR friction OR backlash) AND (compensation OR calibration))`
- `((CANopen AND SYNC AND PDO) OR (EtherCAT AND distributed clock)) AND (CSP OR cyclic position OR multi-axis motion)`
- `("contour error" AND (cross-coupled OR task coordinate frame OR multi-axis integrated control))`

正式创新判断还需查每项最接近工作的引用与被引链；阶段FR不宣布已完成无遗漏综述。

