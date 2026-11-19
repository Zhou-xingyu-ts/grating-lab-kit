# 阶段E最终交接：参数可辨识性、标定链与返校实验协议

> 本文件可独立交给网页端GPT审核。阶段E基于D1最终模型，完成A/B/D参数可辨识性分类、正式标定链、时间/数据合同、安全激励设计和纯离线合成辨识验证。没有重新审计阶段A—D，没有修改D1或`motion_acquisition`，没有访问硬件、发送命令、实现高级控制器或把合成参数称为台架参数。

## 1. 阶段E命名

- `Kp_pos`：B外位置控制器比例增益（D1代码`kp_per_s`），单位1/s。
- `KD_dc`：D二阶黑箱直流增益（D1历史公式$K_p$/代码`closed_loop_gain`），无量纲。

阶段E用不同名字消除“两个$K_p$”歧义，不改D1历史文件。

## 2. D1修补后基线版本

D1当前代码于2026-07-15通过15/15单元测试。阶段E只调用`run_baselines()`重算12条B+3条D默认基线；未运行35条敏感性扫描，未生成图片，未写回D1。

阶段E保存：

- 当前重算：`results/d1_baseline_regenerated.csv`；
- 与D1旧CSV逐字段比较：`results/d1_baseline_comparison.csv`；
- 代码SHA-256、种子、参数、测试与产物状态：`results/d1_baseline_provenance.json`。

D1旧`baseline_metrics.csv`和图片是修补前生成物。显式零命令历史改变含噪B首拍对象输入，因此120个数值字段有非零差异；点到点五项核心指标最大绝对差1.0 μm。D1旧文本字段无变化。这些差异不改变“全部是H/R离线模型”的等级。

## 3. 可辨识性核心结论

### A

count—位移比例只有通过外部长度基准、多位置、正反向重复和不确定度预算才能独立辨识。现有内部位置闭环可以辨识外部等效动态、总延迟、重复性和方向误差映射，但不能从目标—位置数据分解驱动内部控制器与机械对象。Velocity/A/D/J需单变量或因子化重复，现有150/300组合只能支持参数敏感现象。

### B

- $K_v$在速度命令尺度、位置比例和对象输入已知时可辨识；仅闭环位置时与`Kp_pos`和尺度混淆。
- $\tau_v$需要覆盖拐点频带并独立处理延迟。
- 只有位置记录时，$d_y$和$d_u$不能独立分离。
- 位置量化/噪声可统计，但物理单位依赖正式标定。
- 方向残余项只能辨识$F/M$；没有独立质量/力测量时不能分别给$F$和$M$。
- 速度/加速度限制只有记录限幅前后命令时才能独立识别。

### D

- `KD_dc/omega_n/zeta`只有输入覆盖低频和固有频率附近、SNR及采样充分时才实际唯一。
- $d_u$、更新周期、量化、ZOH和漏更新需计划/发送/总线/生效事件链。
- 内部目标插值不能从低率SDO位置独立辨识，需RPDO/SYNC和高率反馈/总线时间证据。

六项强制判断均已落实：$F/M$结构不可分；$K_v$存在尺度混淆；位置记录无法分离$d_y/d_u$；$\tau_v$与延迟混淆；D低带宽参数不唯一；方向差异不能与重力、间隙、线缆拖曳和时间错位自动区分。

## 4. 正式count—位移标定链

比较了激光干涉仪、独立高精度位移传感器和千分表/测微头类别，不推荐品牌。正式协议包括：预热和环境、坐标/零点、至少11个覆盖行程的位置点、每点正反向重复、回程误差、重复性、线性拟合、必要时有条件的非线性映射、残差、独立验证点和不确定度预算。

`10000 counts≈5 mm`与`0.5 μm/count`在正式标定前仍只是工程换算；一个count步长不等于绝对精度。映射必须绑定方向、位置、温度、负载和安装适用范围。

## 5. 时间语义与数据合同

合同定义了目标要求的全部字段：`experiment_id/cycle_id/host_monotonic_ns/cycle_start_ns/object_request_ns/object_response_ns/feedback_available_ns/command_compute_ns/command_tx_ns/object_index/object_subindex/value/success/error_type/source_route/target_position/actual_position/following_error/actual_velocity/experiment_condition`，并建议增加计划轮次时刻、轮结束、对象顺序、传感器锁存、总线帧、SYNC、目标生效、标定ID和代码哈希。

当前`motion_acquisition`一轮只生成一次时间戳后顺序读取对象，所以同轮对象不是同步采样。实际轮次率必须从首末`cycle_start`计算；现有`estimated_rate_hz`只是工作段吞吐，`deadline_miss_count`只是工作段overrun。Windows单调时钟能证明主机观察顺序和相对耗时，不能证明设备内部采样、真正上总线、跨设备同步或硬实时最坏时延。

$d_y$需要传感器锁存—反馈帧/响应—反馈可用—控制采用事件；$d_u$需要命令计算—提交—总线帧—驱动目标生效/回显。PDO/SYNC还需SYNC、RPDO/TPDO、映射/传输类型、设备采样语义和时钟映射。

本阶段只定义合同，未修改采集软件。

## 6. 安全激励设计

A、B、D分别覆盖：小幅阶跃、五次S曲线、恒速正反向扫描、多速度/多加速度往返、多正弦、扫频、PRBS。每类均说明能激励和不能辨识的内容、幅值/带宽、持续和重复、行程/饱和风险、训练与留出。

共同原则是从静止和最小可观测幅值逐级增加，命令/反馈/行程/电流保留停止余量；PRBS/扫频最后实施并需单独安全放行。阶段E没有生成任何真实驱动执行序列。

## 7. 合成辨识框架

源码`src/phase_e_identification.py`只使用NumPy：

- B：整数延迟候选下拟合$v[k+1]=av[k]+bu[k-d]+c$，恢复$K_v$和$\tau_v$；
- D：拟合二阶离散ARX，离散极点经$s=\ln z/T_s$映射恢复`KD_dc/omega_n/zeta`；
- 连续二次微分回归只作噪声敏感性对照；
- 训练/留出按完整轨迹分开；
- 残差报告均值、RMS、一阶自相关和输入相关；
- 所有入口双安全守卫。

阶段E8/8单元测试通过。首次测试发现D直接二次微分的$\omega_n$偏差约4.9%超过3%阈值，随后改用离散极点主方法，没有通过放宽阈值掩盖问题。

## 8. 合成结果

### 宽带成功案例

- B真值$K_v=0.92,\tau_v=35$ ms,$d=3$拍；全部正确恢复，独立多正弦留出RMSE约$6.73\times10^{-17}$ m/s。
- D真值`KD_dc=0.97`、20 Hz、$\zeta=0.82$、$d=2$拍；全部正确恢复，独立频线留出RMSE约$7.89\times10^{-11}$ m。

### 噪声/量化偏差

- B位置加入0.25 μm噪声和0.5 μm量化后，$K_v$偏差-1.82%、$\tau_v$偏差+1.68%，残差仍与输入相关。
- D同样噪声/量化后，`KD_dc`偏差+0.014%、$\omega_n$偏差+0.95%、$\zeta$偏差+1.68%；直接二次微分给出更大偏差。

### 失败案例

1. B低带宽：真实35 ms/3拍，最低误差却给约8.0 ms/6拍；5拍候选RMSE只高3%，不能可靠分离。
2. $F/M$：0.10 N/0.8 kg与0.20 N/1.6 kg输出差为0，结构不可辨识。
3. D低带宽：20 Hz/$\zeta=0.8$与35 Hz/$\zeta=1.1$候选在1.00/1.24 μm RMSE下都能拟合，参数不唯一。

这些数值只证明方法和失败机制，不是台架参数。

## 9. A/B/D决策门

- **A可继续为主要实施路线：**当B/D前提不能关闭，但A能完成正式标定、重复性、方向误差建模和独立验证补偿时。
- **D进入实施准备：**型号匹配CSP、PDO、SYNC、目标更新/量化/ZOH/漏更新、插值、时序和完整安全协议全部明确。
- **B进入实施准备：**独立电机反馈、光栅进入实时控制器、速度环/CSV、PDO/SYNC、$d_y/d_u$测量、尺度标定、实时性及限位/STO/Quick Stop/heartbeat/失联保护全部验证。

当前A可继续研究；B、D均未通过实施准备门。不得凭合成仿真放行。

## 10. 产物清单

- `00_D1_PATCH_AND_BASELINE_PROVENANCE.md`
- `01_PARAMETER_IDENTIFIABILITY_MATRIX.md`
- `02_CALIBRATION_CHAIN_DESIGN.md`
- `03_TIMING_AND_DATA_CONTRACT.md`
- `04_EXCITATION_AND_EXPERIMENT_DESIGN.md`
- `05_SYNTHETIC_IDENTIFICATION_METHODS.md`
- `06_SYNTHETIC_IDENTIFICATION_REPORT.md`
- `07_A_B_D_DECISION_GATES.md`
- `08_USER_LEARNING_GUIDE.md`
- `09_LAB_EXECUTION_CHECKLIST.md`
- `src/`、`scripts/`、`tests/`、`results/`
- `PHASE_E_SELF_REVIEW.md`、本文件

## 11. 纯离线复现

```powershell
python -m unittest discover -s tests -p "test_*.py" -v
python scripts\regenerate_d1_baseline.py --simulation-only true --hardware-disabled true --test-status "15/15 passed"
python scripts\run_synthetic_identification.py --simulation-only true --hardware-disabled true
```

第二条只生成15条默认基线；不得改成D1全脚本入口以免重跑敏感性扫描。

## 12. 建议网页端GPT审核问题

1. A/B/D矩阵的“独立/组合/不可辨识”分类是否与所需输入输出一致？
2. $F/M$、`Kp_pos-Kv-尺度`和位置记录下$d_y/d_u$的结构判断是否充分？
3. $\tau_v—delay$和D低带宽失败案例是否足以否定“最低训练误差即真值”？
4. 标定链是否能区分count步长、重复性、线性度和绝对精度？
5. 时间合同是否正确降级现有SDO轮次时间戳、`estimated_rate_hz`和`deadline_miss_count`？
6. PDO/SYNC所需总线侧证据是否足以测$d_u$与内部目标行为？
7. 激励矩阵是否在辨识带宽和运动安全之间保持了可执行但不越界的原则？
8. B/D主辨识方法、留出验证和残差解释是否可复现？
9. 合成噪声/量化偏差是否被正确限制为方法验证，而非真实参数？
10. A/B/D决策门是否避免凭合成仿真或只读SDO放行硬件路线？

## 13. 最终结论与停止

阶段E完成了D1基线版本核查、参数可辨识性分类、正式标定链、时间数据合同、安全激励、B/D合成辨识、失败案例、留出验证、残差分析、路线决策门、用户学习材料和返校清单。所有运行均为纯离线，B/D当前未放行。

到此停止，不进入控制器增强、STM32开发、真实硬件实验或下一研究阶段。

