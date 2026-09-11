# 外层 Work Hub 包装内部 AWiki Skill

## 执行方式

外层 `awiki-workpod` 持有 Work Request、来源、授权、I/O 和 Suggestion；现有 AWiki Skill 持有通讯实现。Codex 在需要时读取 `awiki.skill_path` 的 SKILL.md，按它引用的真实 CLI/MCP/API 执行。不要求内层理解 Matter，也不修改内层协议。

本文件中的 `communication.*` 是**语义契约名称**，不是可以直接调用的 API。本地执行者必须填能力映射：真实 Skill 章节、真实命令/工具、输入字段、返回字段、分页/游标、错误和回单行为。缺失能力标 unavailable，不编造 endpoint。

| 语义 | 输入 → 输出 | Phase 1 落地 |
|---|---|---|
| list_contacts | query/cursor → candidates, next_cursor | 调内层真实发现能力；若只支持已知地址，用户提供地址并如实记录缺口 |
| get_relationship | contact_ref → known facts | 有则使用，无则 unavailable，不推断私人关系 |
| prepare_request | scope/question/recipient → draft | 外层完成，无需新增通讯服务 |
| send_request | authorized draft/request_id → provider_message_id/status/receipt | 内层真实发送能力；保留原收件人和正文范围 |
| get_updates | provider refs/cursor → messages, next_cursor | 仅用户查询或活跃工作回合调用；不建 poller |
| respond | inbound reference/authorized text → result | 内层支持时使用，否则明确不可用 |

统一结果 envelope：`capability, request_id, status, provider_message_id, receipt_ref, observed_at, result_summary, error, next_cursor`。真实返回没有的字段保留 null。保存脱敏摘要，凭据由内层原有机制管理。

## 一次完整工作请求

1. 外层从当前 Thread 的最小 Work Surface 准备 WR：目的、问题、收件人、选定决策和成果引用。
2. 写飞书 WR draft，并给用户看精确草稿。已有明确授权时引用该授权；否则取得授权后才发送。只给链接不代表对方有权限，不自动扩大飞书分享权限。
3. WR ready + authorization_ref 持久化成功后调用内层，能传 client request ID 时传入。无法传递也保留外层 WR → provider_message_id 映射。
4. 写 I/O 与发送结果。发送成功但飞书回写失败时，显示真实 provider ID；下一回合先查原 WR/pending operation 和 provider，不重复发信。
5. 显式获取 updates。按 provider_message_id / in_reply_to 找原 WR，确认消息来源和对象；无法关联先 pending，不按主题强行归线。
6. 以 provider reply ID 去重，落 External Feedback、原 Thread 事件和证据。影响已接受判断则 questioned；解除 Todo 阻塞可 ready；再评估 Suggestion。
7. 回复已被工作消化并有结果来源后 WR resolved。外部说“完成”不等于成果已审核通过。

超时不重发；先查询实际 provider 状态。若内层既不支持幂等又无状态查询，结果保持 unknown，交用户核实，不能宣布端到端 exactly-once。

## 不可用时

- 未发现内部 Skill：blocked_missing_awiki_skill，继续飞书闭环；需要确切安装来源/路径，不安装猜测包。
- 无联系人查询：可测试已有授权测试联系人收发，但 Find Person 测试记 BLOCKED。
- 无收件能力：可保存 request 草稿/发送状态，回复闭环未通过。手工输入模拟回复只计 MOCK。
- 凭据过期/403：停止相应调用，报告原始脱敏原因；不能自动换身份或扩大权限。


## 官方内层 Skill 的已发现映射（2026-09-11）

从官方 stable 发布包读取；以下是 Skill 声明，执行前用本机 CLI schema 核验版本，不等于已完成通讯测试。

| 外层语义 | 内层声明命令 |
|---|---|
| list_contacts | `awiki-cli people contacts list`；查询关注可用 `people following`，不是任意全网专家搜索 |
| get_relationship | `awiki-cli people status <TARGET>` |
| prepare_request | 外层生成，不调用网络 |
| send_request | `awiki-cli msg send --to <target> --text <text>`，先 dry-run |
| get_updates | `awiki-cli msg inbox` / `awiki-cli msg history --with <target>`，不使用 `--mark-read` |
| respond | `msg send`；是否有 reply 参数以 schema 为准，无则仅附 WR 关联，不能伪造原生 reply threading |

官方源：[CLI 与 Skill](https://awiki.ai/cli/)。归一化时解析 `ok/data/error/meta`，不只看 summary。若当前 schema 支持 `--client-message-id` / `--idempotency-key`，传稳定 WR 关联键并验证返回；支持声明不替代真实幂等测试。
