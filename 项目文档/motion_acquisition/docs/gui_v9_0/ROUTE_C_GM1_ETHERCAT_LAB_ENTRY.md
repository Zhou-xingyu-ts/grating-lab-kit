# Route C：GM1 / EtherCAT 线下测试入口（V9.0.0 候选）

## 当前边界

Route C 的固定拓扑是：

```text
Motion Acquisition → Modbus TCP → Panasonic GM1 → EtherCAT → DS-BVM
```

GM1 是 EtherCAT 主站；Windows 只管理 GM1，绝不作为 Windows EtherCAT
master。本文件是**后续线下测试的入口清单**，不是连接、扫描、进入
SafeOp/OP、写 Modbus、Servo On 或运动的授权。V9 的已实现内容仅包括
`gm1_route_v1` 离线寄存器契约、内存 Mock GM1 和 ESI/XML 只读审计。

当前真实 Route C 状态是 `unavailable`。在以下每个具体动作得到用户明确
授权之前，软件不得打开网络端点、枚举设备或发送任何命令。

## V9 离线证据

| 项目 | V9 已有内容 | 不能据此声称 |
|---|---|---|
| Windows—GM1 边界 | `gm1_route_v1`，含 magic/version/build ID/boot/heartbeat、状态、32 位 signed counts、命令—ACK 对账 | 当前 GM1 已实现该寄存器图或网络已可达 |
| Modbus | 内存 `MockGm1ModbusServer` 与 fail-closed adapter | TCP 连接、寄存器地址、IP、unit ID、写权限有效 |
| EtherCAT | ESI/XML 读取审计 PDO、DC 和 InitCmd | 已扫描从站、PDO 映射、DC、SafeOp/OP 或实时周期通过 |
| 停机 | Mock 的“请求保护停机”语义 | 物理急停、STO、GM1 或驱动器已进入安全状态 |
| 数值 | 高字在前的 signed 32-bit raw-counts 合同 | 工程 count 换算、校准精度或三轴位置准确度 |

离线审计的 ESI 报告必须另存到新的构建/验证报告目录，不能回写
`90_原始资料/`。审计器会显式列出 `InitCmd`；尤其 SafeOp→OP 过渡中的
对象写入必须视为写操作，而不是“天然只读扫描”。

正式产物入口为 `packaging/write_v9_route_c_esi_audit.ps1`：它要求显式
`-AcknowledgeLocalReadOnlyAudit`、本地来源说明和一个不存在的 `build/` 子目录，输出
`ROUTE_C_ESI_AUDIT.json`、`ROUTE_C_ESI_AUDIT.md`、`SHA256SUMS.txt`。它不联网验证
“官方”来源，来源记录与可选期望 hash 必须由调用者提供；详情见
[V9 Route C ESI 审计入口](V9_0_ROUTE_C_ESI_AUDIT_ENTRY.md)。该入口本轮未执行。

## 上台前必须准备的事实

以下项目不是可由 V9 推断的默认值，均需来源、照片/工程文件或实测记录：

1. 当前 GM1 型号、工程源文件、工程 build ID、下载状态和备份；
2. 已批准的 GM1 IP、网络隔离方式、Modbus unit ID 和只读诊断通道；
3. `gm1_route_v1` 在 GM1 工程中的实际寄存器映射、字序、boot/heartbeat
   更新语义及命令提交/ACK 生命周期；
4. EtherCAT 从站 identity、正确的 ESI 版本、拓扑和接线；
5. PDO、DC、InitCmd、SafeOp/OP 的计划操作及其每一项写入影响；
6. `39/40/46` 现场安全锚、E-stop/STO、刹车、限位和一个运动主站规则；
7. DS-BVM 使能状态及 Windows-PCAN/STM32/GM1 三条路线之间的排他性。

任何一项未知时，保持 Route C `unknown` 或 `unavailable`，不切换到其他
运动主站，也不发送补偿性命令。

## 经逐项授权后的最小阶段顺序

每一阶段都应单独记录：时间、操作者、输入配置、GM1 build/boot、ESI
hash、结果、失败码、停止状态和原始证据路径。失败不能删除，应成为下一步
的回归用例或现场缺口。

1. **工程与契约核对（不上 EtherCAT）**：离线比对 GM1 工程与
   `gm1_route_v1`。只确认 magic、版本、build ID、boot counter、heartbeat
   和 high-word-first signed 32-bit counts 的含义；不假定历史 IP/寄存器可用。
2. **受控网络只读诊断**：在用户授权的隔离网络中，验证 identity 和
   heartbeat 是否与合同匹配。magic/version/build ID/心跳不符时，只显示
   诊断并中止，禁止命令。
3. **ESI 与 EtherCAT 只读核对**：用原件哈希审计 ESI，记录 PDO/DC 和全部
   InitCmd。扫描、SafeOp 和 OP 均需要再次单独授权，因为 ESI 可能含隐式写。
4. **GM1—从站状态验证**：在现场放行后，才逐项验证扫描、从站 identity、
   PDO、DC、状态机和故障反馈。不能把“能看到设备”称为可控制。
5. **GM1 Modbus 命令对账**：仅在寄存器契约、停止策略和安全条件获放行后，
   验证 sequence、commit、ACK、阶段、错误码、重复命令和断线/重启后的
   `unknown` 行为。
6. **后续控制门**：只有前五项、`39/40/46`、物理停止链和一主站锁均闭合，
   才能另行提交 GM1/DS-BVM 使能或运动试验方案；本 V9 包不授予该权限。

## 通过与停止判据

- magic、协议版本、已登记 build ID 或 heartbeat 不匹配：仅诊断，Route C
  fail-closed；不自动重试未确认命令。
- boot counter 改变、网络断线、GM1/从站状态不能确认：状态为 `unknown`，
  保留主站锁，禁止切换路线或继续命令。
- ESI 审计发现 InitCmd：逐项列为潜在写入；未评审前不得进入相应转换。
- Windows 不安装、不启用 EtherCAT master 依赖；所有 EtherCAT 主站行为由
  GM1 工程及其现场放行承担。

只有完成上述现场证据链后，才能生成新的现场结论。V9 离线候选的结论仅为
`READY_FOR_LAB_TEST`，不代表 GM1、EtherCAT、PDO/DC、SafeOp/OP、三轴或
运动已通过。
