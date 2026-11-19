# V9 Route C 官方 ESI/XML 审计入口

## 当前状态

本入口已经写入源码，但本轮未执行。它只允许读取一个本地 ESI/XML 原件，向一个全新的
`build/` 子目录写 JSON、Markdown 和 SHA-256 账本；不访问网络、不枚举设备、不启动
Windows EtherCAT master、不扫描从站，也不进入 SafeOp/OP。

“官方”来源不能由文件名自动证明。因此调用者必须给出本地归档/下载记录的
`OfficialSourceReference`，并可选给出已登记的 `ExpectedSourceSha256`。入口会记录这两项，
但不会联网验证来源或下载更新文件。

## 目的、输入、输出与风险

| 项目 | 内容 |
|---|---|
| 目的 | 让 ESI 声明中的 identity、PDO、DC、InitCmd（含 SafeOp→OP 潜在写入）和未闭合项可复核 |
| 本地输入 | 一个现存 XML 文件、来源说明、可选输入 SHA-256、项目 Python |
| 新输出 | `ROUTE_C_ESI_AUDIT.json`、`ROUTE_C_ESI_AUDIT.md`、`SHA256SUMS.txt` |
| 覆盖策略 | 输出目录必须不存在；入口拒绝覆盖或复用，不删除旧目录 |
| 源文件策略 | XML 只读、不复制、不改名、不写回 `90_原始资料/` |
| 风险边界 | 只产生新本地证据文件；不证明 GM1 工程、ESI 真伪、设备 identity、PDO/DC 实配、SafeOp/OP、安全或运动 |

`SHA256SUMS.txt` 覆盖两个生成产物；原始 XML 保持在其保护位置，其 SHA-256 同时记录在
JSON 与 Markdown 中，避免把原件复制进发布候选或原始资料目录。

## 未来经批准后的命令

以下命令只是待批准的离线入口示例。运行前仍需逐批确认精确原件、来源记录、输出目录和
无硬件 I/O 边界；不要在已有目录上重试。

```powershell
.\packaging\write_v9_route_c_esi_audit.ps1 `
  -PythonExe .\.venv\Scripts\python.exe `
  -EsiSource "..\90_原始资料\官方软件\DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网\WELCON Systems EtherCAT Drive_V3_12_P0x00000001_WE.xml" `
  -OfficialSourceReference "90_原始资料/官方软件/DINGS_DS-BVS-FETC_DS-BVM-FETC_E_ESI_XML_官网; local archived official-source record" `
  -OutputDir .\build\v9_route_c_esi_audit_YYYYMMDD_01 `
  -AcknowledgeLocalReadOnlyAudit
```

若已有独立登记的原件 SHA-256，可额外传入
`-ExpectedSourceSha256 <64-hex-value>`。不应先用此入口生成的 hash 回填成“官方来源证明”；
该 hash 仅固定本次读到的本地字节。

成功时，控制台会输出每个生成产物的路径与 SHA-256，并明确输出：

```text
real_hardware_access=false
device_write_capability=false
ready_for_lab_test=false
```

## 发布接口

`packaging/package_v9_release.ps1` 不会替调用本审计入口。只有本审计入口已经成功并保留
完整目录后，才可将该目录通过 `-EsiAuditEvidenceDir` 附加到发布候选。打包脚本会要求并验证：

- `ROUTE_C_ESI_AUDIT.json` 的 `candidate_version=9.0.0`、本地只读结果和六项 false
- 输入的 operator-declared official-source reference、64 位原件 SHA-256、`source_write_performed=false`
  以及六项 false 的硬件边界；
- `ROUTE_C_ESI_AUDIT.md`；
- `SHA256SUMS.txt` 中每项生成物的实际 hash。

失败、输入 hash 不匹配、XML 无法解析或目录已存在时，不得把任何残余目录伪装成通过证据。
保留失败目录并另选新的输出名，记录失败原因。
