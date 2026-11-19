# V0.4 Build 与本机离线验收记录

## 唯一目标

把 V0.3 只读采集与回放工具扩展为实验数据管理、比较和基础分析工作站，不改变采集线程、16列 CSV 或无写控制边界。

## 计算定义

- 实际采样频率：`(有效采样轮数-1)/(最后轮次 elapsed_s-第一轮次 elapsed_s)`。
- 持续时间：最后轮次与第一轮次 `elapsed_s` 之差。
- 对象统计：仅使用 `read_status=ok` 且 `parsed_value` 存在的行；标准差为总体标准差。
- 跟随误差：最大绝对值、算术平均值、`sqrt(mean(x^2))` RMS。
- 实际位置范围：`max-min`。
- 速度峰值：`max(abs(x))`。
- 失败读取：CSV 中 `read_status != ok` 的记录数。
- deadline miss：读取 manifest 中采集线程形成的 `statistics.deadline_miss_count`；CSV-only 显示“不具备该数据”。

## 兼容与安全

- manifest schema 保持 1.0，V0.4 只增加可选 `experiment` 对象。
- 分析逐行读取并只保留在线聚合量和有界曲线点；任务可取消。
- 三种摘要原子写入，不修改 `samples.csv`。
- 不访问 CAN/USB，不包含任何设备写入或运动功能。

## 2026-07-22 本机离线结果

- 兼容审计：18 个既有会话全部识别和分析，包括14组旧 CSV/sidecar、3个 completed 原生会话和1个 stopped 原生会话；合计217,084条记录，源 CSV SHA-256 全部不变。
- 自动化：最终全量 `64 passed in 9.91s`；`pip check` 无损坏依赖。
- 成品 GUI：从源码目录外启动，Demo 完成50轮/350行/49.88 Hz/0失败/0 deadline miss；后台扫描识别4个测试根目录会话；单会话分析、三种导出、打开回放和播放均通过。
- 发布目录：`release/MotionAcquisition`，377个文件，170,037,644 bytes（162.16 MiB）。
- EXE SHA-256：`C74C45C09F046F5332D4E759501E220763E48B2C0F652ABDD6F276F4F82C07BF`。
- 路径检查：无固定用户名、构建盘符、源码路径、旧网页和历史采样数据夹带。

V0.4 达到当前电脑离线验收；真实 PCAN/运动会话和第二台 Windows 电脑部署仍延期，未形成相关实测结论。
