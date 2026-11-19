# Motion Acquisition V4.0 Windows 部署

## 复制

复制整个目录：

```text
release/
  MotionAcquisition/
    MotionAcquisition.exe
    MotionAcquisitionSilController.exe
    _internal/
    docs and release files
```

不要只复制主 EXE 或 helper EXE。

## 启动

双击 `MotionAcquisition.exe`。默认不连接 CAN。

“本机跨进程 STM32 边界 SIL”会由主程序自动启动 helper，无需命令行、CAN 驱动或硬件。Windows 防火墙不应需要开放外部端口；V4 只接受数字回环地址。

## 用户数据

配置使用当前 Windows 用户的 Qt `AppConfigLocation`，数据目录默认为当前用户“文档”下的 `MotionAcquisitionData`。不依赖固定用户名、盘符或源码路径。

## 真实只读采集

只有既有 CANopen SDO upload 采集可能需要外部 PCAN 驱动。V4 没有真实控制器/CAN 写入适配器。

## 新电脑检查

1. 保留完整目录；
2. 双击主 EXE；
3. 确认标题为 V4.0；
4. 在三轴控制页选择跨进程 SIL；
5. 连接 Scheme A，模拟使能/回零/任务/保护性停止并保存；
6. 选择 Scheme C，确认 GUI 按能力显示 velocity；
7. 打开历史采集并回放/分析；
8. 确认用户数据写入该电脑当前用户目录。

当前没有第二台 Windows 实测证据，上述步骤是部署验收方法，不是已完成声明。

## 故障

- helper 启动失败：检查完整目录和杀毒软件隔离记录；
- 连接后 helper 退出：控制会话应以 error 收尾，最后反馈显示 stale/unknown；
- 误删 helper：重新复制完整发布目录；
- CAN 连接失败：与三轴 SIL helper 无关，检查 PCAN 驱动和现场只读配置。

## 安全

- 跨进程 SIL 仍是模拟器；
- 保护性停止请求不是急停/STO；
- 停止采集不停止电机；
- 真实设备访问仍只有 SDO upload；
- 未经现场放行不得把任何 SIL 命令接到真实控制器。
