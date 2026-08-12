# MVP 硬门判定

日期：2026-08-12
状态：GATED

| 硬门 | 结果 | 证据 |
|---|---|---|
| `mcp_auth` | PASS | `pandawiki-mcp-baseline.md` |
| `clause_evidence` | PASS | `mvp-knowledge-base-inventory.md` |
| `docker_socket_removal_plan` | PASS | `agent-compose-runtime-baseline.md` |
| `runtime_hardening_applied` | PASS | `agent-compose-deploy.md` |
| `pandawiki_license_current` | BLOCKED | `agent-compose-deploy.md` |
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
- PandaWiki 授权于 2026-08-12 13:27:02 +08:00 被管理 API 删除，当前 MCP 被 edition 0 版本门拒绝。
- Agent Compose 尚未注入独立 `LLM_API_KEY`，模型网关明确返回缺少 API Key。

## 阻断项与解除条件

1. 恢复有效 PandaWiki 授权，并复测纯净知识库 MCP 工具发现和检索。
2. 注入 Agent Compose 专用模型 Key，不得复用 PandaWiki 凭据。
3. 重新执行 Agent 工具发现、跳过审批拒绝和 P4-T4 七场景烟测。

在上述条件完成前，不得开放 Agent 写能力或宣称完整写链路验收通过。
