# Motion Acquisition V3.0 开发与数据格式

## 模块

```text
motion_acquisition.control.models       # 不可变轴/控制器/任务/命令模型
motion_acquisition.control.protocol     # 版本化 JSON-safe envelope
motion_acquisition.control.adapter      # 共享适配器能力与 Protocol
motion_acquisition.control.service      # PC 侧序号、校验、超时和观察器
motion_acquisition.control.simulator    # 唯一 V3 适配器
motion_acquisition.control.recorder     # 控制证据和采集哈希关联
motion_acquisition.control.analysis     # 流式描述性控制分析
desktop.control_worker                  # 独立 Qt 工作线程
desktop.control_workspace               # 增量三轴 GUI
```

`control` 与两个 `desktop.control_*` 模块不得导入 `canopen_client`、`SdoSampleSource`、python-can、serial、socket 或真实传输。V2 的 `CanopenSdoClient` 保持 SDO upload-only。

## 控制记录

每个目录包含：

| 文件 | 内容 |
|---|---|
| `control_manifest.json` | 身份、outcome、控制器能力、轴配置、统计、采集关联和证据边界 |
| `commands.jsonl` | PC 提交的完整命令 |
| `acknowledgements.jsonl` | 接受/拒绝确认 |
| `feedback.jsonl` | X/Y/Z 完整快照 |
| `control_events.jsonl` | 控制器和 PC 通信事件 |
| `tasks.jsonl` | 完整任务定义 |
| `control.log` | 人可读日志 |
| `checksums.sha256` | 上述八类终态文件的 SHA-256 |

终态只有 `completed`、`stopped`、`error`；正在记录时为 `in_progress`。文件逐条 flush，终态 manifest 使用临时文件替换。

## 采集关联

`control_manifest.json.acquisition_links[]` 保存：

```json
{
  "run_id": "run-...",
  "experiment_name": "...",
  "session_relative_path": "projects/.../run-...",
  "csv_relative_path": "projects/.../run-.../samples.csv",
  "csv_sha256": "64 hex",
  "outcome": "completed"
}
```

关联必须位于同一用户数据根目录内，解析时拒绝目录逃逸。建立关联和验收后都计算 CSV SHA-256；不写源目录。

## 控制分析定义

- `duration_s`：首末反馈的模拟控制器单调时间差；
- 每轴位置最小/最大/范围：所有反馈帧的 raw SIL position；
- 每轴累计位移：相邻反馈位置差绝对值之和；
- 峰值速度：反馈速度绝对值最大值；
- 故障反馈帧：该轴 `fault_code` 非空的反馈帧数；
- 命令接受/拒绝、任务和事件：对应 JSONL 条目计数。

输出为 `control_analysis_summary.json/csv/md`。它只做描述性计算，不进行物理精度、同步性或统计推断。

## V2 兼容

- 不修改 schema-1.0 采集 manifest 已有字段语义；
- 不修改 16 列 `samples.csv`；
- 不向 V2 会话写控制字段；
- 项目、预设、事件、换算、标定、实验组、影子、研究输出和历史加载路径保持原实现。

## 测试

```powershell
$env:QT_QPA_PLATFORM = "offscreen"
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m compileall -q src
.\.venv\Scripts\python.exe -m pip check
```

持久 SIL 冒烟脚本必须明确输出新目录，且只使用 `SimulatedControllerAdapter`。
