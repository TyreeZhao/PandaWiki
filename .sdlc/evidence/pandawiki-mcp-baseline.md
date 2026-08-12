# PandaWiki MCP 基线

日期：2026-08-12
状态：PASS（认证与协议已验证；知识证据质量见独立基线）

## 现场

- 目标：`root@10.2.138.74`
- PandaWiki 版本：`v3.86.4`
- 当前知识库：`GD-MVP`
- 知识库 ID：`1e10849c-e77e-4000-9caa-b38957213cb1`
- MCP 应用类型：`12`（`AppTypeMcpServer`）
- MCP 应用 ID：`3eec28a3-51f4-4370-8013-7a91911e7a42`

## 入口与协议

- `https://10.2.138.74:2443/mcp`：经 PandaWiki nginx，POST 返回 `405`；该端口不是当前 MCP 入口。
- `http://10.2.138.74/mcp`：经宿主机 Caddy 反代到 PandaWiki API，Streamable HTTP MCP 初始化成功。
- 初始化协议版本：`2025-03-26`
- 初始化响应包含 `Mcp-Session-Id`；后续请求必须携带该会话 ID。
- 正确调用顺序：`initialize` → `notifications/initialized` → `tools/call`。

## MCP 配置

已通过 PandaWiki 管理 API 只修改 MCP 应用配置：

- `is_enabled=true`
- Tool：`get_docs`
- Tool 描述：为解决用户的问题从知识库中检索文档
- `sample_auth.enabled=true`
- 访问口令保存在目标机 root-only 文件中，未进入本证据、日志或 Git。

## 认证验证

| 场景 | 工具调用结果 |
|---|---|
| 无认证 | 拒绝，`unauthorized: invalid token` |
| 错误 Bearer 口令 | 拒绝，`unauthorized: invalid token` |
| 正确 Bearer 口令 | 成功返回 `get_docs` 结果 |
| 正确原始 `Authorization` 口令 | 成功返回结果 |
| `X-API-Key` | 拒绝 |

认证硬门结论：`mcp_auth=PASS`。

## 工具清单

只有一个工具：

```text
name: get_docs
arguments: { message: string }
```

观察到的工具元数据为 `readOnlyHint=false`、`destructiveHint=true`、`openWorldHint=true`，与其实际只读知识检索用途不一致。该协议元数据问题必须在后续接入与评审中修正或由 Agent 侧显式限制，不能据此开放写能力。

## 重要限制

当前 `GD-MVP` 不是纯净知识库，检索会返回标准之外的演示、安装和模型接入节点；其中已有历史演示凭据/Token 文本。不得把该知识库直接接入客户 Agent 的生产问答链路。
