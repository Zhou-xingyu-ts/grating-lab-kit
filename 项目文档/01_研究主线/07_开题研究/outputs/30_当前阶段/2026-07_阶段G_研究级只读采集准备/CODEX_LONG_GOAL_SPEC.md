# 阶段G长期Goal规范：研究级只读采集与平衡补采准备闭环

## 1. 唯一目标

把当前motion_acquisition和现有研究材料推进到“可以安全开展下一轮平衡重复实验”的研究级准备状态：

1. 只读采集具有清楚、可验证的V2时间语义；
2. 每个实验具有完整元数据、参数快照和事件标记；
3. 旧日志仍可读取，新日志结构能够离线验证；
4. 平衡重复实验、数据质量门和模型裁决规则已经预注册；
5. 所有实现仅通过离线模拟、单元测试、契约测试和静态安全审计验证；
6. 最终停止在真实硬件连接和实验执行之前。

本Goal不重新研究论文方向，不选择控制算法，不建立正式模型，不联系导师，不改变研究路线。

## 2. 必须首先读取

### 当前平台与路线证据

- 阶段FR的PHASE_FR_FINAL_HANDOFF.md
- 阶段FR的05_REVISED_PROVISIONAL_RESEARCH_LINE.md
- workspace_evidence_provenance/WORKSPACE_PROVENANCE_FINAL_HANDOFF.md
- workspace_evidence_provenance/02_KEY_CLAIM_PROVENANCE_LEDGER.md
- workspace_evidence_provenance/03_VERSION_AND_CONTRADICTION_REGISTER.md

### 历史模型审计

读取：

01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_师兄论文机构模型审计/

至少完整读取FINAL_HANDOFF.md，并按需追溯其余七份文件。

### 近五年文献调研

读取：

01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_近五年论文方向与控制算法调研/

重点读取FINAL_HANDOFF.md、05、06、07、08。

### 既有数据可行性审计

完整读取：

01_研究主线/07_开题研究/outputs/10_证据与现状/2026-07_D1-D3既有数据可行性审计/

包括分析脚本和audit_results.json。

### 当前采集代码

完整审计motion_acquisition的：

- src
- tests
- configs
- docs
- logs中的manifest结构
- pyproject及CLI入口

不得只根据HANDOFF推断当前源码。

## 3. 不得改变的事实边界

必须保持：

- 当前CANopen实物链只允许SDO upload；
- 不允许SDO download；
- 不允许NMT状态切换；
- 不允许PDO配置或收发控制；
- 不允许SYNC生产；
- 不允许Controlword、Target Position或其他运动对象写入；
- 不允许连接CAN、串口、USB或EtherCAT；
- 不允许发送任何运动命令；
- 不允许修改原始CSV、manifest、实验记录或历史审计输出；
- 不允许把人工事件标记解释为真实USB命令发送时刻；
- 不允许把SDO轮内对象解释为同步快照；
- 不允许把counts换算值解释为正式物理标定；
- 不允许设计补偿器、增大模型阶数或选择最终控制器。

所有测试必须使用fake client、mock、fixture或已有静态日志。

## 4. 允许修改的范围

只允许修改：

- motion_acquisition/
- 新建目录：
  01_研究主线/07_开题研究/outputs/30_当前阶段/2026-07_阶段G_研究级只读采集准备/

不得修改阶段A—FR、模型审计、文献调研和D1—D3审计的原文件。

## 5. 持续状态管理

立即创建并持续更新：

- GOAL_STATE.md
- ITERATION_LOG.md
- BLOCKER_REGISTER.md

GOAL_STATE必须记录：

- 当前检查点；
- 已验证内容；
- 未通过质量门；
- 最近测试结果；
- 当前代码变更；
- 下一项最优动作；
- 是否出现阻塞；
- 剩余完成条件。

任务中断恢复时必须先读取上述三个文件，不得从头重复。

## 6. 执行循环

每个检查点必须采用：

PLAN
→ IMPLEMENT或ANALYZE
→ TEST
→ SAFETY_REVIEW
→ DATA_SEMANTICS_REVIEW
→ REPRODUCIBILITY_REVIEW
→ REVISE
→ LOCK

任何审查未通过，不得进入下一检查点。

不得因为文档已经写完就宣布Goal完成。

## 7. 检查点G0：仓库基线

完成：

1. 记录当前motion_acquisition目录和版本状态；
2. 运行现有离线测试；
3. 确认测试不调用真实硬件；
4. 建立当前日志和manifest模式基线；
5. 确认现有只读安全测试；
6. 记录已知缺陷和兼容性约束。

不得修改功能，先锁定基线。

通过条件：

- 基线测试结果可复查；
- 当前安全边界有具体源码和测试证据；
- 旧日志结构已登记；
- 未运行硬件。

## 8. 检查点G1：V2时间语义规范

定义并文档化：

- experiment_id
- run_id
- sample_seq
- requested_period_s
- round_start_monotonic_ns
- object_request_start_monotonic_ns
- object_response_end_monotonic_ns
- round_end_monotonic_ns
- object_order
- actual_round_interval_s
- round_processing_duration_s
- object_duration_s
- deadline_miss
- manual_event_marker
- internal_target_change_candidate
- wall_clock时间与monotonic时间的关系

明确：

- actual cadence与processing capacity不是同一量；
-manual event marker不是USB命令真时刻；
- 0x6062变化只能作为内部目标变化候选；
- 轮内对象存在串行偏移；
- 无法确定的时刻必须保留为unknown或区间。

先完成规范、示例和失败案例，再进入实现。

## 9. 检查点G2：只读安全契约

在修改采集代码前先增加或强化测试，证明：

- 公开API不存在写对象方法；
- fake client记录的全部调用均为upload；
- 真实硬件适配器在测试中不能实例化；
- 0x6040、0x607A及其他写对象不能进入真实采集列表；
- NMT、PDO、SYNC和运动命令路径不存在；
- manifest始终声明只读安全边界；
- 旧有安全测试继续通过。

安全契约测试不通过时不得继续实现。

## 10. 检查点G3：时间语义V2实现

在motion_acquisition内实现：

1. 每轮开始和结束时间；
2. 每对象请求开始和响应结束时间；
3. 轮内对象顺序；
4. 实际轮次间隔与处理耗时分离；
5. event marker的离线/人工接口；
6. 实验元数据输入；
7. 参数快照字段；
8. manifest schema_version；
9. 旧日志读取兼容；
10. 新日志字段的类型和缺失值策略。

要求：

- 默认行为仍为只读；
- 未提供新元数据时仍可运行；
- 不把人工输入伪装成设备读取；
- 不覆盖旧格式；
- 不修改已有原始日志；
- 所有新功能由fake client测试。

## 11. 检查点G4：实验元数据和manifest V2

至少支持记录：

- 唯一实验编号；
- 操作者填写的运动模式描述；
- 起始位置；
- 目标counts；
- 方向；
- velocity；
- acceleration；
- deceleration；
- jerk；
- 预停留和后停留；
- 工况随机化序号；
- 重复编号；
- 训练/验证批次标签；
- 设备身份的已知与未知字段；
- 参数来源：人工填写、配置导入、设备读取或未知；
- 异常记录；
- 软件版本；
- 配置哈希；
- 数据文件哈希。

不得声称人工填写参数已由驱动器回读确认。

## 12. 检查点G5：平衡重复实验协议

设计但不执行：

- 使用既有安全小行程；
- 两个已验证速度等级；
- A/D/J=150与300；
- 完整正反往返；
- 每个工况至少6次重复；
- 分块随机顺序；
- 独立批次或独立日期留出；
- 统一预停留与后停留；
- 不同方向使用匹配位置区间；
- 异常运行单独标记，不静默删除。

必须明确：

- 哪些字段必须实验前填写；
- 哪些由程序自动生成；
- 哪些需要实验后核验；
- 中止条件；
- 安全边界；
- 数据不足时的降级结论。

不允许自动生成任何硬件运动命令。

## 13. 检查点G6：预注册分析和模型门

预注册D4→D1→D2→D3分析链。

### D4

检查：

- 时间语义；
- cadence；
- deadline miss；
- 串行对象偏移；
- 数据完整性；
- counts域与物理单位边界。

### D1

只允许：

- 静态仿射；
- 一阶；
- 一阶加延迟；
- 二阶；
- ARX2；
- 分工况低阶模型。

### D2

必须分离：

- 方向效应；
- 速度效应；
- A/D/J效应；
- 运动阶段效应；
- 交互效应；
- 重复批次效应。

### D3

主要依据：

- free-run；
- 独立验证；
- 跨重复；
- 跨工况；
- 参数稳定性；
- 残差结构；
- 域外退化。

一步预测只能作为辅助指标。

预先写明模型升级、保留、拒绝和暂停条件。

## 14. 检查点G7：离线端到端验证

使用fake client和已有fixture完成：

1. 一次完整V2采集模拟；
2. V2 CSV与manifest生成；
3. 旧格式兼容读取；
4. 事件标记；
5. 参数元数据；
6. 非均匀耗时模拟；
7. deadline miss；
8. 失败读取或超时分类；
9. 数据哈希和配置哈希；
10. 离线审计报告。

不得连接任何实物适配器。

## 15. 检查点G8：三重反方审查

分别以三种角色审查。

### 安全审查者

寻找任何可能产生写操作、真实硬件访问或越过安全边界的路径。

### 数据语义审查者

寻找时间戳误读、处理容量冒充实际采样率、人工参数冒充设备参数、串行读取冒充同步采样等问题。

### 可复现性审查者

寻找缺少版本、哈希、配置、随机化记录、实验编号、训练验证污染和不可重复步骤的问题。

每名审查者必须列出：

- 发现的问题；
- 严重性；
- 修订；
- 修订后验证。

存在高严重性问题时必须返回相应检查点重新迭代。

## 16. 检查点G9：最终验证

只有同时满足以下条件才完成：

1. 所有原有离线测试通过；
2. 所有新增测试通过；
3. 只读安全契约通过；
4. 不存在真实硬件调用；
5. V2时间语义具有代码、测试和文档三方证据；
6. 旧日志保持兼容；
7. 新日志示例可被重新读取；
8. 实验协议完整；
9. 预注册模型门完整；
10. 三重反方审查关闭全部高严重性问题；
11. 所有改动仅位于允许目录；
12. 原始日志和既有阶段文档未改变；
13. 最终交接能指导未来现场操作，但本Goal没有执行现场实验。

## 17. 最终输出

阶段G目录最终至少包含：

- CODEX_LONG_GOAL_SPEC.md
- GOAL_STATE.md
- ITERATION_LOG.md
- BLOCKER_REGISTER.md
- 00_BASELINE_AUDIT.md
- 01_TIMING_SEMANTICS_V2.md
- 02_READONLY_SAFETY_CONTRACT.md
- 03_MANIFEST_V2_SPEC.md
- 04_BALANCED_EXPERIMENT_PROTOCOL.md
- 05_PREREGISTERED_ANALYSIS_AND_MODEL_GATES.md
- 06_OFFLINE_END_TO_END_VALIDATION.md
- 07_ADVERSARIAL_REVIEW.md
- FIELD_EXECUTION_CHECKLIST.md
- FINAL_HANDOFF.md

代码和测试保留在motion_acquisition的合理位置，不复制进输出目录。

## 18. 阻塞停止条件

只有遇到以下情况才可在未完成状态停止：

- 必需文件实际不存在；
- 现有代码结构无法在不破坏只读安全边界的前提下实现；
- 基线测试本身无法离线运行且无法在允许范围内修复；
- 出现必须由用户选择的互斥数据合同；
- 达到预算限制。

停止时必须写入：

- 已完成检查点；
- 尝试过的路径；
- 具体证据；
- 阻塞原因；
- 最小解锁输入；
- 未完成的成功条件。

不得用“工作量较大”“建议以后做”作为阻塞理由。

## 19. 明确禁止自动扩展

本Goal完成后不得自行启动：

- 真实补采；
- CANopen写控制；
- CSP/CSV；
- STM32开发；
- EtherCAT调试；
- 补偿器设计；
- 正式系统辨识；
- 模型增阶；
- 新一轮联网调研；
- 论文写作；
- 最终路线冻结。

达到检查点G9后生成FINAL_HANDOFF并停止。
