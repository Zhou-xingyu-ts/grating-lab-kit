# 阶段 B2 交接

## 五条最重要结论

1. 论文设计为 Z 轴光栅全闭环、X/Y 霍尔半闭环；“三轴全光栅尺”被论文列为未来工作，而非既有成果。
2. `h750_demo` 确有静态 1 ms TIM6 + FreeRTOS + CANopenNode 路径，但只证明通用本地节点骨架；没有 DS-BVM CiA 402、三轴 PDO、编码器读取或控制器。
3. 遗留对象字典默认 RPDO 全部无映射/失能，唯一启用 TPDO 映射本地 `0x2001`；`STM32Master` 文件名不能证明主站已实现。
4. 论文 PID/前馈/LQR 结果属于仿真；STM32 固件未发现这些算法。MATLAB 脚本的质量参数 `7.7` 还与论文表 2.6 的 `0.5 kg` 冲突。
5. 论文报告 Z 轴激光干涉仪实验、±1 μm 定位、±0.27 μm 重复定位和 0.5 μm 最小步进，但没有把这些指标与 STM32 1 ms PDO/LQR 建立可追溯因果链，原始数据也不足以独立复核。

## 五个最大缺口

1. 无 1 ms 周期/抖动/总线抓包/三轴负载的实测证据。
2. 无 NMT Master、有效 SDO Client、DS-BVM CiA 402 与 PDO/SYNC 实现。
3. 无光栅/Hall 接入 STM32 的硬件与采样源码。
4. 无固件 PID/LQR/前馈、离散化、限幅和安全状态机。
5. 无带元数据的定位/重复/步进原始数据、分析脚本和固件版本链。

## 输出文件

- `phase_B2_legacy_thesis_firmware/00_LEGACY_SOURCE_INVENTORY.md`
- `phase_B2_legacy_thesis_firmware/01_THESIS_CLAIM_AUDIT.md`
- `phase_B2_legacy_thesis_firmware/02_FIRMWARE_ARCHITECTURE_AUDIT.md`
- `phase_B2_legacy_thesis_firmware/03_TIMER_CANOPEN_PDO_AUDIT.md`
- `phase_B2_legacy_thesis_firmware/04_CONTROL_ALGORITHM_IMPLEMENTATION_AUDIT.md`
- `phase_B2_legacy_thesis_firmware/05_THESIS_CODE_EXPERIMENT_CROSSCHECK.md`
- `phase_B2_legacy_thesis_firmware/06_REUSABLE_ASSETS_AND_MISSING_DEPENDENCIES.md`
- `phase_B2_legacy_thesis_firmware/07_LEGACY_EVIDENCE_GAPS.md`
- `phase_B2_legacy_thesis_firmware/HANDOFF.md`

## 阶段 C 应优先使用的判断

- 把当前台架、DS-BVM 手册和遗留资产三方分列，不让论文/源码替代台架证据；
- 将“DS-BVM 内部闭环基线”与“STM32 外环研究候选”分成近期开题基线和中期研究路线；
- 对 CSV/CST/CSP 候选均显式计入电机侧反馈、换相、主站、PDO/SYNC、周期无官方保证及安全依赖；
- 仅把 `h750_demo` 视为工程骨架可复用，不把控制功能或 1 kHz 性能计为已完成。

