# Motion Acquisition V6.0 发布说明

## 新增

- STM32H743IIT6 Cortex-M7 C/ASM/CMSIS/linker 工程；
- Arm GNU 交叉构建的 ELF/HEX/BIN/map；
- 绝对截止期、固定容量、无 heap 的 embedded runtime；
- PC byte stream、CANopen stack、feedback、safety 明确端口；
- protocol 4.0 bounded decoder 和 typed dispatcher；
- 三轴位 CANopen node health manager；
- debugger 持久诊断区和 Cortex fault handlers；
- 官方 CMSIS 最小依赖、提交和许可证 provenance；
- 一键固件构建、segment/symbol/no-write 审计和 build report；
- 未来上板的 evidence-gated checklist。

## 调整

- shared `core.c` 不再使用 `snprintf`，避免 MCU 引入 stdio/heap；
- Windows native controller、Python SIL、采集、回放、分析、研究和数据格式继续
  保持；
- GUI 只更新版本和边界说明，不增加控制功能或重做界面。

## 明确未包含

- 板级 FDCAN/UART/USB/Ethernet；
- CANopenNode target port 或任何驱动写 profile；
- SDO download、NMT write、PDO configuration、controlword/mode write；
- 真实反馈、安全 IO、watchdog、STO/制动/限位；
- 三轴轨迹、同步、运动、精度或现场验收。

## 本机离线验收

- Arm GNU 交叉构建生成 H743 ELF/HEX/BIN/map；
- 固件发布包源码可独立复编并得到相同 ELF SHA-256；
- Python 145/145、CTest 4/4、Python→embedded C 协议一致性通过；
- 当前可访问历史会话 19/19 兼容，212,378 行源 CSV 哈希零变化；
- A/B/C SIL、native completed/stopped/error 和超时收尾记录均可校验；
- Windows onedir GUI、Python SIL helper、native C helper 和两层 checksum
  manifest 完成本机冒烟；
- 固定用户名、盘符、源码路径扫描为零。

这些结果只证明构建、协议、边界、兼容和软件在环行为，不证明板卡启动、真实
通信、三轴控制、安全、同步、精度或第二台电脑部署。
