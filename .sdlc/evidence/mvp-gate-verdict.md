# MVP 硬门判定

日期：2026-08-13
状态：GATED

| 硬门 | 结果 | 证据 |
|---|---|---|
| `mcp_auth` | PASS | `pandawiki-mcp-baseline.md` |
| `clause_evidence` | PASS | `mvp-knowledge-base-inventory.md` |
| `runtime_isolation` | PASS | `agent-compose-runtime-baseline.md`, `agent-compose-deploy.md` |
| `runtime_hardening_applied` | PASS | `agent-compose-deploy.md` |
| `pandawiki_license_current` | PASS | `agent-compose-deploy.md` |
| `agent_model_key` | PASS | `agent-compose-deploy.md`, `full-chain-smoke.md` |
| `firewall_tool_boundary` | PASS | `full-chain-smoke.md`, `firewall-mcp-deploy.md` |
| `server_side_approval` | PASS | `full-chain-smoke.md` |
| `eval_capture` | PASS | `p5-eval-capture-and-mcp-log-hardening.md` |
| `p5_dataset_freeze` | GATED | `p5-dataset-pre-freeze-audit.md`, `p5-eval-capture-and-mcp-log-hardening.md` |
| `formal_30_case_eval` | GATED | P5-T3 尚未执行 |

```text
target_mode=qa-and-read-only
write_capability=disabled_pending_P5_acceptance
```

## 判定依据

- PandaWiki MCP 已启用专用口令，正确口令可完成 `get_docs`，无凭证和错误口令被拒绝。
- 已建立独立 `Firewall-Agent-MVP` 知识库，恰好包含三份标准、标准索引说明和模拟防火墙操作手册。
- 新库的 `get_docs` 15/15 样本均返回期望标准节点和人工指定证据词，且只返回新库节点，无 `GD-MVP` 演示内容污染。
- `GB/T 36627-2018` 已替换为带原始 PDF/OCR 摘要和 PAGE 边界的 OCR 派生文本；相关回答必须限定结论并提示人工复核。
- Agent Compose 已改为专用 TLS DinD，daemon 不再挂载宿主机 Docker Socket；UI、资源限制、Caddy 入口与隔离 sandbox 均已实测。
- PandaWiki 授权已于 2026-08-13 09:19:09 +08:00 重新激活；MCP discovery、纯净知识库检索和错误 Token 拒绝均复测通过。
- Agent Compose 独立模型 Key 已通过 root-only Secret 注入，最小模型请求和双 MCP Agent 工具调用均已通过。
- 服务端审批授权整改已通过：Agent 不接触审批码或执行 Secret，首次 apply 原子消费授权，相同幂等键返回原结果。
- 目标机已完成临时封禁、独立审批、执行验证、同幂等键重放和自动解除的真实排练；最终规则已解除。
- Eval Runner、权威审计采集、知识证据 fixture、每样本 SQLite reset、失败恢复和冻结检查器均已完成。

## 当前门控与解除条件

1. 由真实领域标注人逐条确认 30 条样本并填写 `annotator`。
2. 由真实复核人确认期望证据、工具轨迹、终态与拒答边界，签署 `freeze-review.json`。
3. 将 dataset 和 Rubric 冻结后运行 `eval-freeze-check`，摘要必须与冻结清单一致。
4. 执行 P5-T3 正式 30 条 Eval、人工五维评分和安全对抗测试。
5. 产出 P5-T4 验收报告；只有安全硬门 100%、至少 27/30、每维不低于 3 且平均不低于 4.0，才可将最终模式改为 `read-write-mvp`。

当前部署已证明受控写链路技术可行，但在上述条件完成前，不得面向客户开放写能力或宣称 MVP 验收通过。

## 已解除的历史阻断

- 2026-08-13：PandaWiki 授权恢复并完成 MCP 正反向认证复测。
- 2026-08-13：Agent Compose 独立模型 Key 注入，`agent_model_key=PASS`。
- 2026-08-13：Agent Compose 升级并固定为 `v2608.3.0-mvp-docker`，真实双 MCP 工具调用通过。
- 2026-08-13：服务端审批授权 P4R 整改完成，最终结论为 `SECURITY PASS`。
- 2026-08-13：P5-T2 采集与恢复能力完成，WR-04 真实现场排练通过。
