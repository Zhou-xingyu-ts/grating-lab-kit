# Motion Acquisition V2.0 开发与数据格式

## 1. 稳定架构

```text
GUI
├─ StationWorker → AcquisitionService → SessionRecorder → authoritative files
├─ AnalysisWorker / DeliveryWorker
└─ ResearchTaskWorker
   ├─ data_quality
   ├─ research_analysis
   └─ algorithm_runs
```

- 采集循环与 CSV 写入不经过 GUI queued signal。
- 停止使用 `threading.Event` 封装的 `StopToken`，循环逐条检查。
- GUI 仅接收 10～20 Hz、有界的显示副本。
- 研究任务使用独立、可取消 QThread，不持有完整历史数据。
- 真实访问点仍是 `CanopenSdoClient`，只公开 SDO upload。

## 2. 权威会话格式

`samples.csv` 保持原 16 列，列名和含义不变：

```text
wall_time_iso, monotonic_s,elapsed_s,sample_seq,node_id,
object_key,object_name,index,subindex,raw_hex,raw_value,
parsed_value,unit,read_status,error_message,attempts
```

V2.0 不在 CSV 中增加物理值、设备时间戳、标定值、质量标签或影子输出。

`manifest.json` schema 继续是 `1.0`。新增字段均可选：

- `time_semantics`：主机时钟、轮次范围、同步/硬实时否定和限制；
- `calibration`：完整 `CalibrationProfile` 快照；
- `experiment_group`：所选 `ExperimentGroup` 快照；
- `shadow`：影子参数快照；
- `files.shadow_outputs`：启用影子时的 JSONL 文件名。

安全边界增加：

```json
{
  "raw_can_write_enabled": false,
  "shadow_device_write_capability": false
}
```

旧 manifest 缺少这些字段时按 `None` 或旧主机时间语义加载。

## 3. 标定配置

每用户 `calibrations.json` 使用 schema `2.0`。配置包含：

- 稳定 `calibration_id`；
- 线性仿射 `scale/offset`；
- 原始参考点和可选不确定度；
- `CalibrationEvidence` 来源、时间、可追溯性和附件引用；
- `draft` 或 `externally_referenced`；
- RMS/最大残差；
- 分辨率、换算、测量精度、重复定位、绝对定位五个独立 claim；
- 创建/更新时间和解释边界。

只有每个参考点都链接到可追溯 `external_reference` 时才会成为 `externally_referenced`。

## 4. 实验组

每用户 `experiment_groups.json` 使用 schema `2.0`。每个组含 2～20 个 `GroupSessionReference`：

- 数据根目录相对 `relative_csv`；
- 源 CSV SHA-256；
- run ID、label、role、repeat index；
- 方向、速度、加速度和备注。

评价前重新验证路径不逃逸数据根目录且 SHA-256 不变。

## 5. 数据质量输出

目录中包含：

- `data_quality_report.json`
- `data_quality_report.csv`
- `data_quality_report.md`

JSON 的 `result_class` 是 `direct_data_quality_evidence`，并保存源 CSV 哈希、显式/推断时间语义、对象覆盖、指标、问题和限制。

## 6. 成组评价输出

`group_evaluation.json/.csv/.md` 包含：

- 实验组完整快照；
- 每个会话的直接指标和质量状态；
- aggregate、方向/条件摘要；
- 可选标定换算及证据等级；
- reproducibility fingerprint；
- “不是定位精度”的解释边界。

## 7. 算法运行目录

```text
<run_id>/
  run_spec.json
  result.json
  series.csv
  run_manifest.json
  checksums.sha256
```

运行先写唯一 `.part` 目录，完成后原子发布。`run_spec` 固定源路径/哈希、参数、算法版本、代码指纹和标定快照。`run_manifest` 声明离线结果、raw 未修改和 device-write=false。取消或失败不保留可误用的正式目录。

## 8. 影子输出

启用实时影子时，worker 在每个成功 `following_error` 后写一行 `shadow_outputs.jsonl`。典型字段包括 sample sequence、时间、误差、P/I/D 分量、建议输出、限幅和输出单位。

`ShadowEngine` 只接收 `AcquisitionRecord`，没有 reader、CAN client 或 device 引用。常量 `DEVICE_WRITE_CAPABILITY` 为 false；完整性检查把任何相反 manifest 声明视为错误。

## 9. 未来只读数据源边界

`readonly_source_contract.py` 定义：

- `ReadOnlySourceCapabilities`：source kind、clock domain、标称频率、源时间戳、同步/触发证据和 write=false；
- `TimestampedReadOnlyBatch`：记录、主机接收时间、可选源时间和不确定度；
- `TimestampedReadOnlySource`：只有 `open/read_batch/close`。

未来 PDO、EtherCAT 或外部传感器适配器必须通过该边界声明时间证据。V2.0 没有这些实现，也不把接口存在写成现场可用。

## 10. 兼容与发布约束

- V0.2～V1.0 会话、legacy sidecar manifest 和 CSV-only 输入继续支持；
- 缺事件、checksum、project、conversion、calibration、group、shadow 或 time semantics 都不是格式错误；
- 研究输出、回放和分析不得修改源 CSV；
- per-user 设置由 Qt `AppConfigLocation` 决定，不允许固定用户名/盘符/源码路径；
- 发布包排除旧网页和历史会话；
- 静态无写测试扫描 SDO download、NMT/PDO、send 和运动控件。
