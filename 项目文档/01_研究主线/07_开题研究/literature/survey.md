# 开题文献综述总表

> 生成日期：2026-06-01。用途：支撑“自研光栅尺反馈闭环运动控制”开题方向选择。PDF 和精修引用以 Zotero 集合为主，本文件保存结构化阅读线索。

## 1. 文献分类统计

| 分类 | 数量 | 对课题的作用 |
|---|---:|---|
| 01_光栅尺与光栅干涉仪 | 6 | 说明测量端能力、多自由度潜力和ACSM背景 |
| 02_精密运动平台与纳米定位 | 7 | 说明可选实验平台和闭环对象 |
| 03_运动控制算法 | 13 | 筛选可落地控制算法 |
| 04_多轴同步与轮廓误差 | 7 | 支撑二维/多轴扩展和轮廓误差 |
| 05_实验评价与误差补偿 | 4 | 支撑误差补偿和评价方法 |
| 06_组内与导师相关论文 | 2 | 对齐组内/导师路线 |

## 2. 核心文献清单

### 1. Precision Nanometrology: Laser Interferometer, Grating Interferometer and Time Grating Sensor

- 年份/来源：2025，Sensors
- DOI/URL：https://doi.org/10.3390/s25216791
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：光栅尺/光栅干涉测量链路
- 传感器：激光测量/干涉仪
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：开题背景综述：说明光栅干涉仪与激光干涉仪、时栅传感器在纳米计量中的定位。

### 2. A Review: Grating Encoder Technologies for Multi-Degree-of-Freedom Spatial Measurement

- 年份/来源：2025，Sensors
- DOI/URL：https://doi.org/10.3390/s25196071
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：多自由度测量/定位系统
- 传感器：光栅尺/光栅编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：支撑多自由度光栅尺作为未来升级方向，但不宜作为第一阶段承诺。

### 3. An Ultra-Precision Absolute-Type Multi-Degree-of-Freedom Grating Encoder

- 年份/来源：2022，Sensors
- DOI/URL：https://doi.org/10.3390/s22239047
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：多自由度测量/定位系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：组内/相近方向参考：多自由度光栅编码器说明测量端能力，但控制端仍需另建。

### 4. Ultraprecision Real-Time Displacements Calculation Algorithm for the Grating Interferometer System

- 年份/来源：2019，Sensors
- DOI/URL：https://doi.org/10.3390/s11105957
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：光栅尺/光栅干涉测量链路
- 传感器：光栅尺/光栅编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：强调实时位移解算和多自由度误差耦合，适合写“信号能读到不等于可控”。

### 5. Development of a grating based multi-degree-of-freedom laser linear encoder using diffracted light

- 年份/来源：2012，Sensors and Actuators A: Physical
- DOI/URL：https://doi.org/10.1016/j.sna.2012.05.004
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：多自由度测量/定位系统
- 传感器：激光测量/干涉仪
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：多自由度光栅编码器经典工程参考。

### 6. NPS6D200—A Long Range Nanopositioning Stage with 6D Closed Loop Control

- 年份/来源：2024，Applied Sciences
- DOI/URL：https://doi.org/10.3390/app14166972
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：多自由度测量/定位系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：高目标参考：长行程、6D闭环，说明多自由度闭环系统复杂度。

### 7. Development of a Large-Range XY-Compliant Micropositioning Stage with Laser-Based Sensing and Active Disturbance Rejection Control

- 年份/来源：2024，Sensors
- DOI/URL：https://doi.org/10.3390/s24020663
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：XY/双轴精密平台
- 传感器：激光测量/干涉仪
- 执行器：柔性/顺应微定位台
- 算法/方法：ADRC
- 本课题可借鉴点：直接参考：XY平台+外部测量+ADRC，适合方向B论证。

### 8. Closed-Loop Range-Based Control of Dual-Stage Nanopositioning Systems

- 年份/来源：2021，IEEE/ASME Transactions on Mechatronics
- DOI/URL：https://doi.org/10.1109/tmech.2020.3020047
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：双级平台闭环控制参考，说明粗精结合和范围约束。

### 9. Digital closed-loop nanopositioning using rectilinear flexure stage and laser interferometry

- 年份/来源：2005，Control Engineering Practice
- DOI/URL：https://doi.org/10.1016/j.conengprac.2004.04.019
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：精密运动控制系统
- 传感器：激光测量/干涉仪
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：经典：外部干涉仪反馈闭环纳米定位，用于历史脉络。

### 10. Large range nanopositioning stage design: A three-layer and two-stage platform

- 年份/来源：2016，Measurement
- DOI/URL：https://doi.org/10.1016/j.measurement.2016.03.075
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：平台设计参考；提示正式平台不是算法单点问题。

### 11. Design Optimization for the Measurement Accuracy Improvement of a Large Range Nanopositioning Stage

- 年份/来源：2016，Sensors
- DOI/URL：https://doi.org/10.3390/s16010084
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：平台结构和测量精度耦合参考。

### 12. A dual-loop tracking control approach to precise nanopositioning

- 年份/来源：2018，Journal of Vibration and Control
- DOI/URL：https://doi.org/10.1177/1077546318793219
- 主题分类：02_精密运动平台与纳米定位
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：内环阻尼/外环跟踪思想，支撑分层控制结构。

### 13. Integral back-stepping active disturbance rejection control for piezoelectric stick-slip drive nanopositioning stage

- 年份/来源：2023，Review of Scientific Instruments
- DOI/URL：https://doi.org/10.1063/5.0150909
- 主题分类：03_运动控制算法
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：压电执行器
- 算法/方法：ADRC
- 本课题可借鉴点：ADRC在纳米定位平台上的近期落地参考。

### 14. Phase leading active disturbance rejection control for a nanopositioning stage

- 年份/来源：2021，ISA Transactions
- DOI/URL：https://doi.org/10.1016/j.isatra.2021.01.004
- 主题分类：03_运动控制算法
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：ADRC
- 本课题可借鉴点：ADRC处理带宽/相位裕度问题的参考。

### 15. Active disturbance rejection control for nanopositioning systems with piezoelectric actuators

- 年份/来源：2016，ICCAS
- DOI/URL：https://doi.org/10.1109/iccas.2016.7832478
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：压电执行器
- 算法/方法：ADRC
- 本课题可借鉴点：ADRC用于压电纳米定位的早期参考。

### 16. Disturbance Observer Based Model Prediction Control for a 2-DOF Nanopositioning Stage

- 年份/来源：2019，IECON
- DOI/URL：https://doi.org/10.1109/iecon.2019.8927729
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：DOB/MPC
- 本课题可借鉴点：DOB+MPC组合，适合作为进阶算法储备。

### 17. Feedforward Tuning by Fitting Iterative Learning Control Signal for Precision Motion Systems

- 年份/来源：2021，IEEE Transactions on Industrial Electronics
- DOI/URL：https://doi.org/10.1109/tie.2020.3020032
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：ILC/轮廓误差补偿
- 本课题可借鉴点：把ILC学习结果转为前馈，适合重复轨迹/扫描任务。

### 18. Learning Feedforward Control Using Multiagent Control Approach for Motion Control Systems

- 年份/来源：2021，Energies
- DOI/URL：https://doi.org/10.3390/en14020420
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：前馈/学习前馈
- 本课题可借鉴点：学习型前馈作为展望，不建议第一阶段主线。

### 19. Adaptive Feedforward Control For Reset Feedback Control Systems - Application in Precision Motion Control

- 年份/来源：2021，European Control Conference
- DOI/URL：https://doi.org/10.23919/ecc54610.2021.9654834
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：前馈/学习前馈
- 本课题可借鉴点：自适应前馈参考。

### 20. Adaptive RBFNN-based predictive control for the nanopositioning of an electrostatic MEMS actuator

- 年份/来源：2023，IET Control Theory & Applications
- DOI/URL：https://doi.org/10.1049/cth2.12586
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：DOB/MPC
- 本课题可借鉴点：学习/预测控制作为前沿储备，不作为近期落地主线。

### 21. Robust Control of Dual-Linear-Motor-Driven Gantry Stage for Coordinated Contouring Tasks Based on Feed Velocity

- 年份/来源：2023，IEEE Transactions on Industrial Electronics
- DOI/URL：https://doi.org/10.1109/tie.2022.3199920
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：双直线电机龙门/XY平台
- 传感器：外部位移传感器/编码器
- 执行器：直线电机
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：XY/龙门平台轮廓控制重点参考，支撑方向B。

### 22. Accurate Contour Error Estimation-based robust contour control for dual-linear-motor-driven gantry stages

- 年份/来源：2024，Mechatronics
- DOI/URL：https://doi.org/10.1016/j.mechatronics.2024.103174
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：双直线电机龙门/XY平台
- 传感器：外部位移传感器/编码器
- 执行器：直线电机
- 算法/方法：轮廓误差估计与补偿
- 本课题可借鉴点：轮廓误差估计与鲁棒控制，适合多轴扩展论证。

### 23. Adaptive Iterative Learning Constrained Control for Linear Motor-Driven Gantry Stage with Fault-Tolerant Non-Repetitive Trajectory Tracking

- 年份/来源：2024，Mathematics
- DOI/URL：https://doi.org/10.3390/math12111673
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：双直线电机龙门/XY平台
- 传感器：外部位移传感器/编码器
- 执行器：直线电机
- 算法/方法：ILC/轮廓误差补偿
- 本课题可借鉴点：ILC/约束控制用于线性电机龙门平台，作为多轴算法储备。

### 24. Design of a trajectory contour controller for a dual-axis precision motion stage based on improved iterative learning

- 年份/来源：2023，IET Control Theory & Applications
- DOI/URL：https://doi.org/10.1049/cth2.12472
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：XY/双轴精密平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：ILC/轮廓误差补偿
- 本课题可借鉴点：双轴精密平台ILC轮廓控制，方向B关键文献。

### 25. An iterative contouring error compensation scheme for five-axis precision motion systems

- 年份/来源：2022，Mechanical Systems and Signal Processing
- DOI/URL：https://doi.org/10.1016/j.ymssp.2022.109226
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：ILC/轮廓误差补偿
- 本课题可借鉴点：多轴轮廓误差补偿，适合开题综述中的高阶参考。

### 26. Five-axis contouring error compensation based on variable-gain iterative learning

- 年份/来源：2025，MIC 2024
- DOI/URL：https://doi.org/10.1117/12.3065397
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：ILC/轮廓误差补偿
- 本课题可借鉴点：近年多轴ILC轮廓补偿参考。

### 27. A novel curvature circle iterative algorithm for contour error control of multi-axis CNC machine tools

- 年份/来源：2020，Precision Engineering
- DOI/URL：https://doi.org/10.1016/j.precisioneng.2020.05.005
- 主题分类：05_实验评价与误差补偿
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：轮廓误差估计与补偿
- 本课题可借鉴点：轮廓误差定义与估计方法可迁移到XY平台。

### 28. Friction compensation for Nanopositioning and Nanomeasuring Machines

- 年份/来源：2010，AMC
- DOI/URL：https://doi.org/10.1109/amc.2010.5464105
- 主题分类：05_实验评价与误差补偿
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：误差补偿经典：摩擦是高精度闭环不可忽略项。

### 29. Control of an Overactuated Nanopositioning System with Hysteresis by Means of Control Allocation

- 年份/来源：2022，AMC
- DOI/URL：https://doi.org/10.1109/amc51637.2022.9729294
- 主题分类：05_实验评价与误差补偿
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：多执行器/过驱动平台控制分配参考。

### 30. Design Optimization for the Measurement Accuracy Improvement of a Large Range Nanopositioning Stage

- 年份/来源：2016，Sensors
- DOI/URL：https://doi.org/10.3390/s16010084
- 主题分类：05_实验评价与误差补偿
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：测量精度和平台设计耦合，用于提醒不能自证闭环精度。

### 31. Motion control system design pattern from Li Xinghui RSI 2016 paper

- 年份/来源：2016，Review of Scientific Instruments
- DOI/URL：https://doi.org/10.1063/1.4953653
- 主题分类：06_组内与导师相关论文
- 平台/对象：精密运动控制系统
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：导师早期运动控制实验组织范式：需求、模型、控制、实验指标闭环论证。

### 32. Lai Jianjun thesis system: grating-scale full closed-loop and error compensation workflow

- 年份/来源：2020，Internal thesis material
- DOI/URL：待补
- 主题分类：06_组内与导师相关论文
- 平台/对象：光栅尺/光栅干涉测量链路
- 传感器：光栅尺/光栅编码器
- 执行器：依平台而定
- 算法/方法：测量/建模/实验评价
- 本课题可借鉴点：内部资料：光栅尺全闭环、查表补偿、外部基准区分，是本课题实验评价的重要来源。

### 33. Grating interferometer: The dominant positioning strategy in atomic and close-to-atomic scale manufacturing

- 年份/来源：2025，Journal of Manufacturing Systems
- DOI/URL：https://doi.org/10.1016/j.jmsy.2025.08.008
- 主题分类：01_光栅尺与光栅干涉仪
- 平台/对象：光栅尺/光栅干涉测量链路
- 传感器：光栅干涉仪/光栅尺
- 执行器：依平台而定
- 算法/方法：测量/建模/误差补偿
- 本课题可借鉴点：2025年ACSM综述，适合作为光栅干涉仪定位策略、误差源和补偿方法的最高层背景。

### 34. Cross-coupled iterative learning control: A computationally efficient approach applied to an industrial flatbed printer

- 年份/来源：2024，Mechatronics
- DOI/URL：https://doi.org/10.1016/j.mechatronics.2024.103170
- 主题分类：04_多轴同步与轮廓误差
- 平台/对象：XY/双轴精密平台
- 传感器：外部位移传感器/编码器
- 执行器：工业平板打印机运动平台
- 算法/方法：交叉耦合ILC/轮廓误差补偿
- 本课题可借鉴点：说明重复轨迹和轮廓误差补偿可以在工业多轴平台落地，适合作为XY扩展算法依据。

### 35. A design framework for nonlinear iterative learning control and repetitive control: Applied to three mechatronic case studies

- 年份/来源：2024，Control Engineering Practice
- DOI/URL：https://doi.org/10.1016/j.conengprac.2024.105976
- 主题分类：03_运动控制算法
- 平台/对象：精密运动控制系统
- 传感器：依平台而定
- 执行器：依平台而定
- 算法/方法：非线性ILC/重复控制
- 本课题可借鉴点：说明ILC/RC不是单一算法名词，而是可按重复任务设计的框架；适合扫描轨迹和轮廓误差。

### 36. LSTM-Inversion-Based Feedforward-Feedback Nanopositioning Control

- 年份/来源：2024，Machines
- DOI/URL：https://doi.org/10.3390/machines12110747
- 主题分类：03_运动控制算法
- 平台/对象：纳米定位平台
- 传感器：外部位移传感器/编码器
- 执行器：依平台而定
- 算法/方法：LSTM逆模型前馈+反馈控制
- 本课题可借鉴点：支撑学习型前馈可作为展望，但需要数据量、验证和安全边界，不适合第一阶段承诺。

### 37. Proportional-integral-proportional control and compensation design for low-speed motions of permanent magnet synchronous motor driven servomechanism with position-dependent disturbance

- 年份/来源：2022，Journal of the Chinese Institute of Engineers
- DOI/URL：https://doi.org/10.1080/02533839.2022.2101532
- 主题分类：03_运动控制算法
- 平台/对象：低速伺服运动系统
- 传感器：电机侧/位置反馈
- 执行器：永磁同步电机伺服机构
- 算法/方法：PI/PIP+扰动补偿
- 本课题可借鉴点：低速、小行程伺服运动与本课题近期 Z 轴/单轴验证相近，支撑 PID/PI 作为第一版可落地基线。

### 38. Cascaded proportional-integral-derivative controller parameters tuning for contour following improvement

- 年份/来源：2016，Proceedings of the Institution of Mechanical Engineers, Part I: Journal of Systems and Control Engineering
- DOI/URL：https://doi.org/10.1177/0959651816657454
- 主题分类：03_运动控制算法
- 平台/对象：多轴轮廓跟踪系统
- 传感器：位置反馈/运动控制反馈
- 执行器：依平台而定
- 算法/方法：级联 PID 参数整定
- 本课题可借鉴点：支撑二维/多轴阶段仍需以级联 PID 等传统控制作为可比较基线。

### 39. Dynamic modeling, controller design and experimental validation of a planar motion stage for precision positioning

- 年份/来源：2005，Precision Engineering
- DOI/URL：https://doi.org/10.1016/j.precisioneng.2004.11.005
- 主题分类：03_运动控制算法
- 平台/对象：平面精密运动平台
- 传感器：位置反馈/运动控制反馈
- 执行器：平面运动平台
- 算法/方法：PID+DOB/模型控制
- 本课题可借鉴点：平面精密平台的建模、控制器设计和实验验证参考，可用于方向 B 的基础控制对照。
