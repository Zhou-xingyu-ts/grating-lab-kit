# V0.5 本机离线构建与验收记录

## 结论

V0.5 核心版通过当前 Windows 电脑离线验收。验收只使用 Demo 数据源；未访问 CAN、USB 或运动硬件，也未执行任何设备写入或运动控制。

## 核心功能

- 采集预设：支持新建、保存、加载、复制、重命名和删除，并将对象、周期、时长及实验记录字段填入现有采集页。预设存于当前用户配置目录，历史会话保存独立快照。
- 人工事件：支持运动开始、运动结束、方向切换、异常和自定义备注。事件按点击顺序逐条刷新到 `events.jsonl`，completed、stopped 和 error 收尾均保留；时间基准明确为上位机人工标记时间。
- 简单换算：支持 `physical_value = scale * raw_counts + offset`，同时显示 raw counts 与带有“未验证换算”标记的换算值。换算不写入 `samples.csv`，会话 manifest 保存完整快照。

## 数据兼容性

- `samples.csv` 保持原有 16 列，仅保存原始值。
- `manifest.json` 继续使用 schema `1.0`，新增可选的 `preset`、`conversion`、`events_summary` 和 `files.events`。
- 新会话增加 UTF-8 JSON Lines 文件 `events.jsonl`；旧会话缺少该文件时按零事件正常加载。
- 持久兼容语料共 19 个 V0.5 以前的会话：14 个 legacy、4 个 completed、1 个 stopped，共 217,434 条记录；全部兼容，审查前后 CSV SHA-256 不变。

## 测试与冒烟

- 全量测试：`73 passed in 17.48s`。
- 依赖检查：`pip check` 返回 `No broken requirements found.`。
- 发布版 GUI Demo 冒烟覆盖：预设、换算显示、自然完成、人工事件、主动停止、会话扫描、回放事件竖线及点击跳转。
- 主动停止会话：1,946 轮、13,622 条记录、49.98 Hz、0 次失败读取、0 次 deadline miss；保留 1 个异常事件，点击后跳转到 `sample_seq=516`。
- 四个发布版冒烟会话均包含 `samples.csv`、`manifest.json`、`acquisition.log`、`config_snapshot.yaml`、`events.jsonl`；分析和回放未修改原始 CSV。

## 发布包

- 目录：`release\MotionAcquisition`
- 文件数：377
- 大小：170,068,263 字节
- EXE SHA-256：`1B8F31561AF7479496BDB1F0AD0018EEAF47B398C3F91FD18622BFA826D1E54D`
- 内容检查：未嵌入会话文件，未发现固定用户名、构建盘符或源码路径。

## 延期验证

- 真实 PCAN/CANopen 只读采集、静止采集和配合 Servo Studio 的运动采集。
- 第二台 Windows 电脑或干净虚拟机上的复制、启动、用户目录隔离及驱动环境验证。

V0.5 没有新增 SDO download、NMT 写入、PDO 配置、控制字写入、运动命令或其他设备写控制路径。
