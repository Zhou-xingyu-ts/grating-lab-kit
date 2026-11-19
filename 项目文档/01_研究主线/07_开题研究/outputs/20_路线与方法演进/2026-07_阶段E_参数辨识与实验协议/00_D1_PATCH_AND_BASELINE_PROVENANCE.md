# D1修补与默认基线版本清单

> 本文件只建立阶段D1修补后代码与默认基线产物的版本关系。未重审阶段A—D，未运行D1敏感性扫描，未更新D1目录中的CSV或图片。

## 1. 阶段E统一命名

- `Kp_pos`：方案B外位置控制器比例增益，单位1/s；对应D1控制器代码中的`kp_per_s`。
- `KD_dc`：方案D二阶黑箱从位置目标到实际位置的直流增益，无量纲；对应D1历史公式中的$K_p$或代码中的`closed_loop_gain`。

阶段E全文只用这两个名字区分控制器增益和对象直流增益，不覆盖D1历史文件。

## 2. 代码版本

基线重算使用D1当前文件，SHA-256来自`results/d1_baseline_provenance.json`：

| 文件 | SHA-256 |
|---|---|
| `src/model_v01.py` | `0677415aba3a0a83198273541066936c26aeba2b69620acc4ef6bf4c253ef54f` |
| `src/controllers.py` | `6a03e515c1f47535b152d84d06a6b96ddb5ace8fe338f09b9c367faeeba39f13` |
| `src/metrics.py` | `a2c5a548dcb11b281b87f161e68e65d6961268dddc257fe8c46687b4fdff42a8` |
| `tests/test_model_v01.py` | `6dc520e4cadcb3645f68e28a18311c74f4c923e2f36656935613334da09f6150` |
| `scripts/run_corrected_baselines.py` | `160a9358ca7f0b237be7e5f7fcb347be7dbba1031409f0e0853f3e496d7790ef` |

## 3. 测试状态

2026-07-15执行：

```powershell
python -m unittest discover -s tests -p "test_model_v01.py" -v
```

结果：15/15通过，运行时间0.646 s。该测试只访问D1离线代码，安全守卫反向测试仍会拒绝`simulation_only=false`或`hardware_disabled=false`。

## 4. 默认基线输入

- 积分周期：1 ms；
- 随机种子：`20260714`；
- 轨迹：点到点五次S曲线、恒速扫描、往返五次S曲线；
- B控制器：P、PI_AW、PI_INV_FF_AW、PID_INV_FF_AW，共12条；
- D：同3条轨迹，共3条；
- 合计：15条，不含任何敏感性扫描。

完整`BModelParams`、`DModelParams`和控制器参数逐字段写入`results/d1_baseline_provenance.json`。关键默认值包括：B的$K_v=1$、$\tau_v=20$ ms、$d_y=d_u=1$拍、显式零测量/命令历史；D的`KD_dc=1`、$\omega_n=2\pi\times20$ rad/s、$\zeta=0.9$、$d_u=1$拍、显式零目标历史。所有值仍是H/R离线参数。

## 5. 修补前后产物关系

| 产物 | 状态 | 解释 |
|---|---|---|
| D1 `results/baseline_metrics.csv` | 未更新 | 保留D1修补前生成值，不能再称当前代码的逐位复现结果 |
| D1 `results/sensitivity_metrics.csv` | 未运行、未更新 | 阶段E禁止重跑敏感性扫描 |
| D1 `figures/` | 未更新 | 图片仍对应修补前基线/扫描产物 |
| E `results/d1_baseline_regenerated.csv` | 已生成 | 当前D1修补代码的15条默认基线 |
| E `results/d1_baseline_comparison.csv` | 已生成 | 当前15条与D1旧CSV逐字段差异 |
| E `results/d1_baseline_provenance.json` | 已生成 | 代码哈希、测试、种子、参数和产物状态 |

逐字段比较得到：15条均成功重算；文本字段无变化；120个数值字段存在非零差异。差异来自D1修补后显式零命令历史取代首拍`constrained[0]`重复，含噪B模型的首拍对象输入因而改变。D正向点到点五项核心指标不变；B/D正向点到点五项核心指标合计最大绝对差为1.0 μm。不同单位字段不能用一个总最大值排名；例如往返滞回面积单位是μm²，必须单独解释。

## 6. 当前采用规则

1. 讨论D1修补后代码行为时，使用阶段E重算CSV和provenance manifest。
2. 引用D1原报告时明确它是修补前生成物，不把旧表与新表拼接。
3. 不因本次版本核查改写D1目录；阶段E以旁路清单保留可追溯性。
4. 这些基线仍是H/R离线模型内部比较，不是台架参数或性能。

