# 检索协议与查询日志

## 1. 研究范围

- 核心时间：2021-01-01 至 2026-07-15。
- 2020 年以前只用于经典模型/算法源头、近年论文明确继承的基线和最接近工作的引用链。
- 对象优先级：滚珠丝杠进给/精密位移台 > 柔性进给轴/机床进给系统 > 与问题高度同构的商业伺服闭环。
- 核心问题：物理现象、可观测量、控制输入、采样/同步、辨识边界和验证方法；不按算法名组织创新。

## 2. 当前平台硬约束

1. 输入是 Servo Studio 相对位置目标，不是力、转矩、电流或归一化加速度。
2. Ri0040 进入 DS-BVM 内部位置闭环；内部位置/速度/电流环不可见。
3. CANopen 当前只做 SDO upload；无写控制、PDO、SYNC、NMT Master、CiA 402 运动链。
4. 7 对象运动约 50 Hz、3 对象约 200 Hz；不是 1 ms 控制数据。
5. 对象轮内串行读取、无同步快照；尚无正式外部长度标定。
6. 当前只能研究位置目标到实际位置的外部整体等效闭环及工况相关规律。

## 3. 来源优先级

1. 出版商正式论文页与 DOI 元数据；
2. IEEE、ASME、Elsevier、Springer、Wiley、Taylor & Francis、IET、IFAC、MDPI正式期刊；
3. Crossref/Scopus/WoS元数据；
4. 作者机构仓储的已接收稿，用于阅读全文；
5. CNKI高相关期刊/博士论文；
6. arXiv等预印本仅标 PREPRINT，不计入正式同行评议数量。

博客、营销页、问答、自媒体不进入技术证据。厂家资料只证明接口能力。

## 4. 纳入标准

- 2021 年后；
- 对滚珠丝杠、进给驱动、精密伺服或高度同构对象有直接贡献；
- 至少可核验题名、作者、年份、出版源和 DOI/稳定入口；
- 能回答模型、反馈位置、控制输入、采样/周期、实验还是仿真中的至少两项；
- 与规定主题簇或当前平台适配有明确关系。

## 5. 排除标准

- 只有算法名、没有对象/输入/传感器/实验信息；
- 纯直线电机、机器人或车辆且不能提供可迁移方法边界；
- 纯营销、产品宣传、课程资料；
- 与运动控制无关的滚珠丝杠寿命、制造或热分析；
- 重复版本：预印本与正式版并存时保留正式版，预印本只记录版本关系；
- 只凭标题无法确认相关性且摘要也不可得。

## 6. 去重与核验规则

- DOI 为主键；无 DOI 时用规范化题名 + 第一作者 + 年份。
- Online First 与卷期年分别记录，正式页优先。
- 全文未得：标 ABSTRACT_ONLY，只填摘要可确认字段。
- 原文未给采样率、输入约束或实验数字：写 N/A，不推测。
- 出版商、Crossref、机构仓储冲突：以出版商为主，登记冲突。
- 同一研究组同一平台的系列论文可以都保留，但不当作独立平台重复计数。

## 7. 阶段质量门

### 广泛检索

- 初始 60—100 篇，去重/排除后 40—70 篇；
- 14 类规定主题均有候选；
- 不因达到数量停止。

### 核心核验

- 20—30 篇核心；
- 8—12 篇最接近工作；
- 核心题名、作者、年份、出版源、DOI核验；
- 2024—2026状态核验；
- 只能由全文确认的字段不得从摘要猜测。

### 饱和

- 每核心主题至少 3 篇 2021 后；
- 重点主题至少 1 篇 2024—2026；
- 最近一轮无新主题、无更近工作、新增核心比例低于约 10%；
- 所有算法映射到传感器、权限和采样；
- 所有方向完成反方审查。

## 8. 初始查询族

### 英文

1. ("ball screw" OR "feed drive") AND ("precision motion control" OR positioning OR tracking)
2. ("load-side encoder" OR "linear scale feedback" OR "table-side feedback") AND servo
3. ("commercial servo drive" OR "black-box servo") AND ("closed-loop identification" OR "non-invasive identification")
4. ("ball screw") AND ("position-dependent dynamics" OR LPV OR "gain scheduling")
5. ("ball screw") AND (friction OR reversal OR backlash OR "lost motion" OR "quadrant glitch" OR hysteresis)
6. ("ball screw") AND ("iterative learning control" OR repetitive OR feedforward OR "trajectory shaping")
7. ("ball screw") AND (jerk OR acceleration) AND ("tracking error" OR compensation)
8. ("flexible feed drive" OR "two-mass system") AND (vibration OR resonance OR control)
9. ("closed-loop identification" OR "in-process identification") AND "feed drive"
10. ("data-driven" OR SINDy OR "Gaussian process" OR neural) AND "ball screw"
11. (CSP OR CSV OR EtherCAT OR CANopen) AND "precision motion control"
12. ("contour error" OR "cross-coupled control") AND feed drive

### 中文

13. 滚珠丝杠 进给系统 精密定位 控制 2021 2022 2023 2024 2025
14. 光栅尺 负载侧反馈 伺服 滚珠丝杠
15. 进给系统 闭环辨识 黑箱辨识 商业驱动器
16. 滚珠丝杠 位置相关 动力学 LPV 增益调度
17. 滚珠丝杠 低速 换向 摩擦 回程 迟滞 象限突变
18. 进给系统 加速度 jerk 跟踪误差 前馈 补偿

## 9. 查询日志

| 轮次 | 日期 | 阶段/循环 | 查询族 | 来源 | 原始结果 | 新候选 | 核心新增 | 发现的新主题/冲突 | 审查与下一步 |
|---|---|---|---|---|---:|---:|---:|---|---|
| R0 | 2026-07-15 | 阶段0 PLAN-EXECUTE-AUDIT-REVISE-LOCK | 本地8份模型审计+5份FR证据 | 工作区一级/审计文件 | 13份 | 0 | 0 | 锁定单质量历史基线、位置目标入口、50/200 Hz非同步只读边界 | 阶段0通过；开始R1广泛检索 |
| R1 | 2026-07-15 | 阶段1 PLAN-EXECUTE-AUDIT | 18个初始查询族及其题名/DOI定向变体 | Elsevier、Springer、IEEE、Wiley、MDPI、Nature、中文期刊官网 | 约184条结果 | 84题名级候选；去重保留64 | 14篇完成出版页或全文页核验 | 新主题：在制程内部信号非侵入辨识、模型簇、jerk分段学习；冲突：算法论文的控制电压/电流权限与当前位置命令权限不一致 | 数量门通过；R2补商业黑箱、低速换向、只读外部闭环；继续核验核心字段 |
| R2 | 2026-07-15 | 阶段3/5 REVISE | black-box servo、external position、quadrant glitch、position command/actual position | IEEE/Elsevier/机构页 | 约36条 | 新候选1 | 1（K27） | 数据驱动DOB换向补偿已成熟；未找到同等低率非同步只读限制的论文 | 收窄D2，不把摩擦算法当创新 |
| R3 | 2026-07-15 | 阶段8/9 AUDIT-REVISE-LOCK | closed-loop feed-drive identification、standard G-code、commercial proprietary controller、irregular sampling | 期刊官网/Elsevier | 约24条 | 近五年0；经典1 | 0 | 2011外部商业闭环辨识先例更早；无新主题 | 否定“首次黑箱辨识”；饱和条件满足 |

### R1 审查摘要

- 控制理论：LPV/鲁棒/滑模论文普遍把电压、电流或可改写的外环控制量作为输入；不等于可在 DS-BVM 上以相对位置命令复现。
- 实验与辨识：高质量柔性控制论文普遍需要旋转编码器与直线光栅的同步高频数据、扫频/激励或控制器内部信号；当前采集链不满足。
- 论文与创新：2021—2026 已形成成熟的 LPV、双反馈柔性控制、ILC/轨迹整形和扰动观测文献簇；单纯“换算法”不构成可信创新。更可行缺口转向权限受限商业闭环的外部等效、工况依赖和可证伪边界。

## 10. 每轮三视角审查模板

### 控制理论审查者

- 输入、输出、状态和控制器位置是否明确？
- 稳定性/优越性是否超出原文前提？
- 比较对象、约束、摩擦、采样是否统一？

### 实验与辨识审查者

- 信号是否实际可得、同步、带标定？
- 参数是否独立辨识？
- 是否有实物、独立验证和混淆因素控制？

### 论文与创新审查者

- 是否找到更接近工作和相反结果？
- 是否把成熟算法/总线/传感器名称当创新？
- 假设是否可证伪，且不依赖未确认资源？
