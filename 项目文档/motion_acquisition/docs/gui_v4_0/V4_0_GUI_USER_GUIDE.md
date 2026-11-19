# Motion Acquisition V4.0 GUI 使用教程

## 1. 启动

复制完整 `MotionAcquisition` 发布目录，双击：

```text
MotionAcquisition.exe
```

不要单独复制 EXE。程序默认不连接 CAN 或任何硬件。

## 2. 三轴软件在环

打开“三轴控制（SIL）”。

### 选择传输

- **本机跨进程 STM32 边界 SIL**：推荐。程序启动独立 `MotionAcquisitionSilController.exe`，用本机回环协议模拟未来 PC/STM32 边界。
- **进程内单元 SIL**：用于快速功能与故障注入检查。

两种方式都没有真实硬件访问。

### 选择职责档案

- **方案 A**：驱动器位置环，STM32 管三轴任务/轨迹/同步；
- **方案 B**：STM32 位置环，反馈由 STM32 直接取得；
- **方案 C 示例**：上位机看到 velocity 命令接口。

档案是软件接口验证，不代表现场已经具备相应硬件能力。

### 连接和记录

1. 输入记录名称；
2. 可选关联一个 V1～V3 采集会话；
3. 选择传输和职责档案；
4. 点击“连接模拟控制器并开始记录”；
5. 查看协商能力：loop owner、feedback source、drive mode、transport、状态率和控制器周期配置。

控制器周期显示的是 SIL 配置，不是硬件实测。

## 3. 方案 A/B 操作

方案 A/B 提供 position 命令接口：

1. 依次模拟使能和回零；
2. 输入目标与速度，执行绝对/相对定位；
3. 可编辑三轴同步位置任务；
4. 观察 position/velocity 的 source 和 validity；
5. 查看 ACK 和 command lifecycle。

全部动作只改变软件模型。

## 4. 方案 C 操作

方案 C 示例提供 velocity 命令接口：

1. 模拟使能轴；
2. 输入速度；
3. 点击“发送 velocity”；
4. 不可用的回零、位置任务等按钮会按能力自动禁用。

这说明 GUI/服务可随接口变化，不证明任何驱动器已配置速度模式。

## 5. 停止语义

- “请求保护性停止（非安全功能）”只发送普通控制器请求；
- 它不是物理急停、STO 或安全继电器；
- physical E-stop/STO 未知时界面显示 unknown；
- “停止采集”仍只停止数据读取，不停止电机。

## 6. 保存和分析

- “完成并保存”：control outcome 为 completed；
- “停止并保存”：control outcome 为 stopped；
- 通信/进程错误：control outcome 为 error。

每个 V4 控制会话保存：

```text
control_manifest.json
commands.jsonl
acknowledgements.jsonl
command_statuses.jsonl
feedback.jsonl
control_events.jsonl
tasks.jsonl
control.log
checksums.sha256
```

点击“分析最近控制记录”会生成 JSON/CSV/Markdown 描述性摘要，不修改关联的 `samples.csv`。

## 7. 现有采集与研究功能

V1～V3 的项目、Demo/SDO upload 采集、事件、回放、完整性、分析、比较、标定、实验组和离线算法保持原操作。真实 CANopen 页面仍只有 SDO upload；本轮没有新增任何真实写入。

## 8. 不能从 V4 得出的结论

V4 本机软件在环不能证明：

- 真实 STM32 通信和周期；
- 三轴同步、轨迹、精度或稳定性；
- 驱动器模式和对象字典；
- 物理反馈来源、单位或标定；
- 急停、STO、制动、限位和安全链；
- 第二台 Windows 电脑部署。
