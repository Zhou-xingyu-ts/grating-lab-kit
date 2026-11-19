# Motion Acquisition V1.0 开发与数据格式说明

## 1. 运行架构

```text
PySide6 MainWindow
  ├─ long-lived StationWorker QThread
  │    ├─ DemoSampleSource
  │    └─ SdoSampleSource -> CanopenSdoClient -> SDO upload only
  ├─ AcquisitionService
  │    ├─ StopToken(threading.Event)
  │    ├─ SessionRecorder
  │    └─ EventBuffer
  ├─ short-lived scan / analysis / delivery QThreads
  └─ replay and bounded display copies
```

采集循环逐条调用 reader 的 `read(ObjectSpec) -> ReadResult`，逐条写入 `SessionRecorder`。GUI 接收的是限频、有界、可丢弃的显示副本，不参与 CSV 权威保存。

## 2. 主要扩展接口

- 新增只读数据源：实现现有 `SdoReader` 的 `read` 契约，在 `StationWorker` 中显式注册连接生命周期。
- 新增对象：在 `object_dictionary.py` 增加地址、类型和解释规则，保持 CSV 字段不变。
- 新增分析：读取保存后的 CSV，使用 `StopToken`，保持流式聚合和有界绘图序列。
- 新增导出：只读会话文件，写到用户选择的新目标，不修改源会话。

未来 PDO、EtherCAT、外部传感器只能在具备真实硬件证据和独立安全放行后作为新的只读 source 实现。本 V1.0 没有这些实现，也没有为它们开放设备写控制。

## 3. 用户配置

- `settings.json`：数据根目录、只读连接默认值、周期、时长、对象和当前项目 ID。
- `projects.json`：稳定项目 ID 与项目快捷信息。
- `presets.json`：采集对象、周期、时长和实验记录预设。
- `conversions.json`：可选的未验证线性换算。

这些文件位于当前 Windows 用户的 Qt `AppConfigLocation`，使用临时文件后原子替换。

## 4. `samples.csv`

固定 16 列：

```text
wall_time_iso,monotonic_s,elapsed_s,sample_seq,node_id,
object_key,object_name,index,subindex,raw_hex,raw_value,
parsed_value,unit,read_status,error_message,attempts
```

每行对应一个对象的一次读取结果；同一 `sample_seq` 可以有多个对象行和部分失败。V1.0 不增加物理换算列。

## 5. `manifest.json`

schema 继续为 `1.0`。已有字段含义不变，V1.0 新字段仍是可选项：

- `project`：本次项目完整快照；
- `integrity`：SHA-256 算法、ledger 文件、受保护文件和生成时间；
- `files.checksums`：`checksums.sha256`。

V0.4/V0.5 的 `experiment`、`preset`、`conversion`、`events_summary` 保持兼容。

## 6. `events.jsonl`

每行一个 UTF-8 JSON 对象：

```text
event_id,event_type,label,elapsed_s,wall_time,sample_seq,note
```

时间是上位机人工点击时间，不是驱动器同步时间。文件按点击顺序追加并逐事件刷新。

## 7. `checksums.sha256`

终端会话先原子写入最终 manifest，再为以下存在的文件生成 SHA-256：

- `samples.csv`
- `manifest.json`
- `acquisition.log`
- `config_snapshot.yaml`
- `events.jsonl`

ledger 不对自身做递归校验。旧会话没有 ledger 时状态为 `not_available`，仍可兼容加载；hash 不一致时状态为 `mismatch`。

## 8. 异常恢复

会话打开时先写 `outcome=in_progress` manifest。程序异常终止后：

1. CSV、日志、配置和已刷新事件保持原样；
2. 完整性服务识别为 `incomplete`；
3. 用户在 GUI 中明确恢复；
4. manifest 原子改为 `recovered_incomplete`；
5. 记录恢复时间、保留的记录/轮次数并生成新 ledger；
6. 不修改 `samples.csv`。

## 9. V1.0 交付包

交付包使用临时 `.part` 文件生成，成功后才原子替换目标 ZIP。包含：

- 一个或多个源会话；
- 每会话完整性报告；
- 每会话 JSON/CSV/Markdown 分析摘要；
- bundle manifest 和说明；
- 包内全部其他文件的 SHA-256 ledger。

取消或失败会删除临时 `.part`，源会话始终只读。

## 10. 测试与安全

- 单元/GUI 测试使用 Demo、fake reader 和临时目录。
- 历史兼容测试读取旧 CSV，不修改源文件。
- `test_no_write_api.py` 扫描禁止的写调用和桌面控件。
- 发布冒烟必须从 onedir EXE 启动，默认 Demo，并如实区分离线、硬件和第二电脑证据。
