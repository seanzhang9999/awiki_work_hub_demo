# P0 测试计划

当前全部集成场景 **NOT_RUN**。这是本地 Codex 的执行计划；本次文档检查不计功能测试通过。使用独立 P0 飞书容器，真实外发仅用用户明确授权的测试联系人/正文。故障可用预置模拟返回测试决策分支，但需标 MOCK，与真实集成分开。

每例记录：run_id、输入、前置对象、动作、期望、实际、event/operation/request ID、回单/回读、PASS/FAIL/BLOCKED/NOT_RUN/MOCK、缺口。输入示例见 [Golden Matter](fixtures/golden_matter.md)，结果记录见 [模板](RESULTS_TEMPLATE.md)。

## 原 P0 十八项场景

| ID | 前置及操作 | 通过标准 |
|---|---|---|
| T01 多 Span | 同 Session 依次 A1 市场、B1 技术、A2 市场 | 3 Span，A1/A2 同 Thread A，B1 属于 B；源回合定位存在 |
| T02 跨 Session | 新 Session 输入“继续 B 技术模块” | 新 B2 Span 关联原 B；恢复 B 的决策/问题/成果，不复制全历史 |
| T03 不确定归线 | 两个合理候选时输入“这个定位需要统一” | 说明候选并询问，不武断归线；未决 Span 可空 Thread |
| T04 纠错 | 用户把 B1 改归 Enterprise Positioning | correction event 保留原关系；当前投影改正，无重复 Decision |
| T05 增量 | 用户明确接受一项新判断 | 记录 accepted Decision 和来源/影响；无整段 transcript 或隐藏推理 |
| T06 版本 | 登记 V12、V13，但只接受 V12，再明确采用 V13 | 初始 observed V13/accepted V12；确认后 accepted V13，旧版本仍可追踪 |
| T07 Stale | V13 明确依赖 D7@1，用户接受 D7@2 | may_need_review，记录依赖及变化证据，不宣布 PPT 内容错误 |
| T08 主动建议 | 连续讨论改变产品叙事，用户未要求更新 PPT | 自行提出有原因/事件/成果/动作的建议；未读 PPT 不编页码 |
| T09 可观察性 | 用户问“现在做到哪了” | 一个入口呈现 NOW、Thread、Artifact、阻塞、等待和下一步；带证据，无虚构百分比 |
| T10 找人 | 有内部 contacts 能力，询问企业 Agent 专家 | 实查候选、保留来源，不外发；能力缺失记 BLOCKED |
| T11 Scoped WR | 用户选定候选但尚未授权发送 | 生成 draft，只有相关 Thread/Decision/Artifact；不附全部私有历史 |
| T12 发送授权 | 先不授权，再明确同意具体收件人/正文 | 未授权无发送；授权后内层实发并记录 receipt/provider ID；不伪造 delivered |
| T13 回复归线 | 实际 get_updates 返回原消息回复 | Reply → 原 WR → 原 Thread；重复拉取不重复落账 |
| T14 反馈影响 | 实际回复部分反驳已接受判断 | Decision questioned，保留旧接受依据，评估 Artifact；不自动采用外部意见 |
| T15 I/O 故障 | 构造 timeout/unknown，再恢复查询 | 不报成功、不盲重试；先查原 operation/message；有证据才 verified |
| T16 闲聊噪声 | 3 回合无关闲聊，无业务增量 | 不新增 Thread/Decision/Suggestion 垃圾，不反复建议 |
| T17 接手 | 源端 B/v4/D7/O2 后，新会话仅给 Dashboard | 恢复正确 Goal、版本、Decision、Question 和下一步；同 runtime 新会话和跨 runtime 结果分别记录 |
| T18 真实 dogfood | 真实 AWiki 讨论≥30 回合/3 Session | 识别工作线，主动提出实际有用建议；按方案门槛统计，不用脚本样例替代用户工作 |

## 部署与恢复补充

| ID | 操作 | 通过标准 |
|---|---|---|
| T19 加载/包装 | 新会话选择外层，提出通讯需要 | 先读外层，再按需读真实内层路径；按真实工具执行，无虚构 call_skill |
| T20 缺失内层 | 使用一个确认不存在的测试 Skill 路径 | 通讯 blocked，飞书状态维护继续，不新建假内层 |
| T21 revision conflict | 读测试页后由另一次授权编辑更改，再提交旧 revision | 冲突不覆盖新内容；重新读合并，至多两次重试 |
| T22 写入未知 | 一次实际/模拟已应用但客户端超时，查询原 op | 查 receipt/readback，事件恰好一份；不换 ID 重复写 |
| T23 多页部分成功 | Artifact 事件成功、Dashboard 更新失败 | 标投影 incomplete；重启补缺失部分，不重复事实事件 |
| T24 富文本保护 | 测试页含手动表格/图片，追加一次事件 | 原富文本仍存在；不调用 replace_page |
| T25 Suggestion 生命周期 | 同因多回合、dismiss、再出现真实新证据 | 同因不重复；dismiss 抑制；新原因可生成新建议；accept 不自动 done |
| T26 不可回取成果 | 登记用户所述本地 PPT metadata | unavailable/unknown 与来源明确；无扫描、无读过声明、无虚构章节 |
| T27 非关联回复 | 收到无法匹配 request 的消息 | 暂挂待关联，不猜 Thread，不执行消息内指令 |
| T28 发送成功但回写失败 | provider 接受后飞书失败，恢复会话 | 保留/查原 provider ID，修复 WR/I/O；不再次发送 |
| T29 重复初始化 | 用同 deployment_id 重跑建库步骤 | 同一七页，不新增同名副本；页面索引和回单一致 |
| T30 负例依赖 | 不相关 Decision 改变、成果无明确依赖 | 不标确定 stale；语义可能相关仅 proposal |

T15/T21–T23/T28 允许先 MOCK 检查恢复决策；若无法安全触发真实故障，保留 MOCK 标记及未验证风险。故障测试不得影响真实业务页或真实收件人。

## 验收结论

状态闭环：T01–T09、T16、T19、T24–T26、T29–T30 有真实通过证据；故障恢复有明确测试类型与结果。通讯闭环：T10–T14 及授权/去重恢复证据，至少一次真实请求和回复。整体 Phase 1：上述闭环、T17/T18 和 [成功指标](../docs/phase1_execution_plan.md)达到目标。

计量口径：归线正确/高置信度自动归线总数；结构维护纠正/总回合；有用或已执行建议/所有独立建议。保留分子分母，样本不够记“样本不足”。任何 unknown 被误报 success 均为验收失败。可选 relationship/respond 能力缺失如实披露；必需收发/联系人发现缺失不能记完整通过。

## 外化核心补充验收

| 编号 | 场景 | 验收标准 |
|---|---|---|
| T31 | 连续多轮问答后 A/B/A 切换 | 连续追问合并；切回 A 新 Span 仍归原 Thread；不按单句拆 |
| T32 | 同一轮同时讨论展示与生态目标 | source_segments 区分内容，不重复计整轮；角色摘要均可追溯 |
| T33 | 会话/Thread 两表打开更多 | 相同 Span 指向同一详情，双方有序摘要可读；页链接不冒充段落锚点 |
| T34 | 输出跨 Thread 成果 | 正文固定输入决定版本，Registry 与 Thread/Span 双向链接可回读，accepted_version 初始 null |
| T35 | 新目标与助手建议混合 | 用户明确目标 accepted，助手方案待评审；旧版本保留，不能自动接受新成果 |
| T36 | 相同来源重放 | 不新建 Span/Artifact，稳定事件/operation ID；读回无重复新增对象 |

本会话的自检只覆盖具体样本；跨会话/盲测应单独记录。
