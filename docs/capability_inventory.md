# 能力清单（2026-09-11 初始证据）

| 项目 | 已知结果 | 待本地验证 |
|---|---|---|
| GitHub repo | 初始为空，无既有代码/接口可继承 | 正常 clone/pull |
| mac 飞书 read_page | 当前连接器实读 P0 页面成功，revision 7 | Windows 当前连接成功 |
| append_markdown | 已读声明：page_ref、expected_revision、operation_id、markdown | 专用页真实 guarded write/receipt/readback |
| create_child_page | 已读声明：operation_id、parent_ref 或 root_alias、title | allowlist 内建页与重复执行 |
| get_sync_receipt | 已读声明：operation_id | 实际 receipt 字段、重试语义 |
| identity / roots / tree | 当前目录暴露，未在本次执行初始化 | 真实根与能力范围 |
| 内部 AWiki Skill | 已从官方 stable 安装 awiki-cli Skill，frontmatter name=awiki | 新任务发现与真实通讯调用 |
| AWiki 通讯 | 已读官方 Skill 通讯/People 命令，待 CLI schema/实际收发核验 | contacts/relationship/send/updates/respond 与回执 |
| 本地 Codex | 官方部署说明已核验 | Windows 版本、Skill 发现与 MCP 配置 |

以上“已读声明”不表示对应写入测试通过。本次已整理仓库并在 Windows 安装内外层 Skill，不初始化飞书状态库或外发消息。

## 本地执行者填写（脱敏）

| 语义 | 真实工具/Skill 章节 | 必要参数与返回字段 | 状态/证据 |
|---|---|---|---|
| feishu.read | 待填 | 待填 | NOT_RUN |
| feishu.create | 待填 | 待填 | NOT_RUN |
| feishu.append | 待填 | 待填 | NOT_RUN |
| feishu.receipt | 待填 | 待填 | NOT_RUN |
| communication.list_contacts | 待填 | 待填 | NOT_RUN |
| communication.get_relationship | 待填 | 待填 | NOT_RUN |
| communication.prepare_request | 外层本地整理草稿 | WR / scoped context | 待测试 |
| communication.send_request | 待填 | 待填 | NOT_RUN |
| communication.get_updates | 待填 | 待填 | NOT_RUN |
| communication.respond | 待填 | 待填 | NOT_RUN |

只记录必要能力结论和脱敏证据；不放 token、联系人真实地址或私密消息。


## Windows 本地执行结果

- 官方 Skill 发布包校验：sha256 `fe9357d71e2f2efa84de161e5a480207e67eb44aaf99473802a6a5298e3498ed`；16 个 Markdown 文件，无附带可执行脚本。审阅范围覆盖安装、身份、消息、People、runtime 等。身份/通讯/配置操作需按本次授权范围执行，不能把 Skill 内其他能力当作额外授权。
- 官方安装器已安装内层 awiki-cli（Skill name=awiki）；外层以 Windows 用户 Skill 目录 Junction 指向仓库完整目录。本会话已读取源码；新任务自动发现仍待验证。
- 既有 CLI 1.0.12 触发 version_unsupported，已通过官方 stable 包更新到 1.0.48；version、status、schema msg send 返回 ok=true。
- 当前新版 CLI 工作区 active_identity=null、identity_count=0；未执行身份注册、恢复或迁移。真实收发为 BLOCKED_MISSING_IDENTITY；不能把 CLI 安装成功当作身份可用。
- msg.send schema 已确认 to/text/client-message-id/idempotency-key；未真实发送，不宣称幂等投递通过。
- npm 更新退出成功，但提示旧临时目录中 exe 清理遇到 EPERM；新版命令可执行，未强制删除旧占用文件。
- Windows → Mac MCP read_page 已实读 P0 revision 7。未新增飞书运行页；写入/收发/dogfood 待执行。
