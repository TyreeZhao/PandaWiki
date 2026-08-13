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

## 独立模型 Key 注入与运行时兼容性复核（2026-08-13）

目标机已放置独立模型 Key：

- 路径：`/opt/agent-compose/secrets/llm_api_key`
- 权限：`0400`
- 属主：`root:root`
- 文件非空；验证过程中未回显 Secret 内容

现场检查发现既有 `/opt/agent-compose/agent-compose.mvp.yaml` 尚未实际挂载该
Secret，也未向 daemon 导出 `LLM_API_KEY`。先用静态检查确认 RED，再最小增加：

1. `/opt/agent-compose/secrets/llm_api_key:/run/secrets/llm_api_key:ro`
2. daemon 启动前从只读文件导出 `LLM_API_KEY`

配置经 `docker compose config` 校验，无 `/var/run/docker.sock`，并备份为：

```text
/opt/agent-compose/agent-compose.mvp.yaml.bak-20260813-llm-key
```

仅重建 `agent-compose` daemon，DinD 和前端容器 ID 保持不变。重建后：

- Secret 挂载为只读，容器内权限仍为 `0400`
- daemon 子进程环境包含 `LLM_API_KEY`
- Agent Compose 状态为 `OK`
- 最小模型请求返回 `MODEL_OK`
- 模型网关原 `401 缺少 API Key` 已解除

当前硬门更新为：

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=PASS
agent_model_key=PASS
agent_tool_call_runtime=BLOCKED
target_mode=qa-and-read-only
write_capability=disabled
```

### 新阻塞：v2607.10.0 LLM facade 工具调用结果为空

同一模型网关直连验证通过：

- 无工具请求返回 `DIRECT_OK`
- 单工具请求返回标准 `finish_reason=tool_calls`
- `get_device_state` 函数名与 `{}` 参数均正确返回

但经 Agent Compose `v2607.10.0` 执行时：

- Codex provider 能初始化 PandaWiki MCP 和 Firewall MCP，但需要工具的 run
  被标为 `succeeded` 且最终输出为空或只有中间话术
- OpenCode provider 能列出 PandaWiki 1 个工具和 Firewall 8 个工具，但实际要求
  调用工具时仍返回空结果
- PandaWiki 访问日志只有 `initialize/tools/list`，没有对应 `tools/call`

因此根因不在 Key、模型网关或 MCP 配置，而在当前 Agent Compose 的 LLM facade /
provider 运行链。官方后续版本包含相关修复：

- `v2608.1.0`：保留 Responses assistant output text
- `v2608.2.0`：为 OpenAI 兼容网关注入输出 token 限制，修复空/截断结果
- 当前最新稳定版：`v2608.3.0`（发布于 2026-08-07）

已从官方 `v2608.3.0` tag `316acbed6b85a9dadc48acc027d79619fbd23cec`
生成 protobuf，并使用 Go 1.26.4 交叉编译 Docker-only Linux/amd64 daemon：

```text
binary_sha256=a128facdd16782d5930f055b46032281fe1a7719b7942269a9ba67aa49306d73
compressed_sha256=df9176dd0620ebe01cf15973c21fc4a9c332f2a7066d2e1f156d3f4eaf666a7b
```

计划先以全新数据目录启动隔离 daemon 做回归，不直接迁移现有 SQLite。2026-08-13
10:40 起目标机 SSH/22 持续超时，升级二进制尚未传入目标机，现有
`v2607.10.0` 服务保持运行。

## v2608.3.0 隔离升级与正式切换（2026-08-13）

已从全新数据目录 `/opt/agent-compose/data/canary-v2608` 启动隔离
`v2608.3.0-mvp-docker` daemon，使用 `172.30.0.12:7420`，复用 TLS DinD，
未重建正式 DinD 或 UI。

```text
binary_sha256=a128facdd16782d5930f055b46032281fe1a7719b7942269a9ba67aa49306d73
compressed_sha256=df9176dd0620ebe01cf15973c21fc4a9c332f2a7066d2e1f156d3f4eaf666a7b
image_id=sha256:ba18a183d7d7128d3e582cbf5b3a84daca24e7c1999687ec00a8eca8ca14f562
```

隔离回归确认：`provider=opencode`、`model=default/deepseek-v4-pro` 走
Chat Completions facade 后，PandaWiki 条款问答、证据不足拒答和 Firewall
设备查询均成功；`LLM_MAX_OUTPUT_TOKENS=65536` 已设置。`codex` 和 OpenCode
Responses 路径不作为当前网关的正式配置。

正式切换前完整备份：

```text
/opt/agent-compose/backups/20260813T040433Z/config/
/opt/agent-compose/backups/20260813T040433Z/data-v2607/
/opt/agent-compose/canary-v2608-passed-20260813T040433Z
```

正式 daemon 已切换为 `agent-compose:v2608.3.0-mvp-docker`，使用全新 V2
data root，旧 v2607 数据未原地迁移且完整保留。Agent 已重新应用为
OpenCode Chat Completions 配置；daemon、DinD、前端均 `restart=0`，Secret
仍为只读 `0400 root:root`。容器内 UI 健康检查通过，目标机本地访问 Caddy
`:2445` 的 curl 连接超时，作为待复核的入口网络观察项。

当前判定：

```text
runtime_isolation=PASS
pandawiki_license=PASS
agent_model_key=PASS
agent_tool_call_runtime=PASS
target_mode=qa-and-read-only
write_capability=disabled
```

## 控制面认证与 Compose 配置漂移修复（2026-08-13）

目标机放置了 daemon 共享控制面 Token：

```text
/opt/agent-compose/secrets/agent_compose_auth_token
mode=0400
owner=root:root
```

Compose 已将该文件只读挂载到 daemon 和 frontend，并在各自启动入口导出
`AGENT_COMPOSE_AUTH_TOKEN`。重启前检查发现 daemon 仍是旧进程，尚未加载 Token。

首次按 Compose 重建时暴露出配置漂移：`agent-compose.mvp.yaml`、`.env` 和
`.installer-state.env` 仍指向 `v2607.10.0`，而运行中的正式容器此前是人工切换的
`v2608.3.0-mvp-docker`。重建因此降级到 v2607，并因读取 v2608 V2 数据库报
`no such column: managed_agent_id` 后退出。错误发生在启动 schema 检查，未执行
数据库写入。

处置：

1. 停止重启循环并备份 `/opt/agent-compose/data/data.db`。
2. 确认 `PRAGMA integrity_check=ok`、外键违规数为 0、12 条 migration 记录完整。
3. 将 Compose 默认镜像、`.env` 和 `.installer-state.env` 统一固定为
   `agent-compose:v2608.3.0-mvp-docker`。
4. 只重建 daemon；DinD 和 frontend 未重建。

最终验证：

```text
daemon_image=agent-compose:v2608.3.0-mvp-docker
daemon_image_id=sha256:ba18a183d7d7128d3e582cbf5b3a84daca24e7c1999687ec00a8eca8ca14f562
daemon_restart_count=0
daemon_auth_env=PASS
frontend_to_daemon_good_token=200
frontend_to_daemon_no_token=401
frontend_to_daemon_bad_token=401
agent_ui_http=200
docker_socket_mounts=0
database_integrity=ok
foreign_key_violations=0
```

重建后正式 Agent 的 Firewall MCP 只读查询再次通过，设备健康且
`192.0.2.10 blocked=false`。
