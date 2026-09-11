# AWiki Work Hub — Phase 1 / Skill P0

用 **Windows 上的 Codex + 现有 mac 飞书 MCP + 本地 AWiki 通讯 Skill**，验证 Agent 能否在正常对话中维护持续工作状态、接续 Thread、追踪成果元数据，并主动提出有依据的下一步。

状态：设计与可加载的 instruction-only Skill 已入库；**Windows 内外层 Skill 已安装，飞书跨主机读取通过；完整通讯与端到端 dogfood 尚待执行**。本仓库没有独立后端，也不宣称已经部署运行。运行状态保存在飞书；Git 只保存方案、Skill、配置模板和测试材料。

## 从这里开始

1. 阅读 [Phase 1 执行方案](docs/phase1_execution_plan.md)。
2. Windows 本地 Codex 按 [接手指南](docs/codex_handoff.md)逐项执行。
3. 加载 [AWiki Work Hub 外层 Skill](skills/awiki-workpod/SKILL.md)，稳定名称为 `awiki-workpod`。
4. 按 [P0 测试计划](tests/P0_TEST_PLAN.md)保存真实证据，区分通过、失败和未运行。

```text
用户 ↔ Codex
         └─ awiki-workpod（外层 AWiki Work Hub Skill）
              ├─ mac 飞书 MCP → 飞书 Work Graph + Observatory
              └─ 读取本地现有 AWiki SKILL.md
                   └─ 按其真实工具/脚本执行通讯 → 结果归回 Work Request
```

Skill 的“调用”是同一个 Codex 按内层说明使用真实工具，不是 `call_skill()`、RPC 或启动另一个 Agent。内层负责通讯，外层负责工作语义和飞书回写。

## 目录

```text
AGENTS.md
README.md
PLAN.md
skills/awiki-workpod/
  SKILL.md
  references/data_contract.md
  references/feishu_store.md
  references/communication_adapter.md
docs/
  phase1_execution_plan.md
  codex_handoff.md
  capability_inventory.md
config/example/workpod.example.json
config/example/codex-mcp.example.toml
tests/P0_TEST_PLAN.md
tests/fixtures/golden_matter.md
tests/RESULTS_TEMPLATE.md
```

保留四种连续性：思考（Session/Span → Thread）、成果（Artifact/Version/Dependency）、协作（Request/Reply → Thread）、观察（工作变化 → 人的注意力）。

明确不做：本地成果扫描/同步/daemon、复杂 Router、ANP federation、A2A 实现、通用图数据库、独立 UI、全量聊天抓取、常驻监听。读取 Skill 和配置文件属于部署，不属于本地成果管理。

## 设计来源与核验边界

基于前序《调研Agent对MCP覆盖》及实际读取的[飞书 Skill P0 文档 revision 7](https://ccnt8r8ypcpo.feishu.cn/wiki/EURQwljk4ipy9rkYevlcRMU4nde)，按用户本次要求把原 Phase A/B 收敛到本地 Codex 第一阶段。[主方案](https://ccnt8r8ypcpo.feishu.cn/wiki/RNmhwm2NziRy55kKks8cSomlnfh)作为背景入口；本次未重新核验主方案内容。未取得先前 zip 原件，本仓库是依据已核验设计重新整理的 Skill，不冒充原包迁移。

2026-09-11 已核验 mac 飞书工具声明及上述页面读取；本地 AWiki Skill 已安装；Windows → Mac MCP 读取已核验。具体证据见 [能力清单](docs/capability_inventory.md)。

本地预检：AWiki CLI 已更新到 1.0.48，当前工作区无可用身份；真实通讯尚未验证。

本机部署更新：飞书七页已初始化并回读验证，幂等重放与旧 revision 拒绝通过；完整测试与通讯迁移状态见 [本机验收记录](tests/results/2026-09-11-windows-bootstrap.md)。
