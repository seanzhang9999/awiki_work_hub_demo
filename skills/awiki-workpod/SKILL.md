---
name: awiki-workpod
description: "AWiki Work Hub 外层工作 Skill：用户在 AWiki Matter 中继续讨论、跟进成果、查询工作状态或提出协作请求时，借助飞书维护 Thread/Span、Artifact 元数据、Decision、I/O 和主动 Suggestion；需要通讯时包装本地现有 AWiki Skill。"
---

# AWiki Work Hub / WorkPod P0

为正常人机工作维护持续、可观察的外部 Work State。用户指令优先于本 Skill。Phase 1 仅在当前活跃会话内执行，不监控不可见会话或离线收件箱。

## 进入工作

读取用户指定的运行配置（默认仓库 `config/local/workpod.json`），解析配置中的真实路径和页面 refs。未部署时按仓库 `docs/codex_handoff.md` 接手；不要拿 example 中的空值执行调用。

首次使用读取 [数据契约](references/data_contract.md)及[飞书存储规则](references/feishu_store.md)。通过当前工具目录确认 mac 飞书真实工具名称；先读取 Dashboard、目标 Matter 最新完整快照和相关增量，不加载全部历史。构造最小 Work Surface：intent、goal、thread、accepted decisions、相关 artifact versions、open questions、allowed capabilities、sources、已知缺口。

## 每个有意义的工作回合

1. **BOOTSTRAP**：恢复 Matter / Goal / 最近工作状态与待确认操作。
2. **OBSERVE CONVERSATION**：从可见的用户与助手多轮问答中提取 Intent、Span 和 Work Delta，区分提议、承诺、已执行结果与用户接受。
3. **RESOLVE**：优先接续已有 Thread；A1→B1→A2 是三个 Span、两个 Thread。明确语义时自动归线，多个合理候选时才询问。低置信度 Span 可暂存 `thread_id=null`。不要用每次换话题创建 Matter。Span 是连续同一工作线的讨论区间，可含多轮问答；追问不自动新建 Span，中途切走再回来使用新 Span 关联原 Thread。同一轮多主题按内容范围拆分，避免重复计整轮。
4. **PROJECT**：只取当前 Thread 和必要依赖；跨 Thread 合成时说明纳入哪些已接受版本。
5. **ACT**：执行当前用户授权的正常工作。需要通讯时按下面的内层流程执行。
6. **EXTERNALIZE**：只记录 Decision、Question、Todo、约束、来源、成果变化、Reply、Receipt、纠错等可持续使用的增量；不保存完整聊天或隐藏推理。按稳定事件 ID 幂等追加，再更新带来源游标的快照。
7. **OBSERVE WORK STATE**：检查 Goal 变化、依赖变化、待办解除阻塞、等待回复、成果可能过时。
8. **SUGGEST**：提出有证据的下一步；没有变化则保持安静。

外化及成果输出时读取 [会话外化与成果映射](references/externalization.md)：维护双向表格、双方问答摘要、可点击的同源详情，以及成果到 Thread/Span/Decision 的双向链接。

以上是本 Skill 的行为步骤，不是已实现的函数或后台调度器。

## 通讯封装

仅在协作需求出现时读 [通讯契约](references/communication_adapter.md)，然后读取配置 `awiki.skill_path` 指向的真实 AWiki SKILL.md 及其相关引用。遵循其真实工具/脚本用法，输出归一化结果供本外层写回飞书。不要调用虚构 `call_skill()`；不要自行重新实现 AWiki 协议。

先持久化带 Matter/Thread 和 scoped context 的 Work Request。用户已明确授权具体收件人及内容范围即可发送；否则展示可审核草稿后获取授权。授权后正文/收件人改变则重新确认改变的范围。外部回复按照 request/provider message 映射回原 Thread，不能凭相似文本猜归属；不确定则暂挂。

内部 Skill 不可用时标记 `blocked_missing_awiki_skill`，继续飞书状态工作。超时为 unknown，先查回单/状态，不能换 ID 盲发。

## 成果、建议与观察

- Artifact Registry 只管 metadata；可按用户请求在飞书创建成果正文，不新增本地成果管理。observed 与 accepted 版本分开，接受必须有用户依据。不可回取则 unavailable/unknown，不宣称读过 PPT，不猜页码，不扫描本地文件。
- accepted Decision 或上游 Artifact 改变且有明确依赖时，标 `may_need_review`；只是语义相关则记 proposal，不能当确定关系。
- 建议包含触发 Delta、目标/成果、原因和具体动作。按“目标+原因事件+动作”去重；默认每回合最多一条、同因 24 小时内不重复；dismissed 同因永久抑制，除非用户重开或出现新证据。执行成功并有证据才 done。
- 用户问进度时展示 NOW/Attention、Matter Map、Thread Timeline、Artifact Map、I/O Monitor、Waiting Requests 和 Next Actions；每项带事件/来源，禁止虚构完成百分比。
- 飞书写入遵循 read/revision → append/operation_id → receipt/readback。未知结果明确说明；日志写入失败不递归记录自身失败，不伪称持久化成功。

结束回合报告：这次改变、受影响成果、已验证动作、真实阻塞、最有用下一步及飞书链接。仅对工作有变化的内容发声。
