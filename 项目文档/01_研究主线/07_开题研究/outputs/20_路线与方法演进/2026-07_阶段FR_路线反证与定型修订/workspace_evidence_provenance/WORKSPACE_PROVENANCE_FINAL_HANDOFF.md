# 阶段 FR 工作区全域证据溯源最终交接

## 1. 交接结论

本轮已把整个 `D:\清华\清华-科研横向\光栅尺运动控制调研\` 作为技术证据范围，完成文件地图、重复组、60项主张、版本冲突、缺失一级证据和阶段FR定点修订。结论不是新架构选择，也不是路线冻结：

> 当前最强事实链是 2026-07-12 商业 Ri0040—DINGS DS-BVM 单轴内部位置闭环，Servo Studio 通过 USB/COM 发出低速相对位置运动，`motion_acquisition` 通过 USB-CAN 对 Node 1、1 Mbps 做 SDO upload；CSV、manifest、配置、源码和现场记录可以互相对应。STM32只有历史多版本静态/编译资产，没有主站—DS-BVM—实物运行闭环；EtherCAT有当前物理口、系列ESI和历史GM1静态资料，但没有当前设备匹配、扫描或运行；自研尺一级资料未找到；三轴只有师兄历史平台的部分图像、数据和论文声明，不能迁移到当前项目。

这组结果支持继续把商业单轴作为近期证据基座，但不替用户选择CANopen、EtherCAT、其他控制器或三轴路线。

## 2. 已完成范围

| 项目 | 完成结果 |
|---|---|
| 全工作区元数据清点 | 排除非证据缓存/依赖后，2,180文件、599,692,409 bytes，覆盖全部顶层目录 |
| 技术分类 | 14类：当前台架、光栅/编码器、DS、采集、运行数据、CANopen、STM32、EtherCAT、三轴、自研尺、论文/仿真、阶段审计、归档、其他 |
| 重复检测 | SHA-256得到219重复组、700个重复文件；重复手册/照片/CANopenNode库不重复计证据 |
| 原始文档读取 | 当前记录、CSV/JSON、源码/IOC/IDE元数据、系列ESI、现场图、师兄论文/PPT/notebook/GM1代码 |
| 关键主张 | 60/60均有指定枚举裁决或`NOT_FOUND` |
| 版本冲突 | 14项指定冲突 + 8项额外状态冲突均单独登记 |
| 缺失证据 | 34项一级缺口，按当前台架、采集、STM32、EtherCAT、自研尺/三轴分组 |
| FR定点更新 | 只修改四份允许文件；不修改阶段F原文件、源码、配置、数据、手册和归档 |

## 3. 最重要的原始证据

1. `motion_acquisition/logs/motion_sdo_20260712_195923.csv` 及 manifest：7对象、20 ms请求、1,500轮、10,500次读取、0失败，实际约50 Hz；USB负责运动，CAN侧只读。
2. `motion_acquisition/logs/motion_sdo_20260712_203414.csv` 及 manifest：3核心对象、5 ms请求、11,986轮，实际约199.75 Hz；处理容量字段约229.74 Hz，35次处理时间超过目标周期。
3. `motion_acquisition/src/motion_acquisition/acquisition.py`：同轮所有对象共用轮次开始时间戳，但对象依次SDO读取，因此不是同步快照。
4. `motion_acquisition/src/motion_acquisition/sampling_stats.py`：`estimated_rate_hz=1/mean_cycle_s`，表示轮次处理容量，不是实际调度频率。
5. 当前接口板视觉材料：DINGS板上ECAT IN/OUT与CAN口可见，当前STM32芯片可见为H743IIT6；均不证明固件或总线运行。
6. 遗留H743/H750源码：有通用CANopenNode节点/SDO client函数和不同timer/RTOS变体；RPDO映射数为0、TPDO仅本地0x2001、1006为0；无DS对象、NMT Master、CiA402或三轴调度。
7. 历史IDE记录：H743工程曾成功生成ELF（0 errors/0 warnings）；Debug launch存在，但无烧录成功、Flash校验或运行输出。
8. DINGS系列ESI：Profile402、固定PDO和DC-Synchron/FreeRun均存在；当前设备ProductCode/Revision匹配尚未找到。
9. 师兄论文/PPT/旧数据：1 ms/333 μs/53.3%/±0.27 μm/三轴需分项裁决；PPT与论文同源，333 μs/53.3%是计算模型，±0.27有13值历史数据但缺完整仪器原始导出，三轴无匹配STM32运行链。

## 4. 对四条路线事实的最终分层

### 当前商业单轴

- `RUNTIME_SUPPORTED`：USB/COM运动、Node 1/1 Mbps SDO upload、7对象约50 Hz运动记录、3对象约200 Hz、真实往返/参数阶梯。
- `PARTIALLY_SUPPORTED`：Ri完整后缀、DS精确型号、模式号、反馈参数、安全配置、总行程、物理单位和正式重复定位。
- 路线含义：可作为近期基线；不可写成精度、安全或时间同步已完成。

### STM32/CANopen

- 当前视觉：H743IIT6硬件存在。
- 历史静态：H743/H750工程、FDCAN、FreeRTOS、CANopenNode、SDO client函数、一次H743成功编译。
- `NOT_FOUND/CONTRADICTED`：NMT Master、有效SYNC producer、DS-BVM PDO映射、CiA402、三轴调度、编码器采集、PID/前馈/LQR固件、烧录成功和实物运行。
- 路线含义：仍是条件候选，但“遗留资产可直接复用”的强度下调。

### EtherCAT/GM1/其他控制器

- 当前视觉：DINGS接口板ECAT IN/OUT可见，资料称GM1/端口可见。
- 系列能力：工作区有DINGS系列ESI，含Profile402/PDO/DC。
- 历史静态：GM1 Modbus TCP上位机代码有ECAT reset和2D目标/使能接口。
- `NOT_FOUND`：当前DS identity匹配、GM1工程、ESI导入、扫描、PDO/DC和现场运行；可借控制器/工业PC承诺。
- 路线含义：EtherCAT从未被实际排除，只是未实施/未放行；静态资产增强不等于优先级自动上升。

### 自研尺与三轴

- 自研尺：规格、样机、电气/频率、读数代码、标定和交付日期均未找到一级材料。
- 三轴：师兄历史平台有设计/爆炸/实物图、论文应用和旧定位数据；当前三轴CAD/BOM/装配、当前需求指标、协调多轴运行仍未找到。
- 路线含义：二者保持独立条件分支，不能串成商业单轴之后的必经路线。

## 5. 对阶段 FR 的实际修订

已定点更新：

- `../01_ROUTE_CLAIM_EVIDENCE_LEDGER.md`：补原始证据、当前/历史和静态/运行分层。
- `../02_CONTROLLER_AND_BUS_ARCHITECTURE_COMPARISON.md`：校正A/G、B、C/D资产与缺口，不改变A—G候选集合。
- `../05_REVISED_PROVISIONAL_RESEARCH_LINE.md`：保留暂定商业单轴主线，补采样时间、参数敏感、STM32/EtherCAT/三轴/自研尺边界。
- `../PHASE_FR_FINAL_HANDOFF.md`：把“未扫描全工作区”修正为已完成全域溯源，增加本子目录输出和完成自检。

路线排序仍为：A/G当前证据工作；B/C/D并列条件评估；E/F外位置环当前不进入。变化在证据基础，不是自动选型。

## 6. 本目录输出

本目录最终只包含以下7份Markdown：

1. `00_WORKSPACE_TECHNICAL_INVENTORY.md`
2. `01_WORKSPACE_PRIMARY_EVIDENCE_MAP.md`
3. `02_KEY_CLAIM_PROVENANCE_LEDGER.md`
4. `03_VERSION_AND_CONTRADICTION_REGISTER.md`
5. `04_MISSING_PRIMARY_EVIDENCE.md`
6. `05_FR_EVIDENCE_PATCH_DECISIONS.md`
7. `WORKSPACE_PROVENANCE_FINAL_HANDOFF.md`

## 7. 明确未做事项

- 未建立新模型、未新建研究阶段、未重新选择路线、未扩大仿真范围。
- 未运行Python、测试、构建、编译、烧录、仿真、固件或实验脚本。
- 未连接CAN、串口、USB、EtherCAT或任何硬件，未发送运动命令。
- 未搜索网络文献，未采购或自动选型。
- 未修改任何源码、配置、CSV、JSON、图片、PDF、PPT、历史工程或归档。
- 未把CANopen/EtherCAT写控制、三轴、自研尺或外位置环标记为现场放行。
- 未关闭STO、限位、制动、Hall等当前安全缺口。

## 8. 完成标准自检

| 标准 | 结果 | 证据 |
|---|---|---|
| 整个根目录技术文件地图 | 通过 | `00_WORKSPACE_TECHNICAL_INVENTORY.md` |
| 60项主张均裁决 | 通过 | `02_KEY_CLAIM_PROVENANCE_LEDGER.md`，ID 1—60各一次 |
| 原始/派生/手册/论文/源码/运行分开 | 通过 | `01_WORKSPACE_PRIMARY_EVIDENCE_MAP.md` |
| 当前/历史台架分开 | 通过 | 证据图与60项总账的时间/台架列 |
| 重复文件不计独立证据 | 通过 | 219个哈希重复组及独立证据族 |
| 型号/版本冲突登记 | 通过 | `03_VERSION_AND_CONTRADICTION_REGISTER.md` |
| 原FR FACT追原始或降级 | 通过 | `05_FR_EVIDENCE_PATCH_DECISIONS.md`及四份定点更新 |
| STM32静态/编译/烧录/运行分开 | 通过 | 总账29—46项 |
| EtherCAT能力/当前可用分开 | 通过 | 总账47—52项 |
| 自研尺/三轴真实资料量明确 | 通过 | 总账53—60项和缺失证据M27—M34 |
| FR四份文件定点更新 | 通过 | 仅四份目标文件加入/修改证据补丁 |
| 可回答路线依据 | 通过 | 本交接第3—5节 |

## 9. 停止点

阶段FR全域证据溯源与证据修订在此停止。本交接可供开题路线复审使用，但不构成路线最终冻结、现场放行、阶段状态收口或下一研究阶段启动。
