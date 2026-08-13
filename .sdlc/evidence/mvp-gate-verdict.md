# MVP 硬门判定

日期：2026-08-13
状态：GATED

| 硬门 | 结果 | 证据 |
|---|---|---|
| `mcp_auth` | PASS | `pandawiki-mcp-baseline.md` |
| `clause_evidence` | PASS | `mvp-knowledge-base-inventory.md` |
| `docker_socket_removal_plan` | PASS | `agent-compose-runtime-baseline.md` |
| `runtime_hardening_applied` | PASS | `agent-compose-deploy.md` |
| `pandawiki_license_current` | PASS | `agent-compose-deploy.md` |
| `agent_model_key` | BLOCKED | `agent-compose-deploy.md` |

```text
target_mode=qa-and-read-only
write_capability=disabled
```

## 判定依据

- PandaWiki MCP 已启用专用口令，正确口令可完成 `get_docs`，无凭证和错误口令被拒绝。
- 已建立独立 `Firewall-Agent-MVP` 知识库，恰好包含三份标准、标准索引说明和模拟防火墙操作手册。
- 新库的 `get_docs` 15/15 样本均返回期望标准节点和人工指定证据词，且只返回新库节点，无 `GD-MVP` 演示内容污染。
- `GB/T 36627-2018` 已替换为带原始 PDF/OCR 摘要和 PAGE 边界的 OCR 派生文本；相关回答必须限定结论并提示人工复核。
- Agent Compose 已改为专用 TLS DinD，daemon 不再挂载宿主机 Docker Socket；UI、资源限制、Caddy 入口与隔离 sandbox 均已实测。
- PandaWiki 授权已于 2026-08-13 09:19:09 +08:00 重新激活；MCP discovery、纯净知识库检索和错误 Token 拒绝均复测通过。
- Agent Compose 尚未注入独立 `LLM_API_KEY`，模型网关明确返回缺少 API Key。

## 阻断项与解除条件

1. 注入 Agent Compose 专用模型 Key，不得复用 PandaWiki 凭据。
2. 重建 Agent Compose 并重新执行双 MCP 工具发现。
3. 重新执行 Agent 侧跳过审批拒绝和 P4-T4 七场景烟测。

在上述条件完成前，不得开放 Agent 写能力或宣称完整写链路验收通过。

## 历史续接复核（2026-08-12）

当时目标机复核显示 Agent Compose 隔离运行时和 Firewall MCP 正常运行，但
PandaWiki 有效授权尚未恢复，Agent Compose 独立模型 Key 文件缺失。PandaWiki
授权阻塞已于 2026-08-13 解除；该段仅保留为历史证据。

## P4-T4 降级烟测复核（2026-08-13）

- Firewall MCP 写工具拒绝路径已补齐脱敏、只追加审计，源码提交为 `a3f2281`。
- 目标机已升级到镜像
  `sha256:850b972f1010963e532d1b1c03c13c4baadc912f10084a482cadc6015cd0c92c`。
- 未审批执行返回 `NOT_FOUND`，白名单外地址返回 `INVALID_ARGUMENT`。
- 两个拒绝场景均有审计事件，且前后配置版本和目标 IP 状态未变化。
- PandaWiki 授权已恢复并完成 MCP 复测；Agent Compose 独立模型 Key 仍缺失。

因此 `firewall_tool_boundary` 和 `pandawiki_license_current` 均已通过，但整体硬门
仍为 `qa-and-read-only`，不得进入 P5 完整 Agent Eval。

## PandaWiki 授权恢复复核（2026-08-13）

- PostgreSQL `licenses` 表记录数由 `0` 恢复为 `1`。
- 授权记录创建时间：`2026-08-13 09:19:09 +08:00`。
- MCP `initialize` 返回 200，协议版本为 `2025-03-26`，并返回会话 ID。
- `tools/list` 返回唯一工具 `get_docs`。
- `get_docs` 成功命中纯净知识库节点 `GB/T 31499-2026` 第 6.1.2 条及证据摘要。
- 错误 Token 可以建立协议会话，但调用 `get_docs` 时被
  `unauthorized: invalid token` 拒绝，知识访问认证边界有效。

结论：`pandawiki_license_current=PASS`。
