# Motion Acquisition V9 — 代码地图（干活用）

<!-- last_reviewed: 2026-10-23 -->

> 这份文档不是设计契约。设计契约看 `V9_0_ARCHITECTURE.md`（讲"应该怎样、什么不许做"）。
> **这份讲"代码实际怎么组织、要改一件事该动哪里"。**
>
> 建立日期：2026-09-18。数据来源：`src/` 实际 import 关系与文件规模实测，不是从设计文档反推。
> 下方规模与行数均为 2026-09-18 历史快照，不能作为当前文件规模或测试数量。当前验证和桌面包入口见工程 `CLAUDE.md`。

---

## 1. 一张图

```text
入口层 ────────────────────────────────────────────────────────────────
  pyproject.toml [project.scripts]
    motion-acq            → main.py            CLI：只读 SDO 采集 / 列对象
    motion-acq-desktop    → desktop/__main__.py  主窗口（日常用的就是这个）
    motion-acq-direct     → desktop/direct_control_window.py  Route A 真实直控
                                                            ★唯一能写设备的 GUI
    motion-acq-esi-audit  → routes/esi_audit.py    Route C 的 ESI 只读审计

desktop/  6,384 行 ─── 只依赖下面两层，不被任何层依赖
  main_window.py            2,719 行  ← 单窗口 + 页签装配 + 后台 worker 接线
  ├── control_workspace.py    870 行  控制页签
  ├── research_workspace.py   940 行  研究/分析页签
  ├── route_manager_workspace.py 611 行  路线页签（含「Route B 静止只读采集」按钮）
  ├── direct_control_window.py 573 行  Route A 独立直控窗口（独立入口）
  ├── direct_telemetry.py             直控显示缓存：2000 轮上限，GUI 10 Hz 取快照
  ├── replay_plot.py                  历史双图、缺测断线与完整会话 PNG 导出
  ├── analysis_worker.py / control_worker.py / *_worker.py  Qt 线程包装
  └── ui/…（页签内部小部件）

routes/  4,832 行 ─── 依赖 control；只被 desktop 依赖
  models.py        427 行  RouteId / RouteState / RouteDescriptor / ActiveRoute 协议
  route_a.py       456 行  PCAN 直连 facade（**委托**给 dsbvm_direct，不自己写设备）
  route_b.py     1,309 行  protocol 4.0 主机端：帧、sequence、ACK、boot ID、断线
  route_c.py       435 行  Modbus TCP → GM1
  virtual_dsbvm.py 396 行  虚拟 DS-BVM（离线帧级验证：SDO/abort/PPM/PDO/SYNC）
  gm1_contract.py  642 行  GM1 Modbus 契约校验
  esi_audit.py     581 行  ESI/XML 只读审计（独立 CLI）
  route_manager.py         注册、资源锁、切换、预检
  transport：FramedByteTransport 抽象 + 内存 Fake + 串口（惰性导入）

control/  4,763 行 ─── 尽量不依赖 root；被 routes 和 desktop 依赖
  models.py      1,031 行  控制器语义模型（AxisState/ControllerSnapshot/CommandAck…）
  adapter.py / protocols.py / service.py    ControllerAdapter 契约与执行
  simulator.py   1,218 行  SIL 三轴仿真
  direct.py / direct_service.py            Route A 真实控制流程（Halt→Disable）
  remote_adapter.py 696 行  远程/桥接适配器
  analysis.py       ~200 行  控制会话（events.jsonl）分析  ← 与 root/analysis.py 同名不同物
  recorder.py / profiles.py / sil_process.py / analysis_worker… 

root  53 个平铺模块 / 8,356 行 ─── 几乎不依赖上面任何层（只有 1 处 root → routes）
  这是"业务与数据"层，也是目前最乱的一层。按职责分六组：
  ┌ 采集内核   acquisition.py · acquisition_service.py · acquisition_state.py
  │           config.py · canopen_client.py · csv_logger.py · sampling_stats.py · dry_run.py
  ├ 会话与数据 session.py · session_store.py · session_integrity.py · session_checksums.py
  │           session_browser.py · manifest.py · experiment.py · experiment_definition.py
  │           experiment_groups.py · projects.py · events.py
  ├ 分析与出图 analysis.py · research_analysis.py · data_quality.py · algorithm_runs.py
  │           delivery_export.py
  ├ 换算与对象 object_dictionary.py · calibration.py · conversion.py · statusword.py
  ├ Route A 直控 dsbvm_direct.py（793 行，**全工程唯一允许写设备的地方**）
  └ 基础设施   config.py · presets.py · user_settings.py · route_metadata.py
              route_preflight_records.py · diagnostics.py · product_diagnostics.py
              shadow_mode.py · replay.py · safety_state.py · readonly_source_contract.py
              models.py · errors.py · logging_config.py · main.py · stop_token.py

utils/  18 行  仅 log_timing
```

**依赖方向（实测，无环）：**
`desktop` → `routes` / `control` / root ；`routes` → `control` / root ；`control` → root（仅 3 处）；root → `utils` 与 `routes`（各 1 处）。

---

## 2. 一次采集会话发生了什么

```text
你点「开始采集」
  desktop/main_window.py  ─ 建 config、选数据源 ─┐
                                                ↓
  acquisition_service.AcquisitionService.run()
      ├─ reader = 数据源
      │    ├─ sample_source.DemoSampleSource        （Demo，走 simulated_backend）
      │    ├─ canopen_client.CanopenSdoClient       （真实：PCAN/CANopen SDO upload）
      │    └─ routes/route_b.RouteBReadOnlySource   （Route B：串口 → STM32）
      └─ recorder = session_store.SessionRecorder.for_session(...)
             落盘目录：<data_root>/<年>/<月>/<YYYYMMDD_HHMMSS>_run-<8hex>/
               samples.csv          16 列，**格式是兼容边界，不许改**
               manifest.json        命令、版本、配置摘要、结果、route 元数据
               config_snapshot.yaml 本次生效配置
               acquisition.log      人类可读日志
               events.jsonl         控制会话事件（采集会话通常为空）
             注意：2026-09-17 起**不再生成** checksums.sha256，manifest 无 integrity 段

  会话结束后：
    session_browser.py      把会话列进界面
    session_integrity.py    按路线白名单 + 权限 + 控制日志判定完整性（855 行）
    session_checksums.py    只读校验**历史**会话的旧账本（能力保留，不再生成新的）
```

`data_root` = `%LOCALAPPDATA%\MotionAcquisitionData`（2026-09-18 起；之前落在被 OneDrive 重定向的 Documents 里）。

20261023-ux2 中，Direct 首次从 `load_user_settings().data_root` 取数据根，之后用独立 `direct_settings.json` 记住目录和实验名。直控的 `session_ready(Path)` 通知主窗口刷新并选择会话，旧分析忙时只排队最新基础摘要。`replay_requested(Path)` 进入 `MainWindow.open_session_replay(Path)` 后台加载流程。独立 Direct 使用 `MainWindow(replay_only=True)`，加载成功后在查看窗内索引外部会话；该窗口不保存主窗口设置或连接设备。

显示缓存只消费已有 `on_record`/`on_progress`；曲线缺测断线，读数按采样时刻标记过期。实时与回放分别维护最后成功时间；回放跳转从目标轮次之前的记录重建。`replay_plot.py` 绘制已记录的给定/实际位置与跟随误差，并在独立图场景导出完整 PNG，不更改采集或原始 CSV。

---

## 3. 分析链路

```text
采集会话（samples.csv）  →  analysis.analyze_session()      → AnalysisSummary
                              ↑ 被这些复用：algorithm_runs、data_quality、research_analysis、delivery_export
控制会话（events.jsonl）  →  control/analysis.analyze_control_session() → ControlAnalysis
                              ↑ 被 control_worker 用
出图/导出                →  analysis.export_analysis() / delivery_export.py
```

**两个 `analysis.py` 不是重复代码，是两个数据源的分析器**——但同名，grep 时极易搞错（见 §5 隐患 2）。

`offline_analysis/metrics.py` 是单独离线管线的分段分析（含驻留期超调、缺测对齐），不等于 GUI 的基础摘要。历史报告不会因源码更新自动重算。

`analysis.py` 的比较序列保留缺测断点，限长抽点不会跨缺测连线；统计仍只使用有效记录。`as_dict(include_plot_series=True)` 将缺测值序列化为 JSON null。

---

## 4. "我想改 X，动哪里"

| 我想做的事 | 改这里 | 不要碰 |
|---|---|---|
| 改采集的采样率/对象/时长默认值 | `config.py`、`configs/*.yaml` | 不要改 16 列 CSV 格式 |
| 加一个新采集数据源 | 实现 `readonly_source_contract.TimestampedReadOnlySource`，在 `routes/` 里挂上 | 不要绕过 Route Manager 直接连设备 |
| 改会话目录命名 / 落盘内容 | `session_store.py`（`create_session_paths`） | `data/`、`logs/` 里已有会话只读 |
| 改 manifest 字段 | `manifest.py` | 只能**加**可选字段；旧字段必须继续可读 |
| 改界面某个页签 | `desktop/<对应>_workspace.py` | `main_window.py` 只做装配，别往里塞业务 |
| 改常用/全部视图、会话与回放衔接 | `desktop/main_window.py`、`user_settings.py` | 不让只读历史查看窗口连接设备 |
| 改直控读数、趋势、结果入口 | `desktop/direct_control_window.py`、`desktop/direct_telemetry.py` | 不新增 CAN 读取或改变原始 CSV |
| 改历史双图或保存结果图 | `desktop/replay_plot.py`、`desktop/main_window.py` | 不用计划目标冒充已记录给定值，不写原始数据 |
| 构建当前桌面包 | `version.py`、`packaging/build_desktop.ps1`、`packaging/motion_acquisition_v9.spec` | 不覆盖旧 release，不重建固件 |
| 改 Route B 的帧/ACK/超时语义 | `routes/route_b.py` + protocol 4.0 契约 | 不要升协议版本号 |
| 真的要让电机动 | `dsbvm_direct.py`（Route A）或固件侧 | **任何其它模块加写设备代码都是违规** |
| 改分析出的图/指标 | `analysis.py`（采集）或 `control/analysis.py`（控制） | 别把分析逻辑塞进 desktop |
| 改换算/单位 | `object_dictionary.py`、`conversion.py`、`calibration.py` | `1 count≈0.5 μm` 是工程换算，不是标定 |
| 改 JSON 实验定义 | `experiment_definition.py` + `configs/` | — |

---

## 5. 这份结构目前的问题（已知，别重复踩）

按"会不会真的害到你"排序：

| # | 问题 | 实测证据 | 危害 |
|---|---|---|---|
| **1** | **root 53 个平铺模块没有子包**，六种职责混在一层 | root = 8,356 行 / 53 文件，其中会话相关占 7 个文件 1.8k 行、分析相关占 5 个文件 2.3k 行 | 找一个文件靠猜名字；新人/新 agent 必须读完 `__init__` 才知道结构 |
| **2** | **同名模块三组，语义不同**：`analysis.py`(采集) vs `control/analysis.py`(控制)；`models.py`(root) vs `control/models.py` vs `routes/models.py`；`profiles.py`(root, 设备/轴档案) vs `control/profiles.py`(SIL 能力) | 见 §1 | grep/搜索会串；import 写错不一定报错，可能静默拿错对象 |
| **3** | **`main_window.py` 2,719 行** | 单文件装 5 个页签的装配 + 状态机 + worker 接线 | 改一处界面要在 2700 行里定位 |
| **4** | **死代码**：`session.py` 里 `ExperimentRun` / `StationSession` 无人引用，该文件实际只剩 `new_id()` 一个函数 | 实测零引用 | 增加误用面 |
| **5** | **控制/分析两个 `_atomic_text()` 各写一遍** | `analysis.py` 与 `control/analysis.py` 各有一份 | 小重复，改一处忘另一处 |
| **6** | **旧发布物继续保留，容易误开** | 旧包不会随源码更新；当前桌面修订为 `20261023-ux2`，包与验证入口见工程 `CLAUDE.md` | 使用时核对窗口修订号，不能用旧候选报告代表当前程序 |
| **7** | **`release/` 864 MB / 7 个候选，`build/` 36 个目录** | 实测 | 拖慢全目录搜索；但按规则不许我擅自删 |
| **8** | **没有自动化测试守门** | 无 CI 配置；当前测试结果见工程 `CLAUDE.md`，需运行 `pytest` 才能确认 | 回归靠自觉 |

**已经确认没问题的（值得知道，别去"修"）：**
- 写设备 API 严格收口：全工程 `.sdo.download`、`.nmt.state=`、SDO 写只出现在 `dsbvm_direct.py`（12 处），Route Manager / routes / control 一处都没有。
- 依赖无环。
- `python-can` / `canopen` 只在 `canopen_client.py` 里 import，其余模块都经它。
- 16 列 `samples.csv` 与 protocol 4.0 是兼容边界，没有越界改动。

**2026-09-18 已清理：**
- 删除了 V0.2 遗留的浏览器 UI（`ui_server.py` + `ui/app.js` + `ui/index.html` + `ui/styles.css`，共 24.7 KB）：`motion-acq ui` 子命令、`pyproject` 的 `package-data`、两个 PyInstaller spec 里的 exclude 条目一并移除；`test_v1_portability.py` 新增一条守卫，防止它被复活。
- 根目录三个工作日志退役（见 `AGENTS.md`）。

---

## 6. 维护约定（本文件怎么保持有效）

- 改动 `src/` 的**包结构、入口、或任一层依赖方向**时，同步改本文件 §1/§2/§3。
- 改「我想改 X 动哪里」的实际落点时，改 §4。
- 修掉 §5 里任一项时，删掉那一行（不要留"已修复"痕迹），历史进 git。
- 本文件只描述**事实**；"应该怎样"属于 `V9_0_ARCHITECTURE.md`，"不许怎样"属于 `AGENTS.md`。
