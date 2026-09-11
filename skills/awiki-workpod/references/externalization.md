# 会话外化与成果映射

## 片段边界与依据

Span 是连续同主题的多轮问答，不是一条用户消息。连续追问、答复、修正和执行结果尽量保留在一个 Span；A→B→A 对应三个 Span、两个 Thread。单条消息涉及多个目标时，用 source_ref + segment_label 标记实际范围，不把整条重复算入多条工作线。主题已有独立目标、决策或成果时可新建 Thread，记录理由；展示名称调整保留稳定 ID。

读取当前可见对话及已授权来源。Session 使用已核验原生标题/ID；不可得时使用明确标注的工作名称，native_session_ref=null。重建历史标 retrospective；轮数不确定为 null。摘要不是逐字转录，引用原话必须实际可见。不记录验证码、密钥、完整私人聊天或隐藏推理。

## 双视图与点击详情

以同一组 Span 对象生成两表：

- 会话视图：会话名称、顺序/来源范围、Span、用户诉求摘要、助手回答/结果摘要、Thread、详情链接、成果链接。
- Thread 视图：Thread 名称/目标、Span、双方问答摘要、来源会话/位置、详情链接、成果链接。

详情保留有序 exchanges（user_summary、assistant_summary、source_ref、segment_label），以及结论、未决问题、执行证据和相关成果。不把助手建议写成用户决定。长答案概括，但保留关键纠正和限制。详情可用飞书页面或已验证块链接；两个表链接同一详情对象。若只有页链接，明确写“打开详情页，定位片段 X”，不声称能跳到具体段落。短详情可直接展开；不得用只有 ID 的 JSON 作为唯一用户界面。

已存在 Span 复用 ID；纠错追加新版本和 supersedes/corrects 引用，旧表标历史。后续只更新受影响片段与投影，不反复复制全会话。受 append-only 工具限制时建立独立可读页并在 Dashboard/Matter 链接，避免用户翻过大量 JSON。

## 成果的输入与输出

输出前投影最小上下文：primary_thread、contributing_threads、source_span_refs、accepted_decision_refs（固定版本）、问题与证据缺口。跨 Thread 汇总明确每部分来源，不能把无关聊天和凭据带入成果。

成果正文包含目标、结论与理由、局限/未决事项、上下文来源入口。Registry 保存 artifact_id、native_object_ref、url、observed_version、accepted_version、source_span_refs、decision_refs、relations；Thread/Span 保存 artifact_refs，提供反向入口。正文创建和回读后才标 available；未经用户接受，accepted_version=null。文档创建不等于安装功能或跨端测试通过。

成果依赖的决定变化时标 may_need_review，并说明受影响章节；新观点只是 proposal 时，不覆盖旧 accepted 决定。事件与快照须能从工作线找到成果，也能从成果还原使用过的上下文版本。

## 行为验收

用实际对话执行：多轮聚合、A/B/A 接续、同轮多主题、纠错、双方摘要、详情可回读、Thread↔Span↔Artifact 映射。报告具体样本和失败/缺口；回顾整理不等于在线检测，单会话不等于跨会话，人工自检不等于独立盲测。重复处理相同来源不能新建重复 Span 或 Artifact。
