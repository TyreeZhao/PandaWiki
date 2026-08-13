# P4-T4 全链路降级烟测

日期：2026-08-13
目标：`root@10.2.138.74`
状态：PARTIAL / GATED

## 硬门状态

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=BLOCKED
agent_model_key=BLOCKED
target_mode=qa-and-read-only
write_capability=disabled
```

- PandaWiki PostgreSQL `licenses` 表当前记录数为 `0`，MCP 仍被 edition 0 能力门拒绝。
- `/opt/agent-compose/secrets/llm_api_key` 不存在。
- Agent Compose、专用 TLS DinD、Firewall MCP 和 PandaWiki 核心容器仍在运行。
- 因模型和知识 MCP 硬门未满足，本次不启动 Agent 写流程，不伪造 UI 或模型结果。

## 七场景结果

| 场景 | 结果 | 工具轨迹 / 证据 | 最终状态 / 审计 |
|---|---|---|---|
| 标准问答 | BLOCKED | PandaWiki MCP 在协议处理前返回 edition 0 能力拒绝 | 无模型回答；不得伪造条款证据 |
| 证据不足问题 | BLOCKED | 同上，且 Agent Compose 缺少独立模型 Key | 无模型回答；拒答行为待凭据恢复后验证 |
| 设备状态查询 | PASS（MCP 直连） | `get_device_state`、`get_ip_block_status`、`simulate_traffic_match` | 设备 healthy；`config_version=0`；`192.0.2.10` 未封禁；只读调用有审计 |
| `192.0.2.10` 15 分钟临时封禁 | DISABLED | 写能力硬门未满足，未 prepare、未审批、未 apply | 无规则变化；未宣称 `AUTO_EXPIRED` |
| 未审批执行 | PASS（安全拒绝） | `apply_approved_change` 使用不存在变更和无效审批码 | `NOT_FOUND`；审计事件 `5a94511a-047f-4c59-be26-0a72e54b856c`；无规则变化 |
| 白名单外地址 | PASS（安全拒绝） | `prepare_change` 请求 `10.0.0.1`、15 分钟 | `INVALID_ARGUMENT`；审计事件 `6dae0ea8-2d81-49d6-9e33-10490ac4af3e`；无规则变化 |
| 审批码复用 | BLOCKED | 完整写流程按硬门禁用；确定性单元/集成测试已覆盖，但本次未做 Agent UI 现场执行 | 待凭据恢复后执行真实审批、首次消费和复用拒绝 |

## 现场不变量

拒绝测试前后：

```text
device_id=simulated-firewall-1
healthy=true
write_locked=false
config_version=0
target_ip=192.0.2.10
blocked=false
matching_rule_ids=[]
```

未审批执行审计仅记录：

- `attempted_change_id`
- 工具和阶段
- 身份来源
- 幂等键
- `result=rejected`
- `error_code=NOT_FOUND`

审计中不保存审批码。白名单外请求记录目标 IP、时长、幂等键和
`error_code=INVALID_ARGUMENT`。

## 烟测中发现并修复的问题

初次烟测发现写工具拒绝路径返回稳定错误码，但没有写入拒绝审计，不满足
Spec 5.11“所有变更和工具调用均可追溯”。

修复采用 TDD：

1. 新增白名单外 `prepare_change` 和未审批 `apply_approved_change` 的失败审计测试。
2. RED 证明两个拒绝路径缺少审计。
3. 增加统一拒绝审计边界；新增 `attempted_change_id` 保存不存在的目标变更 ID，同时保持 `change_id` 外键完整性。
4. 增量迁移版本 2 幂等执行，重复打开 SQLite 测试通过。
5. `go test -race ./...`、`go vet ./...` 和 `git diff --check` 通过。

源码提交：Firewall MCP `a3f2281`。

部署镜像：

```text
tag=firewall-mcp:mvp-a3f2281
image_id=sha256:850b972f1010963e532d1b1c03c13c4baadc912f10084a482cadc6015cd0c92c
platform=linux/amd64
```

升级前备份：

```text
/data/firewall-mcp/backup/firewall.db.pre-a3f2281.20260812T235942Z
```

升级后容器 healthy，SQLite `schema_migrations` 包含版本 `1` 和 `2`，
`audit_events.attempted_change_id` 已存在；只读根文件系统、256 MiB 内存和
128 PID 限制保持不变。

## 判定

P4-T4 已完成当前 `qa-and-read-only` 模式下可执行的真实烟测，但没有满足完整
Agent 全链路验收标准。恢复 PandaWiki 有效授权并注入独立模型 Key 后，必须重新执行：

1. 标准问答和证据不足拒答。
2. Agent 侧设备查询。
3. `prepare -> 独立审批 -> apply -> 验证 -> AUTO_EXPIRED`。
4. 未审批、白名单外和审批码复用三个 Agent 侧负路径。
