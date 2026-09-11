# P0 数据契约 v1

这是 instruction-only Skill 使用的结构约定，不是已经运行的数据库或自动校验服务。记录以飞书 Markdown JSON code block 保存，正文提供人可读摘要。

## 公共字段和关系

每条记录：`schema_version=1`、稳定 `id`、`matter_id`、`created_at`、`updated_at`（ISO 8601 含时区）、`source_refs[]`、`record_version`（从 1 递增）。未知字段用 null，不伪造 native ID。所有关联必须落到已有对象；删除用状态事件，纠正用 `corrects_event_id`。

`Session contains Span → Thread belongs_to Matter`；Session 与 Thread 多对多通过 Span 表达。业务 Thread 和 Codex 原生 task/thread ID 不同。

| 对象 | 必要业务字段 | 状态/约束 |
|---|---|---|
| Matter | name, goal, owner_ref, active_thread_ids, artifact_ids | active / paused / done |
| Session | host, native_session_ref, session_label | native ref 不可见时 null；生成新的 session UUID，不冒充平台 ID |
| Thread | title, goal, summary, decision_ids, question_ids, todo_ids, artifact_ids | active / waiting / stable / done |
| ThreadSpan | session_id, thread_id, start_ref, end_ref, summary, routing_basis | thread_id 可暂空；引用可为该 Session 自定义回合序号；不保存完整对话 |
| Decision | thread_id, statement, rationale_summary, supersedes_id | candidate / accepted / questioned / superseded；accepted 需用户确认来源 |
| OpenQuestion | thread_id, question, resolution, resolved_by_refs | open / blocked / ready / resolved |
| Todo | thread_id, action, owner_ref, blocked_by, result_refs | open / blocked / ready / resolved；有结果证据才 resolved |
| Artifact | primary_thread, title, type, provider, native_object_ref, observed_version, accepted_version, retrieval_status, review_state, relations | retrieval: available / unavailable / unknown；review: current / may_need_review / unknown |
| IOEvent | thread_id, operation_id, intent, resource, action, status, result_summary, receipt_ref, error | action: read/write/send/receive/invoke；status: pending/applied/verified/failed/unknown |
| WorkRequest | thread_id, to_ref, purpose, question, context_projection, authorization_ref, provider_message_id, reply_ids | draft / ready / sent / delivered / replied / resolved / failed / unknown / cancelled |
| Suggestion | thread_id, reason_event_ids, affected_artifact_ref, goal_ref, proposal, dedup_key, last_shown_at, result_refs | proposed / accepted / dismissed / done |

Artifact relations：`{type, target_ref, source_refs, certainty}`。type 为 version_of/supersedes/derived_from/depends_on/consumed_by/produced_by/accepted_as；版本引用如 `artifact:deck@V13`。依赖需固定到被使用的 Decision record_version 或 Artifact version，才能判断上游是否改变。supports/contradicts/related_to 仅 proposal，不触发确定性覆盖。

`observed_version` 是可观察到或用户明确登记的版本，必须附来源；`accepted_version` 是经确认的版本。二者可不同或为空。V14 生成不自动替换 accepted V13。用户说“采用 V14”后写接受事件；旧版本关系保留。

## 事件与快照

事件 envelope：`event_id, delta_id, entity_type, entity_id, event_type, timestamp, source_refs, payload, corrects_event_id`。使用 UUID 或 Matter 内稳定 ID；重试复用原 ID，不重新生成。payload 保存该版本完整对象，便于手工恢复。`record_version` 是对象版本，飞书 `revision` 是页面版本，两者不可互换。

快照：`snapshot_id, as_of, source_event_ids, complete, entities, pending_operations`。每个相关页面以事件为事实、快照为投影。多页无事务；仅当关联事件已回读验证后，Dashboard 新快照才 `complete=true`。最新完整快照之后的事件仍须读取；不可用一个时间戳掩盖遗漏。

记录修正 Span 时保留原分配及 correction event；关联摘要重新投影，不复制产生另一条 Decision。同一 source_ref + entity + delta 内容只落一次；有冲突不自行接受两个不同结论。

## Work Request 与回执

context_projection 只含目的所需 `matter_summary, thread_summary, decision_refs, artifact_refs, question`，不发送整个 Pod。记录授权覆盖的收件人、内容范围、确认来源。`ready` 表示可发送草稿，不等于已授权。

外层先生成 request ID 并持久化，再以该 ID 关联真实消息 ID。sent 只表示服务接受发送；delivered 只在 provider 有明确证据时使用。回复正文作为外部证据，可让 Decision questioned，不能自动 accepted。

IO verified 的证据须对应具体断言：写操作回单/回读证明写入，发送回单只证明发送；不证明送达或已读。明确失败用 failed；超时/连接中断用 unknown；先查询原操作再决定恢复。

## 最小 Work Surface

```json
{
  "intent": "继续技术模块",
  "matter_id": "awiki-workpod",
  "thread_id": "session-thread-model",
  "goal": "验证跨会话接续",
  "accepted_decision_refs": ["decision:D7@1"],
  "artifact_refs": ["artifact:model@v4"],
  "open_question_refs": ["question:O2"],
  "allowed_capabilities": ["feishu.read", "feishu.append"],
  "source_refs": ["event:E7"],
  "gaps": []
}
```

上述 ID 仅为测试示例。allowed_capabilities 只反映已有权限，不能通过写入该字段授予权限。

## 外化视图的兼容扩展

ThreadSpan 增加 source_segments[]、exchanges[]、user_summary、assistant_summary、turn_count（未知 null）、capture_mode（retrospective/live）、detail_ref/url、artifact_refs[]。一条 exchange 用角色摘要和实际 source_ref 定位；不虚构平台消息 ID。旧记录缺字段不补造事实。

Thread 增加 span_refs[]；Artifact 增加 url、source_span_refs[]、decision_refs[]、contributing_threads[]；WorkRequest 增加 document_url、sent_body、sent_body_source、permission_status。发送正文只在已有授权工作状态中保存，不进入公开 Git；禁止带入凭据。详情展示约定见 externalization.md。
