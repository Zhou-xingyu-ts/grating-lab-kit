# 位置对象严格语义审计

## 1. 四个核心位置对象

| 对象事实 ID | 对象 | 严格定义 / 来源 | 单位与访问 | PDO 可映射性 | 模式有效性边界 | 标签 | 官方文件完整路径、版本、页码 / 行号、章节 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|---|---|---|
| OBJ-B1-001 | `0x6062 Position Demand Value` | 位置控制器输入；在 profile motion 中由驱动器内部 profile generator 生成 | INTEGER32，RO，`count` | Reference：NO | PPM 中明确由内部轨迹发生器生成；需要位置控制器的模式才有明确控制含义 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.154，§6.2.96，对象表；另见 PDF p.29，§5.3 | `0x6062` 不是外部目标对象，而是位置控制器实际收到的 demand；PPM 下它是轨迹发生器输出 | 不证明它来自传感器，不证明它等于 `0x607A`，不证明在 CSV/CST 下有同等控制意义 |
| OBJ-B1-002 | `0x6063 Internal Position Actual Value` | Reference 只给对象名称与字段表，没有正文语义定义 | INTEGER32，RO，`count` | Reference：NO | 手册未给逐模式有效性说明 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.155，§6.2.97，对象表 | 驱动器公开一个名为 Internal Position Actual Value 的 32 位只读计数量 | 不证明它是“原始编码器计数”“电机侧反馈”“Feedback1”或与 `0x6064` 的差值来源；手册没有定义二者关系 |
| OBJ-B1-003 | `0x6064 Position Actual Value` | 由传感器测得的电机实际位置；未配置传感器时恒为 0 | INTEGER32，RO，`count` | Reference：TxPDO | CST/CSV/CSP 的输出参数均列出；对象章节未声明只在这些模式有效 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.155，§6.2.98；PDF pp.41、44、47，§5.5.1–5.7.1 输出参数表 | `0x6064` 是官方定义的传感器测得实际位置，也是循环同步模式可反馈给控制器的关键位置量 | 不说明双反馈时对应 Feedback1 还是 Feedback2；不证明是独立第二传感器；不证明单位已经换算为 mm |
| OBJ-B1-004 | `0x60F4 Following Error Actual Value` | Reference 只称“实际 Following Error”，未给公式、符号方向或更新时刻 | INTEGER32，RO，`count` | Reference：NO | CSP 章节用 following-error window 判断需求位置与实际位置差是否过大；对象章节未声明全模式有效 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.176，§6.2.141；PDF p.47，§5.7.1 状态说明；PDF p.156，§6.2.99 | `0x60F4` 是驱动器内部给出的跟随误差实际值；`0x6065` 以 Demand 与 Actual 的允许差判断故障 | 不足以断言 `0x60F4 = 0x6062 - 0x6064` 或相反；不证明其取样同步、滤波、饱和及任一传感器绑定 |

## 2. 对象之间能确认的关系

| 关系 ID | 关系结论 | 标签 | 官方文件完整路径、版本、定位 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| OBJ-B1-R01 | PPM：`0x607A Target Position` 进入内部轨迹发生器，生成 `0x6062`，随后进入位置控制器 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.29、154、163，§5.3、§6.2.96、§6.2.114 | 区分了用户目标 `0x607A` 与控制器需求 `0x6062` | 不证明两者在动态过程中相等；不证明外部控制器可直接写 `0x6062` |
| OBJ-B1-R02 | CSP：外部以 `0x607A` 周期给目标，驱动器内部进行位置控制；`0x60B0` 作为目标位置附加偏置 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.45–47、173，§5.7、§6.2.135 | CSP 外部负责轨迹点，驱动器保留位置环；位置偏置有独立对象 | 不证明外部写入的是 `0x6062`；不证明位置反馈一定来自光栅尺 |
| OBJ-B1-R03 | `0x6065 Following Error Window` 以 `Position Actual Value` 与 `Position Demand Value` 的最大允许差为判断依据 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.156，§6.2.99 | Following error 的概念与 demand/actual 差有关 | 不给出 `0x60F4` 的精确算式、符号和内部滤波，因此不能用对象名补齐公式 |
| OBJ-B1-R04 | 双反馈时 Feedback1 服务 voltage/current/speed，Feedback2 服务 position control | E3-GUI | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.16，§4.2 | 双反馈位置环使用 Feedback2 的职责分配是明确的 | Reference 没有把 `0x6063`、`0x6064`、`0x60F4` 明确绑定到 Feedback1/Feedback2；只能提出待验证假设，不能写成对象定义 |
| OBJ-B1-R05 | `0x6063` 与 `0x6064` 不能被当前官方文字解释为两套不同传感器 | H | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.155，§6.2.97–6.2.98；`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_Servo_Studio_Manual.pdf`，VER 1.1，PDF p.16，§4.2 | 手册只分别命名为 internal actual 与 sensor-measured actual | 不允许据此宣称 `6063=电机编码器`、`6064=光栅尺`；需厂商说明或受控实测关闭 |

## 3. 单位、偏置和分辨率对象

| 对象事实 ID | 对象 / 结论 | 标签 | 官方文件完整路径、版本、PDF 页码、章节 | 原文含义的准确转述 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|---|
| OBJ-B1-005 | `0x607A Target Position`：profile 运动时是轨迹发生器输入，无 profile 时直接进入位置控制器；INTEGER32、RW、`count`、RxPDO | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.163，§6.2.114，对象表 | 位置目标的官方入口及两类使用路径 | 不证明其物理单位是 mm；Controlword 还决定绝对/相对和目标生效方式 |
| OBJ-B1-006 | `0x607C Home Offset` 为回零后偏移；INTEGER32、RW、`count`、不可 PDO 映射 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.163，§6.2.115，对象表 | 回零坐标偏移与实时位置目标是不同对象 | 不证明当前系统完成过回零或建立了物理零点 |
| OBJ-B1-007 | `0x60B0 Position Offset` 在 CSP 中给 Target Position 增加附加命令位置；INTEGER32、RW、`count`、RxPDO | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.173，§6.2.135，对象表 | CSP 存在实时位置偏置入口 | 不证明该入口适合直接实现外位置环，也不证明当前 CANopen PDO 已映射它 |
| OBJ-B1-008 | `0x6089/0x608A` 用于缩放位置对象；Reference 仍把核心位置对象的单位列为 count | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.167–168，§6.2.124–6.2.125；PDF pp.154–176 相关对象表 | 固件存在位置量纲/记数法对象，核心对象原始口径是 count | 不提供当前参数值到 mm 的外部可溯源标定；不能仅凭对象存在把 counts 换成物理长度 |
| OBJ-B1-009 | `0x608F Position Encoder Resolution` 包含 increments 与 motor revolution 两个子项；increments 默认 500、允许 400–40000，motor revolution 默认/固定为 1；不可 PDO 映射 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.170，§6.2.130–6.2.130.2，对象表 | 驱动器有编码器分辨率参数，可影响内部 count 与机械转动关系 | 不证明当前值是多少，不适用于未经确认的直线尺 mm 标定，不证明 10000 counts 等于任意固定长度 |
| OBJ-B1-010 | `0x607D Software Position Limit` 同时检查 Actual 和 Demand；两端均为 0 时关闭；Homing 中禁用 | E3-REF | `D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF p.164，§6.2.116 | 位置模式还有软件限位约束，不能只看目标与反馈对象 | 不证明当前限位已设置或足以替代机械/硬件安全限位 |

## 4. Reference Manual 与 EtherCAT ESI 的 PDO 冲突

| 冲突 ID | Reference Manual | 官方 EtherCAT XML | 结论 | 证明了什么 | 没有证明什么 |
|---|---|---|---|---|---|
| OBJ-B1-C01 | `0x6062`、`0x6063`、`0x60F4` 的对象表标为 PDO Mapping `NO`；`0x6064` 标为 TxPDO。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方手册\DINGS_DS-BVS-BVM_Series_Reference_Manual.pdf`，VER 1.0，PDF pp.154–155、176，§6.2.96–6.2.98、§6.2.141 | XML 将 `0x6062`、`0x6063`、`0x6064`、`0x60F4` 全部标为 `<PdoMapping>t</PdoMapping>`；并把 `0x6062` 放入可配置 TxPDO2、`0x60F4` 放入可配置 TxPDO4。证据：`D:\清华\清华-科研横向\光栅尺运动控制调研\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml`，EtherCATInfo 1.1，lines 10389–10426、11380–11390、11748–11784 | 总线/文件版本存在明确差异；CANopen 设计应遵守 Reference 的 CANopen 对象表，EtherCAT 设计可把 XML 当候选 ESI 证据但须先核对设备身份 | 不得据 XML 推断当前 CANopen 上 `0x6062/6063/60F4` 可 PDO；也不得据 Reference 否定某一精确 EtherCAT 固件的 ESI 映射 |

## 5. 严格结论

1. `0x6062` 是位置控制器的需求输入；`0x6064` 是传感器测得实际位置；`0x60F4` 是驱动器报告的跟随误差，但精确算式未给出。
2. `0x6063` 的名称不能被扩写为“另一只编码器的实际值”。
3. 四个对象都以 `count` 为 Reference Manual 口径，官方对象存在不等于完成了 counts/mm 标定。
4. PDO 可映射性必须区分 CANopen Reference 与 EtherCAT ESI；二者当前不能合并成一个无条件结论。
