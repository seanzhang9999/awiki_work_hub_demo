# Golden Matter：AWiki 产品定义、PPT 与 MVP

测试场景，不代表已经发生的实际工作状态。示例 IDs 和版本必须在专用测试空间使用，不覆盖正式主方案。

Matter `awiki-workpod`；Goal：形成可对外说明、可持续迭代的产品方案、PPT 和 Skill 原型。

候选工作线：PPT Story、Session / Thread Model、Artifact Modularity、Agent Portability、Enterprise Positioning、MVP Skill、Work Communication、Work Observability。实际归线基于对话，不强制每线建记录。

## 最小输入序列

1. Session S1 / A1：“先讨论企业市场，核心客户可能是已有多个 Agent 的团队。”预期 Enterprise Positioning，candidate 判断。
2. S1 / B1：“转技术模型：一个会话应允许多个 Thread Span。”预期 Session / Thread Model。
3. S1 / A2：“回到市场定位，采用‘企业持有持续工作状态’作为这版判断。”预期回到 A，accepted Decision 带本回合来源。
4. 新 Session S2：“继续技术模块 B。”仅给 Dashboard；应恢复 B，不要求粘贴 S1 历史。
5. 用户登记：“测试 Deck V13，引用尚不可回取；采用 V13。其叙事依赖产品定义 D7 当前已接受版本。”元数据有用户来源，retrieval unavailable。
6. 用户接受：“D7 更新：Work Observability 是核心能力，Agent 应在几轮讨论后看出成果需要推进。”应生成 D7 新版本，Deck may_need_review。
7. 不要求更新 PPT，继续讨论下一问题。应主动建议 Review Deck，并解释依据；不能声称已读 V13 或指定页码。
8. 用户忽略建议，再闲聊三回合。不得重复。
9. 用户要求准备企业定位评审。内部 AWiki 实查联系人后，选定测试对象，准备 scoped WR；尚无发送授权。
10. 用户实际确认收件人/正文后才发送。获取真实 Reply 后回到原 Enterprise Positioning，必要时 questioned/ready 和新建议。

## 期望 Observatory 形态

```text
NOW：AWiki 产品定义与 MVP / Session Thread Model
最近变化：D7 更新（链接到事件）
成果：Deck V13，用户已接受，当前不可回取，may_need_review
等待：WR 的真实状态；无请求时明确无等待
需要人：建议采用/忽略或待确认判断
下一步：Review 产品定义相关叙事，依据 D7 变化
```

将当前真实讨论用作 T18 时，先从可访问来源核验版本，不能直接把该测试序列当作事实注入工作状态。
