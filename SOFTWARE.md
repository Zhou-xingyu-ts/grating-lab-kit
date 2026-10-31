# 软件来源与验证范围

整理日期：2026-10-31。仓库为可分发的使用包和手册，不替代源码工程，也不携带原始实验数据。

| 交付内容 | 版本/来源 | 本次处理 |
|---|---|---|
| Motion Acquisition | V9.0.0 · 20261023-ux2；原有 `motion_acquisition/release/MotionAcquisition_20261023-ux2/MotionAcquisitionV9` | 原样复制全部 707 文件，399,854,680 字节；不重建、不改原发布物 |
| PEAK 驱动 ZIP | PEAK 官方 `https://www.peak-system.com/quick/DrvSetup`，2026-10-31 下载 | ZIP 180,274,120 字节，含 `PeakOemDrv.exe`，产品 PEAK-Drivers 5.1.5、版本 5.1.5.1208；Windows Authenticode 检查为 Valid，签名方 PEAK - System Technik GmbH；未安装 |
| Servo Studio ZIP | 原有 `DINGS_Servo_Studio_官网软件下载_20220712.zip` | 原样复制 5,756,208 字节；不是新下载版本，未安装 |
| DS-BVM 硬件手册 | 原有官方 `DINGS_DS-BVM-FETC_Hardware_Manual.pdf` | 原样复制，1,562,777 字节 |
| DS-BVS/BVM 参考手册 | 原有官方 `DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf` | 原样复制，7,414,531 字节 |
| Servo Studio 手册 | 原有官方 `DINGS_Servo_Studio_Manual_官网20260601下载版.pdf` | 原样复制，交付名 `DINGS_Servo_Studio_Manual.pdf`，5,849,930 字节 |

## 历史验证（不是本次新运行）

2026-10-23 的主窗口和 Direct 离线检查记录分别在 [desktop smoke](evidence/desktop-smoke-20261023.json) 和 [direct smoke](evidence/direct-smoke-20261023.json)。两者 status=passed，回放、基础摘要、PNG 导出通过，blocked_calls/errors 为空，real_hardware_access=false、device_write_capability=false。原构建信息见 [RELEASE_INFO](evidence/RELEASE_INFO-20261023.txt)。

原工程当前状态记录 368 项离线测试通过；本次整理未重跑这些测试，未运行 EXE smoke、模拟、构建、固件或真实采集。Release 下载包不依赖原来的源码路径或 `.venv`。

历史现场证据是 2026-08-01 Windows→PCAN→DS-BVM 单轴固定往返，结论 PASS_WITH_LIMITATIONS。驱动器读数不等于独立自研尺采集，不得据此宣称精度标定、多轴或硬实时。

## 到实验室后仍须确认

- 新电脑为 Windows 10/11 x64，可安装驱动，有稳定本地可写目录。
- 新下载 PEAK 5.1.5 对现有 XCAN-USB 的识别、PCAN-Basic 加载和只读通信。
- 新电脑人工离线启动/保存/回放，以及获准后的真实采集。
- 当次台架电气、安全、主站排他、节点及波特率与现场动作放行。

宿舍电脑仅作为配置对照：PCANBasic.dll 5.0.1.1131，USB 驱动 5.1.2.20099。工具包未从系统目录抽取 DLL 代替驱动安装。
