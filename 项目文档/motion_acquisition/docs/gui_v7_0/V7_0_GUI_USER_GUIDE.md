# Motion Acquisition V7.0 GUI 使用教程

## 启动

发布目录中双击 `MotionAcquisition.exe`，不需要命令行。

V7 的 GUI 操作延续 V6：

- Demo 采集、停止、保存；
- PCAN/CANopen 真实只读采集入口（仅在已有现场条件时使用）；
- 采集预设、事件、未验证 counts 换算；
- 会话浏览、回放、分析与比较；
- 三轴控制工作区的软件在环模式。

## 用 JSON 一次填充采集信息

进入“实时采集”，点击“导入实验 JSON…”，选择实验定义文件。软件会一次
填充项目、预设、counts 换算、证据标定、实验组、实验名称、操作者、轴、
方向、目标位移、速度、Accel/Decel/Jerk、备注、采集对象、周期、时长和
数据目录。项目等上下文既可填写稳定 ID，也可填写当前用户配置中的唯一名称；
不存在或重名时会拒绝导入并指出对应项。

点击“保存模板…”可以随时生成新模板。发布目录也自带
`experiment_definition.template.json`。建议每次复制模板另存，以实验条件、
方向和重复序号命名，再导入；JSON 中的运动参数仍然只是采集记录，不会发送
给驱动器，实际运动仍由 Servo Studio 完成。

## 三轴控制工作区

当前只允许选择 Python SIL 或 native C reference。它们都不访问真实
驱动器。连接后先查看能力：

- `simulated` 应为 true；
- `real_hardware_access` 应为 false；
- `device_write_capability` 应为 false；
- responsibility 显示当前 fixture 的 loop/feedback/command mode。

软件在环可测试 enable、setpoint、stop、protective-stop、状态和事件。
这些操作只作用于虚拟控制器，不是现场运动证明。

## H743 固件文件

发布目录的 `MotionControllerFirmwareV7` 含 ELF/HEX/BIN/map/source/docs。
普通 GUI 用户不要烧录这些文件。它们是未来上板候选，所有物理端口
默认 unavailable，必须先执行 `V7_0_HARDWARE_BRINGUP_CHECKLIST.md`。

## 数据兼容

V7 不改变 `samples.csv`、manifest 的既有字段含义或历史会话目录。
分析/回放不得修改原始 CSV。
