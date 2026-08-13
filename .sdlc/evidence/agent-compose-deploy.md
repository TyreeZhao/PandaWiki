# Agent Compose P4-T3 部署证据

日期：2026-08-12
状态：PARTIAL（隔离运行时、Agent 配置和 PandaWiki 授权/MCP 已完成；独立模型 Key 硬门未满足）

## 部署结果

- 目标机：`root@10.2.138.74`
- Agent Compose：`v2607.10.0`
- Agent：`firewall-assistant`
- 模型：`deepseek-v4-pro`
- UI：`https://pandawiki.docs.baizhi.cloud:2445/`
- 审批页：`https://pandawiki.docs.baizhi.cloud:2444/firewall-agent/approvals/`

目标机运行状态：

- `agent-compose-docker`：running，health=healthy
- `agent-compose`：running，restart=0
- `agent-compose-frontend`：running，restart=0
- Agent Compose UI 经 Caddy 返回 HTTP 200
- 审批页未认证访问返回 HTTP 401

## Docker 隔离

采用专用 DinD 作为 Agent Compose 的 Docker driver：

- Agent Compose daemon 不挂载 `/var/run/docker.sock`
- daemon 使用 TLS 连接 `tcp://docker:2376`
- 运行时探针返回 `store=docker`、`available=true`
- DinD 通过独立 control 网络别名 `docker` 提供服务
- Agent sandbox 创建在 DinD 自己的 Docker bridge 中，不属于宿主机 Docker
- DinD 的 privileged 权限仅存在于独立 daemon 容器内
- daemon、UI 和 DinD 均有内存/PID 限制
- 目标机已有 2 GiB Swap

前端镜像启动时需要调整 Nginx 目录所有权。配置先 `cap_drop: ALL`，再仅恢复：

- `CHOWN`
- `SETGID`
- `SETUID`

本地使用相同前端镜像验证该最小 capability 集合后，容器持续运行且原 `chown ... Operation not permitted` 错误消失。

## 网络与入口

- control 网络为 `internal: true`，固定网段 `172.30.0.0/24`
- daemon 固定地址：`172.30.0.10`
- UI 固定地址：`172.30.0.11`
- UI 不发布宿主机端口，由 Caddy 直接反代 `172.30.0.11:8000`
- daemon 额外加入独立 egress 网络，仅用于访问模型网关
- Firewall MCP 与 PandaWiki 网络保持独立，Agent 不直接访问 Firewall SQLite 或审批接口

## 配置与工具边界

Agent Compose `config` 使用真实 `v2607.10.0` 二进制通过：

- 单一 Agent：`firewall-assistant`
- PandaWiki MCP：纯净知识库入口 `http://10.2.138.74:18080/mcp`
- Firewall MCP：`http://169.254.15.20:8080/mcp`
- 配置规范化输出会掩码两个 MCP Token
- system prompt 明确知识/设备状态边界、两阶段审批、超时先查状态、证据不足拒答和 Prompt Injection 不得改变规则

Firewall MCP 真实协议验证：

- `initialize` 成功
- `tools/list` 恰好返回 8 个允许工具
- `get_device_state` 成功且不修改配置
- 使用不存在变更和无效审批码直接调用 `apply_approved_change` 被拒绝

工具清单：

1. `get_device_state`
2. `list_security_policies`
3. `get_ip_block_status`
4. `simulate_traffic_match`
5. `get_change_status`
6. `list_audit_events`
7. `prepare_change`
8. `apply_approved_change`

批准接口、Shell、SQL、Docker 和通用配置接口未注册为 Agent 工具。

## 历史阻塞记录（2026-08-12）

### PandaWiki 授权

PandaWiki MCP 在 2026-08-12 11:02 前仍可完成纯净知识库检索。API 日志显示：

```text
2026-08-12T13:27:02.814+08:00 DELETE /api/v1/license -> 200
```

随后许可证缓存回落为 edition 0。当前使用与纯净知识库 App 设置完全一致的专用 MCP 口令访问 `18080/mcp`，仍在协议处理前返回：

```text
403 Feature not available in current edition
```

因此当时的问题不是 Agent MCP URL、Token 或启动方式错误，而是 PandaWiki
授权被明确删除后触发版本能力门。该阻塞已于 2026-08-13 解除，见文末恢复复核。

### 模型凭据

Agent Compose 已配置独立模型 endpoint 和模型名，Compose 通过只读
`/run/secrets/llm_api_key` 注入 `LLM_API_KEY`，缺少 Secret 文件时启动 fail closed。
目标机当前尚未提供 `/opt/agent-compose/secrets/llm_api_key`。模型网关在无 Key 时返回：

```text
401 缺少 API Key
```

首次 Agent run 已成功创建 DinD sandbox 并拉取 guest 镜像，但模型代理请求因该凭据缺失返回 502。失败 run 已停止并清理。

## 验证

本地：

- `go test -race ./...`：PASS
- `go vet ./...`：PASS
- `sh deploy/config_test.sh`：PASS
- `sh deploy/agent-compose-config-test.sh`：PASS
- `git diff --check`：PASS

远端：

- 三个 Agent Compose 容器稳定运行：PASS
- 无宿主机 Docker Socket：PASS
- DinD TLS runtime：PASS
- Caddy UI 入口：PASS
- Firewall MCP 8 工具与未审批拒绝：PASS
- PandaWiki MCP：BLOCKED（2026-08-12 当时许可证已删除；现已恢复）
- Agent 模型调用：BLOCKED（缺少独立模型 Key）

## 历史判定（2026-08-12）

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=BLOCKED
agent_model_key=BLOCKED
target_mode=qa-and-read-only
write_capability=disabled
```

该判定记录 2026-08-12 的现场状态。PandaWiki 授权已于 2026-08-13 恢复并完成
`initialize -> notifications/initialized -> tools/list -> get_docs` 复测；当前只剩独立
`LLM_API_KEY`。

## 历史续接复核（2026-08-12）

- 目标机 SSH 可达。
- `agent-compose-frontend`、`agent-compose`、专用 DinD、Firewall MCP 和 PandaWiki 核心容器仍在运行。
- `/opt/agent-compose/secrets/llm_api_key` 仍不存在，`agent_model_key=BLOCKED`。
- 未检测到已恢复的 PandaWiki 有效授权，`pandawiki_license=BLOCKED`。
- 本次未读取或输出任何 Secret，也未重建容器。

结论保持不变：

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=BLOCKED
agent_model_key=BLOCKED
target_mode=qa-and-read-only
write_capability=disabled
```

## PandaWiki 授权恢复复核（2026-08-13）

PandaWiki PostgreSQL `licenses` 表已恢复为 1 条授权记录，创建时间为
`2026-08-13 09:19:09 +08:00`。使用 Agent Compose 项目中既有的 PandaWiki
专用 MCP Token 执行现场复测：

- `initialize`：HTTP 200
- MCP 协议版本：`2025-03-26`
- 会话 ID：已返回
- `tools/list`：唯一工具为 `get_docs`
- `get_docs`：成功命中纯净知识库的 `GB/T 31499-2026` 第 6.1.2 条及条款证据
- 错误 Token：在 `get_docs` 调用阶段返回 `unauthorized: invalid token`

当前判定更新为：

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=PASS
agent_model_key=BLOCKED
target_mode=qa-and-read-only
write_capability=disabled
```

目标机 `/opt/agent-compose/secrets/llm_api_key` 仍不存在，当前执行环境也没有可供
独立注入的模型 Key。不得复用 PandaWiki 模型凭据。模型 Key 就绪前不重建
Agent Compose，不宣称 Agent 端完整链路可用。
