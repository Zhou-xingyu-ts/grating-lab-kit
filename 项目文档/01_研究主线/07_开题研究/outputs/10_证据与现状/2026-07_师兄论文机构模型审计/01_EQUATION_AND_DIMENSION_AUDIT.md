# 式 2.1—2.17 数学与量纲审计

## 0. 审计方法与不确定性

本审计按论文编号逐式检查，并用三类同源材料交叉：

1. thuthesis.pdf 的正文文本层和公式上下文；
2. 答辩 PPT 及论文图片原稿中的框图；
3. 归档 Matlab/Simulink 静态结构，用于判断作者实际实现的意图。

式 2.10、2.15 的 PDF 文本层存在上下标/矩阵错位。由于本任务明确不生成图片，本次没有另存公式截图；相关式仅按“正文上下文 + 原始框图 + 静态源码”重建，并保留 AMBIGUOUS 或“重建式”标记。源码能说明作者想实现什么，不能反向证明论文排版原式正确。

裁决词只使用：CORRECT、CORRECT_WITH_MISSING_STEPS、AMBIGUOUS、DIMENSIONALLY_SUSPECT、ALGEBRAICALLY_SUSPECT、NOT_REPRODUCIBLE_FROM_TEXT。

## 1. 逐式总表

| 公式 | 论文原式/可恢复式 | 变量定义 | 推导来源 | 量纲是否一致 | 符号是否冲突 | 缺失步骤 | 裁决 |
|---|---|---|---|---|---|---|---|
| 2.1 | \(v_{\max}=L/t_{\rm eff}=10{\rm mm}/(0.1{\rm s}\times2/3)=150{\rm mm/s}\) | \(L\)：行程；\(t_{\rm eff}\)：匀速等效运动时间 | 论文把三段中用于面积的有效比例取 \(2/3\) | 是 | 无 | 未画出速度梯形并说明面积 | CORRECT_WITH_MISSING_STEPS |
| 2.2 | \(a=v_{\max}/t_{\rm acc}=150{\rm mm/s}/0.1{\rm s}=1500{\rm mm/s^2}\) | \(t_{\rm acc}\)：加速时间 | 匀加速 \(a=\Delta v/\Delta t\) | 算式自身一致 | 与“加速、匀速、减速各占总时间 \(1/3\)”冲突 | 若总时长 0.1 s，\(t_{\rm acc}=0.1/3\) s，应得 4500 mm/s² | ALGEBRAICALLY_SUSPECT |
| 2.3 | \(\delta_{\rm geo}=\Delta z\sin\alpha+\Delta x\sin\gamma\approx\Delta z\alpha+\Delta x\gamma\) | \(\Delta x,\Delta z\)：直线度误差；\(\alpha,\gamma\)：小角度 | 误差向运动轴的投影和小角近似 | 是，均为长度 | 正负依赖坐标和误差定义 | 未给旋转正向与投影方向 | AMBIGUOUS |
| 2.4 | \(\delta_{\rm installation}=\delta_{\rm abbe}+\delta_{\rm cos}\) | 安装误差由阿贝误差和余弦误差组成 | 误差预算定义 | 是 | 无明显冲突 | 未证明两项独立、可线性相加 | CORRECT_WITH_MISSING_STEPS |
| 2.5 | \(\delta_{\rm abbe}=L_x\tan\gamma+L_z\tan\alpha\approx L_x\gamma+L_z\alpha\) | \(L_x,L_z\)：测量轴与运动轴偏置 | 小转角下偏置臂产生位移 | 是 | 叉乘形式通常含相对正负号；论文未定义方向 | 应从 \(\delta r=\phi\times r\) 推导并标坐标 | AMBIGUOUS |
| 2.6a | \(\phi_1=\arcsin(\Delta h_1/L_{\rm scale})\) | \(\Delta h_1\)：高度差；\(L_{\rm scale}\)：基长 | 直角三角形 | 是，反三角函数输入无量纲 | 无 | 必须说明 \(\lvert\Delta h_1\rvert\le L_{\rm scale}\) 及测量几何 | CORRECT_WITH_MISSING_STEPS |
| 2.6b | \(\phi_2=\arcsin(\Delta h_2/L_{\rm scale})\) | 同上 | 同上 | 是 | 无 | 同上 | CORRECT_WITH_MISSING_STEPS |
| 2.7 | \(\delta_{\rm cos}=l(1/\cos\phi_1+1/\cos\phi_2-2)\) | \(l\)：名义长度；\(\phi_1,\phi_2\)：两个平面夹角 | \(l_{\rm measured}=l/\cos\phi\) 后两项相加 | 是 | 无 | 未画明为何存在两个均以 \(l\) 为基准的独立投影 | CORRECT_WITH_MISSING_STEPS |
| 2.8a | \(F_e=K_fI_q\) | \(F_e\)：直线电磁力；\(I_q\)：q 轴电流；\(K_f\)：论文称力常数 | 电机转矩常数与丝杠转换 | 若 \(K_f\) 为 N/A 则一致；表 2.2 给的却是 N·m/A | \(K_f\) 与器件转矩常数未区分 | 缺 \(\tau=K_tI_q\) 和 \(F=2\pi\eta\tau/p\) | DIMENSIONALLY_SUSPECT |
| 2.8b | \(u_q=L_q\,dI_q/dt+C_ev+R_sI_q\) | \(u_q\)：q 轴电压；\(L_q,R_s\)：电感、电阻；\(v\)：直线速度 | q 轴电压平衡 | 只有 \(C_e\) 被定义为 V·s/m 时一致 | 电机反电势常数通常按转速定义 | 缺 \(\omega=2\pi v/p\)、dq 解耦假设和符号说明 | DIMENSIONALLY_SUSPECT |
| 2.9 | \(F_f=F_c\,{\rm sgn}(v)+F_vv\) | \(F_c\)：库仑摩擦幅值；\(F_v\)：黏性系数 | 库仑 + 黏性摩擦 | 是，\(F_v\) 应为 N·s/m | \(F_f\) 是带符号力还是正幅值须结合 2.10 | \(v=0\) 的静摩擦集合和 Stribeck 未定义 | CORRECT_WITH_MISSING_STEPS |
| 2.10 | \(M\ddot y+B\dot y+Ky=F_e-F_f\)（按上下文重建） | \(M,B,K\)：单质量、阻尼、对地刚度；\(y\)：位移 | 单质量牛顿方程 | 条件成立；\(M\) kg、\(B\) N·s/m、\(K\) N/m | 与前一幅双质量图中的 \(k,c,b_1,b_2\) 不是同一参数集 | 没有从双质量方程降阶，PDF 导数排版亦错位 | NOT_REPRODUCIBLE_FROM_TEXT |
| 2.11 | \(G=1/[Ks^2+(B+F_v)s+K]\)（论文最终写法） | \(G=Y/F_e\) | 2.10 与黏性摩擦闭环/合并 | 否，\(s^2\) 前必须是质量 \(M\)，不能是刚度 \(K\) | \(K\) 同时出现在 \(s^2\) 与常数项 | 正确式应为 \(1/[Ms^2+(B+F_v)s+K]\) | ALGEBRAICALLY_SUSPECT |
| 2.12 | \(z_1=y,\ z_2=\dot y\) | \(z_1,z_2\)：位置、速度状态 | 二阶方程选状态 | 是 | 无 | 无关键缺失 | CORRECT |
| 2.13 | \(\dot z_1=z_2,\ \dot z_2=u-(B+F_v)z_2/M-Kz_1/M\)，\(u=F_e/M\) | \(u\)：归一化加速度输入 | 2.10 忽略库仑摩擦后除以 \(M\) | 是；\(u\) 为 m/s² | 论文前文的输入是力，此处已变成加速度 | 必须显式说明删除 \(F_c{\rm sgn}(v)\) | CORRECT_WITH_MISSING_STEPS |
| 2.14 | \(A=\begin{bmatrix}0&1\\-K/M&-(F_v+B)/M\end{bmatrix},B_u=[0,1]^T,C=[1,0]^T,D=0\) | 状态空间矩阵 | 2.13 | A、B 部分一致 | \(B\) 既是机械阻尼又常作输入矩阵；正文把 C 排成列向量 | 输出式 \(y=Cz\) 要求 \(C=[1,0]\) 为行向量 | ALGEBRAICALLY_SUSPECT |
| 2.15 | \(u=Fz_d+K_ee,\ e=z_d-z,\ F=[K/M,(F_v+B)/M]\)（据 sys.m 恢复意图） | \(z_d\)：目标状态；\(K_e\)：反馈增益；\(F\)：前馈矩阵 | 平衡点前馈 + 状态误差反馈 | 对恒定 \(z_d=[r,0]^T\) 一致 | PDF 文本层矩阵和符号错位；反馈正号依赖 \(e=z_d-z\) | 未给 \(Az_d+B_uFz_d=0\) 条件；非恒定轨迹缺 \(\ddot y_d\) | AMBIGUOUS |
| 2.16 | \(\dot e=(A-B_uK_e)e\)，对应第二行 \(-K/M-k_{e1}\)、\(-(F_v+B)/M-k_{e2}\) | \(e\)：目标减实际 | 2.15 代回 2.14 | 是 | 若误差改成实际减目标，符号全反 | 只对满足平衡条件的恒定目标成立 | CORRECT_WITH_MISSING_STEPS |
| 2.17 | \(K_e=\arg\min J,\ J=\int_0^\infty(z^TQz+u^TRu)dt\) | \(Q,R\)：状态/输入权重 | 连续时间 LQR | 代价项可量纲化，但需由 Q、R 赋权 | 跟踪段应使用误差与增量输入；\(K\) 又与刚度冲突 | 优化变量应是 \(u(\cdot)\) 或明确为 \(J(K)\)，还缺 Riccati 方程 | ALGEBRAICALLY_SUSPECT |

## 2. 关键公式的补齐推导

### 2.1 式 2.1 与 2.2：梯形速度规划

若总时间 \(T=0.1\ {\rm s}\)，加速、匀速、减速各为 \(T/3\)，速度—时间图面积为

\[
L=\frac12v_{\max}\frac{T}{3}
+v_{\max}\frac{T}{3}
+\frac12v_{\max}\frac{T}{3}
=\frac23v_{\max}T
\]

所以式 2.1 的 \(v_{\max}=L/(2T/3)=150\ {\rm mm/s}\) 正确。

但随后应有

\[
a=\frac{v_{\max}}{T/3}
=\frac{150}{0.0333}
\approx4500\ {\rm mm/s^2}
\]

论文与 PPT 均写 1500 mm/s²，说明这是材料中一致重复的逻辑错误，不是单纯 PDF 解析错误。

### 2.2 式 2.8：转矩常数不能直接当力常数

\[
\tau_m=K_tI_q
\]

\[
F_e=\frac{2\pi\eta_f}{p}\tau_m
=\underbrace{\frac{2\pi\eta_fK_t}{p}}_{K_f\,[{\rm N/A}]}I_q
\]

若直接把表 2.2 的 \(K_t=23.38\ {\rm mN\,m/A}\) 当成 \(K_f\)，左边 N、右边 N·m，量纲不闭合。

反电动势同理：

\[
e_q=K_e\omega_m
=K_e\frac{2\pi}{p}v
=C_ev
\]

\[
C_e=K_e\frac{2\pi}{p}
\]

式 2.8b 只有在完成这一步定义后才能使用直线速度 \(v\)。

### 2.3 式 2.10 到 2.11

由

\[
M\ddot y+B\dot y+Ky
=F_e-\left(F_c{\rm sgn}(\dot y)+F_v\dot y\right)
\]

忽略库仑摩擦：

\[
M\ddot y+(B+F_v)\dot y+Ky=F_e
\]

零初值拉普拉斯变换：

\[
\left[Ms^2+(B+F_v)s+K\right]Y=F_e
\]

故

\[
\boxed{\frac{Y}{F_e}
=\frac{1}{Ms^2+(B+F_v)s+K}}
\]

归档 mechanical_system.slx 的传递函数分母参数为 [M B K]，黏性摩擦由外部块反馈，因此也证明作者实现意图中的首项是 \(M\)，而不是论文式 2.11 打印出的 \(K\)。

### 2.4 式 2.14：矩阵形状

\[
z\in\mathbb R^2,\quad u\in\mathbb R,\quad y\in\mathbb R
\]

所以

\[
A\in\mathbb R^{2\times2},\quad
B_u\in\mathbb R^{2\times1},\quad
C\in\mathbb R^{1\times2},\quad
D\in\mathbb R
\]

正文的 \(C=[1,0]^T\) 若真是列向量，就不能计算标量 \(y=Cz\)。归档源码的 \(C=[1\ 0]\) 是正确实现。

### 2.5 式 2.15—2.16：恒定目标前馈

对象为

\[
\dot z=Az+B_uu
\]

取 \(e=z_d-z\)、\(u=Fz_d+K_ee\)。对恒定 \(z_d\)：

\[
\dot e=-Az-B_uu
\]

代入 \(z=z_d-e\)：

\[
\dot e
=-\left(Az_d-Ae+B_uFz_d+B_uK_ee\right)
\]

若

\[
Az_d+B_uFz_d=0
\]

则

\[
\dot e=(A-B_uK_e)e
\]

对 \(z_d=[r,0]^T\)，取 \(Fz_d=(K/M)r\) 可以满足平衡条件。这是静态位置前馈，不是一般轨迹对象逆。

### 2.6 式 2.17：LQR 的完整形式

定义标准状态误差 \(\tilde e=z-z_d\)。对恒定目标，其增量误差系统为

\[
\dot{\tilde e}=A\tilde e+B_u\Delta u
\]

标准连续 LQR 是

\[
\min_{\Delta u(\cdot)}
\int_0^\infty(\tilde e^TQ\tilde e+\Delta u^TR\Delta u)dt
\]

Riccati 方程：

\[
A^TP+PA-PB_uR^{-1}B_u^TP+Q=0
\]

反馈律：

\[
\Delta u=-R^{-1}B_u^TP\tilde e=-K_{\rm LQR}\tilde e
\]

若沿用论文/源码的 \(e=z_d-z=-\tilde e\)，同一控制律应写成 \(\Delta u=+K_{\rm LQR}e\)。论文把“最小化代价”和“反馈增益”压成一行，省略了优化变量、Riccati 方程、误差符号以及跟踪偏置处理。

## 3. 分子、分母阶次审计

| 对象 | 理论阶次 | 论文/文件实际 | 裁决 |
|---|---:|---|---|
| 两质量、每个质量含位移和速度 | 4 阶 | 图 2.15 只有结构图，无方程和参数 | 双质量不可复现 |
| 双质量 \(F_\tau\to X_2\) | 一般四阶分母；分子含 \(cs+k\) | 论文未给 | 未进入最终控制设计 |
| 单质量线性对象 | 二阶分母、常数分子 | 式 2.11 和 mechanical_system.slx | 实际采用；式 2.11 把 \(M\) 误写为 \(K\) |
| PID 闭环 | 对象 2 阶 + 积分/微分滤波状态 | PID.slx | 可形成更高阶闭环，但不是机械双质量阶次 |
| LQR 对象 | 2 状态 | LQR.m、sys.m | 无柔性模态 |

结论：最终仿真中即使出现振荡，也只能来自单个二阶机械极点、控制器状态和非线性摩擦，不能据此声称保留了双质量共振。

## 4. 同名与符号冲突

| 符号 | 冲突 | 影响 |
|---|---|---|
| \(M\) | 表 2.6 为 0.5 kg；PID/LQR 文件为 7.7 kg | 结果不对应同一个名义对象 |
| \(K\) | 机械刚度；式 2.11 又错误占据惯性项；LQR.m 中先为刚度、随后被 lqr 返回增益覆盖 | 极易把 N/m 与状态反馈增益混用 |
| \(B\) | 机械阻尼 \(B\)；状态空间输入矩阵也惯用 \(B\) | 必须写成 \(B_u\) 才不混淆 |
| \(F\) | 力常用 \(F\)；式 2.15 又把前馈矩阵命名为 \(F\) | 物理量和矩阵角色混淆 |
| \(K_f\) | 论文称力常数，器件表只给转矩常数 | 缺丝杠导程转换 |
| \(C_e\) | 用直线速度，但电机常数通常定义在角速度域 | 缺 \(2\pi/p\) |
| \(e\) | PID 正文写 \(Y-R\)；框图和 PID.slx 是 \(R-Y\)；LQR 源码是 \(z_d-z\) | 正反馈/负反馈的符号判断会反转 |
| \(C\) | 输出矩阵排成列向量 | 矩阵乘法维数错误 |

## 5. 归档三状态探索模型的额外警告

Misc.mlx 中另有一个含位置、速度、电流的三状态模型，参数包括 \(R_s=3.3\)、\(L_q=2.62\)、\(K_f=23.6\)、\(C_e=19.2\)、\(M=7.7\)、\(B=0.5\)、\(K=5\)。其静态矩阵第二行没有把机械项除以 \(M\)，电气自反馈项写成 \(+R_s/L_q\) 而标准无源电阻项应为负号。该文件不应被视为论文最终模型，也不能用来修补式 2.8 的参数来源。

## 6. 总裁决

1. 可直接保留的数学骨架：2.1 的梯形速度面积、2.9 在 \(v\ne0\) 时的库仑+黏性形式、2.12 的状态选择、2.13 的线性二阶状态方程、2.16 在恒定平衡目标条件下的误差动力学。
2. 明确公式问题：2.2 的加速时间逻辑、2.11 的 \(M\to K\) 误写、2.14 的 \(C\) 维数、2.17 的 LQR 优化表述。
3. 明确量纲断点：2.8a 的转矩—力转换、2.8b 的角速度—直线速度转换。
4. 不能由正文复现：图 2.15 双质量模型到式 2.10 单质量模型的降阶。
5. 必须保留不确定性：2.3/2.5 的坐标正负、2.15 的 PDF 排版与跟踪律适用范围。
