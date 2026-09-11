# 给 Windows 本地 Codex 的接手指导

## 可直接粘贴的任务

```text
请在本仓库执行 AWiki Work Hub Phase 1。先读 AGENTS.md、PLAN.md、
docs/phase1_execution_plan.md 和 skills/awiki-workpod/SKILL.md。
复用当前已连接、服务端位于 Mac 的 mac 飞书 MCP；发现并读取我本地现有 AWiki 通讯 Skill，
将真实能力映射到外层 awiki-workpod 的通讯契约，不重写通讯服务。
按本指南 M0–M5 执行：盘点、加载、在允许的 AWiki 根内初始化专用 P0 页面、
验证 Thread/Span、Artifact metadata、Observatory、Suggestion 和授权收发。
允许本次建立和维护专用 P0 测试工作状态；此指令不授权向未明确选择的联系人发消息。
不要扫描本地成果文件，不做 daemon、复杂 Router、ANP 或新后端。
保存真实验收证据；凭据/通讯内容留本机或飞书；缺失能力如实记 BLOCKED，
继续不依赖该能力的工作，不用模拟结果冒充真实集成。
```

## M0：检查与发现

从当前 Windows checkout 打开 Codex，先查看仓库状态。没有 checkout 时：

```powershell
git clone https://github.com/seanzhang9999/awiki_work_hub_demo.git
Set-Location awiki_work_hub_demo
```

先用当前 Codex 工具目录发现 mac 飞书能力并调用 read_page；当前连接已证明 Windows 能访问 Mac 服务。无需把 Codex 搬到 Mac。CLI 的 `codex mcp list` 仅列其配置，不是桌面连接器可用性的唯一判断标准。仅在换到独立 CLI 且工具确实缺失时，按现有服务说明接入原 MCP；不要猜 endpoint、复制 Mac 凭据或把 8788 viewer 当 MCP。

AWiki 官方安装入口为 https://awiki.ai/cli/ 。先发现当前 Skill 列表和本机已知 Skill 目录，复用有效安装；没有时安装：

```powershell
npx.cmd --yes skills add https://awiki.ai/cli/stable --agent codex -y -g
awiki-cli version
awiki-cli status
```

本次已将 CLI 从 1.0.12 升级到 1.0.48，version/status/schema 检查通过；当前工作区没有可用身份，真实通讯需选择并接入身份。以下升级命令仅供其他尚未升级的环境使用，不需重复执行。Skill 安装与 CLI 升级分开：需升级时阅读官方 upgrade 说明，核验兼容性，不自动迁移或替换既有身份。已授权依赖升级时，可用官方 stable 包安装后再做只读状态检查；身份恢复/注册另按明确用户范围处理。

```powershell
npm.cmd install -g https://awiki.ai/cli/stable/awiki-cli.tgz
awiki-cli version
```

不要运行 listener/daemon setup。Phase 1 按需读取消息，使用内层支持的 HTTP 历史路径。CLI 自身的身份/消息存储是既有通讯依赖；“不做本地文件”指不新增 Work Hub 成果扫描与工作状态本地库。

## M1：Windows 本地加载

外层完整源目录在 `skills/awiki-workpod`。将其链接到仓库发现目录，Windows 用 Junction 避免管理员 symlink 要求：

```powershell
New-Item -ItemType Directory -Force .agents/skills | Out-Null
$workpodSource = (Resolve-Path skills/awiki-workpod).Path
if (-not (Test-Path .agents/skills/awiki-workpod)) {
  New-Item -ItemType Junction -Path .agents/skills/awiki-workpod -Target $workpodSource | Out-Null
}
New-Item -ItemType Directory -Force config/local | Out-Null
if (-not (Test-Path config/local/workpod.json)) {
  Copy-Item config/example/workpod.example.json config/local/workpod.json
}
```

如果入口已存在，检查目标，不覆盖。完整 references 跟随源码；不要只拷贝 SKILL.md。可将同样 Junction 放用户 `~/.agents/skills` 以跨项目发现，但不要同时装同名副本。

内部 Skill 使用安装器返回的实际路径，读取 frontmatter（官方包的 name 为 `awiki`，发布索引名称是 awiki-cli），填入配置。新任务显式选择外层 Skill；当前会话也可直接读取外层与内层完整 SKILL.md 执行，安装后下一轮/新任务可发现。刷新失败再重启，不能把手动读取当自动发现已验证。

依据 [Codex 官方 Skill 文档](https://learn.chatgpt.com/docs/build-skills)，本地发现读取 `.agents/skills`/用户目录，Skill 是指令包；Windows Junction 是否在当前客户端发现，需新任务实测。

## M2：连接与初始化飞书

1. 核验运行配置 phase=1，deployment_id 使用一个稳定、唯一的部署标识；只填实际允许 root 和工具名。example 中 null 表示尚未绑定，不能直接调用。
2. 通过真实 MCP 发现项目树，复用/创建 `AWiki WorkPod P0` 及七个页面（见 [feishu_store](../skills/awiki-workpod/references/feishu_store.md)）。创建前查标题、parent 和已有 receipt，防止重跑重复建页。
3. 每次建页/写入保留 operation_id、ref、revision 和 receipt；创建后读页再追加，不以空版本写入。
4. 回填 config 的 pages，Dashboard 同步记录页面索引、deployment_id 和 bootstrap operations，使丢失本地配置后仍可恢复。
5. 用一个无敏感内容的 smoke 事件验证 append + receipt + readback。只允许 guarded append，不调用 replace_page。

仅独立 CLI 缺少原连接时需要配置；Codex MCP 配置入口和 STDIO/HTTP 支持参见[官方 MCP 文档](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)。本仓库没有提供未知服务的可直接运行启动命令。

## M3–M5：执行与验收

- 以 [Golden Matter](../tests/fixtures/golden_matter.md)作为场景指引，先跑 T01–T09、T15–T19 和补充存储测试。
- 内层能力映射清晰后，用用户明确选择的测试联系人和精确草稿跑 T10–T14。发送前验证授权；查到真实回复才测试 reply routing。不要为了通过测试生成假回单。
- 在至少 3 个 Session/30 回合的真实 AWiki 产品讨论中采集指标；不要求用户每回合整理结构。
- 新会话只给 Dashboard URL 与当前 Intent，验证 Work Surface 恢复。若用同一平台新会话，只记冷启动；只有实际换 Agent runtime 才记跨 Agent 验证。
- 将 [结果模板](../tests/RESULTS_TEMPLATE.md)复制到 `tests/results/local/` 填写；公开总结可另写脱敏 Markdown，不提交真实消息/私人引用。
- 更新 PLAN.md 中里程碑，仅依据真实证据勾选。最后提供飞书 Dashboard、Skill 路径、通过/失败/阻塞列表和下一步。

## 故障与停止条件

远端 Mac MCP 不可用：停止飞书变更，修复连接或标阻塞；仍可完成加载/契约审阅。AWiki 缺失：继续飞书，通讯阻塞。写超时：原 operation 查询；不能盲目改 ID 重试。403：不扩权，报告受限对象。两次冲突重试仍失败：停止该页写入，保留待补步骤。不可逆动作或新外发未授权：先准备可审核结果，再请求所缺授权。

恢复/撤销只追加修正事件或停用本次 Skill 入口；不删除原飞书材料，不重置全局 Codex 配置。所有“本地文件不做”限制针对业务成果管理，允许必要的 Skill/配置部署文件读写。
