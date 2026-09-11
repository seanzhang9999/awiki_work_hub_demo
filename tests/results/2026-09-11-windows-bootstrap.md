# Windows 本机部署验收

运行版本：AWiki CLI 1.0.48；外层 awiki-workpod，内层 awiki。部署 ID：workpod-win-20260911。

## 实际通过

- 当前 Codex 技能目录实际列出 awiki 与 awiki-workpod，外层 Junction 可读，内层引用完整。
- Windows Codex 成功调用 Mac 飞书 MCP，user 身份有效。
- 创建专用 P0 容器及七页，写入并逐页回读，七页 revision 均为 2。
- 真实会话以“部署 → 架构纠正 → 部署接续”记录三个 Span、两条 Thread；这是已有可见会话的回顾性归线，不冒充在线逐轮检测或跨 Session 测试。
- 用户明确的架构纠正存为 accepted Decision；仓库成果登记 observed commit，accepted_version=null。
- Dashboard 展示 NOW、Thread Timeline、成果、阻塞、等待及下一步，并保存页面索引和恢复快照。
- 相同 Matter 追加操作重放返回 idempotent_replay=true；回读 event_id 恰好出现一次，revision 仍为 2。
- 使用独立操作 ID 和过期 expected_revision=1 写入 revision=2 页面，返回 REVISION_CONFLICT，无覆盖。
- 使用原建库操作 ID 重放，返回原容器与 idempotent_replay=true。
- get_sync_receipt 对 Matter 初始化写入返回 succeeded，与回读结果一致。
- 本机配置已绑定七页和实际 MCP 工具，私密配置未提交 Git。

## 测试范围

T19 加载发现通过；T01/T05/T09 为本会话的实际场景覆盖，尚不宣称通用准确率。T21 覆盖冲突拒绝，未覆盖全部合并重试；T22 覆盖真实幂等回放，未注入网络超时；T29 覆盖容器重放，未完整重跑七页初始化。

T02/T03/T04/T06–T08/T15–T18/T20/T23–T28/T30 尚未完整执行。T10–T14 真实通讯被身份迁移阻塞。没有创建假回单或模拟消息，没有将 MOCK 标作集成成功。

## 通讯阻塞

当前新版租户工作区 identity_count=0。用户选择的旧身份别名存在于升级后的 legacy-archive，资料保留。旧版 id import-v1 dry-run 返回 legacy_plaintext_identity_storage_disabled。

进一步检查 `awiki-cli --migration id vault migrate --help` 与 schema：当前构建只有预检，不提供 CLI-safe standalone migration API，真实执行不会迁移身份文件。未降级存储模式、未复制明文密钥、未替换 DID、未创建身份、未外发消息。

后续应使用官方支持的同身份迁移/设备接入方式，或由用户选择恢复流程；不能以一个 Handle 代替身份认证。完整通讯通过后再验证授权发送、Reply 归线和成果影响。

## 结论

部署和飞书持久化基础检查 PASS；整体 Phase 1 PARTIAL。尚需身份接入、完整场景和至少3个 Session/30回合真实 dogfood。当前不能量化归线准确率或建议有用率。
