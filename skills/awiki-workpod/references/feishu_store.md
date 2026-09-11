# 飞书状态库与 mac MCP 调用

## 页面约定

先读取 allowlist 与项目树，复用现有 P0 容器；没有时在已允许 AWiki 根下创建专用 `AWiki WorkPod P0` Docx 容器。不要把设计源文档作为运行日志覆盖。以下是 P0 容器下的精确标题，与原 P0 文档一致：

| 页面 | 内容 |
|---|---|
| 00_Work_Dashboard | 页面 ref 索引；最新完整 Observatory 快照；NOW/Attention、Matter Map、Artifact Map、等待与下一步 |
| Matter_AWiki_WorkPod | Matter、Thread、Session/Span、Thread Timeline；Goal、当前快照和追加事件 |
| 01_Artifact_Registry | Artifact metadata、版本、依赖及 review 状态 |
| 02_Decision_Todo | Decision / OpenQuestion / Todo 及状态变化 |
| 03_IO_Event_Log | 有业务意义的 read/write/send/receive/invoke、回单和故障 |
| 04_Work_Requests | 草稿、授权范围、provider 映射、Reply、收发状态 |
| 05_Suggestions | 原因、目标、动作、去重键、接受/忽略/完成 |

每页头部说明 schema_version、用途；每次追加“事件摘要 + JSON record + 必要快照”。不建 Bitable record store：当前暴露能力偏结构/表单，不应假设已有通用记录 CRUD。Threads/Spans 在 Matter 页管理，不另建复杂树。

页面标题仅用于首次发现，随后用返回的 node/doc ref。同名多页不自动选第一项；检查所属容器、schema 和历史 receipt。将页面 ref 回填运行配置，同时把索引写 Dashboard，以便换 Agent 后仅凭入口恢复。

## 已核验的工具参数

以下为 2026-09-11 当前连接器声明，不同宿主可能有不同前缀，启动时以真实目录为准。工具调用由 Codex 发起；这些 JSON 不是可在终端直接运行的命令。

```text
mac_read_page({page_ref, detail?: "simple" | "full"})
mac_create_child_page({operation_id, parent_ref?, root_alias?: "awiki" | "journal", title})
mac_append_markdown({page_ref, expected_revision, operation_id, markdown})
mac_get_sync_receipt({operation_id})
```

只读准备：发现并使用 `mac_identity_status`、`mac_list_allowed_roots`、`mac_get_project_tree` 的当前声明，不猜参数。取工具返回 `page.feishu_url` 作跨设备链接。`awiki_local_url` 是 Mac 本机浏览器入口，不是远端读取 API，也不授予权限。

示例调用链（伪代码，字段值均从上一步取得）：

```text
read = mac_read_page({page_ref: configured_ref})
revision = read 返回的实际 revision
op = 本次逻辑写入的稳定 operation_id
mac_append_markdown({
  page_ref: configured_ref,
  expected_revision: revision,
  operation_id: op,
  markdown: 带 event_id 的工作增量
})
mac_get_sync_receipt({operation_id: op})
mac_read_page({page_ref: configured_ref})  // 查 event_id 和实际内容
```

## 幂等、冲突与恢复

1. 先生成 delta_id，读取目标页、检查相同事件是否已存在。每个目标页每个内容版本有独立稳定 operation_id；记录在 Matter pending_operations 中，再执行关联页写入。初始化建页 operation ID 使用部署 ID + 页面逻辑键，重启复用同一部署 ID。
2. 成功回单加回读确认实际内容；投影页更新后最后追加 Dashboard 完整快照。日志自身写入不再产生无限日志；IO 页中的一次汇总记录包含这个批次的状态。
3. 超时优先查询原 operation_id receipt 并回读。结果仍未知时停止重发，报告 unknown。尤其通讯外发不能用新 ID 重试。
4. 明确 revision conflict：重新读取并合并增量，最多两次。内容发生变化时，只有确认原操作未应用，才以关联的 replacement operation_id 提交新 payload；不把同一 ID 对应到不同内容。无法确认就停在 unknown。
5. 多页部分成功：保留成功事件，Dashboard 标 incomplete / pending；下次只补缺失事件和投影。不得通过全页覆盖“回滚”。需要撤销时追加 correction/superseding event。
6. 人工编辑期间冲突按同样流程处理。P0 单写者约定是运行限制，不是分布式锁保证。碰到持续并发写，暂停该页并报告。

使用 append-only，保留富文本。最新快照追加到页尾，标明确 as_of / complete / source_event_ids。体量超出当前工具可完整回读范围时，不悄悄漏历史：暂停该页投影，创建按期分卷的计划交用户决定；迁移查询型存储留后续阶段。

飞书不可用时只能在当前回复呈现待同步增量和 unknown，不创建本地 Work Graph。会话中断可能丢失未写入增量，这是 Phase 1 的明确限制。
