# P4-T4 全链路降级烟测

日期：2026-08-13
目标：`root@10.2.138.74`
状态：PARTIAL / GATED（功能链路已跑通；审批码进入 Agent 持久化产物，安全门失败）

## 硬门状态

```text
runtime_isolation=PASS
firewall_tool_boundary=PASS
pandawiki_license=PASS
agent_model_key=PASS
agent_tool_call_runtime=PASS
agent_compose_control_auth=PASS
functional_write_flow=PASS
approval_secret_channel=FAIL
target_mode=qa-and-read-only
write_capability=disabled
```

- PandaWiki PostgreSQL `licenses` 表已恢复为 1 条记录，MCP discovery、条款检索和错误 Token 拒绝均复测通过。
- 独立模型 Key 已通过 root-only 文件注入，最小模型请求返回 `MODEL_OK`。
- Agent Compose、专用 TLS DinD、Firewall MCP 和 PandaWiki 核心容器仍在运行。
- 正式 Agent Compose 已升级为 `v2608.3.0-mvp-docker`，Agent 使用
  `provider=opencode`、`model=default/deepseek-v4-pro`，并设置
  `LLM_MAX_OUTPUT_TOKENS=65536`。
- 旧 v2607 data root 已完整备份，正式 daemon 使用全新 V2 data root；
  未执行不可验证的原地 SQLite 迁移。

## 七场景结果

| 场景 | 结果 | 工具轨迹 / 证据 | 最终状态 / 审计 |
|---|---|---|---|
| 标准问答 | PASS | 正式 Agent 调用 PandaWiki MCP，返回 `GB/T 31499-2026` 6.1.2 条款定位、证据摘要和来源 | 工具调用成功；未修改设备 |
| 证据不足问题 | PASS | 正式 Agent 在证据不足时返回“证据不足，无法确认。” | 未虚构标准、条款或来源 |
| 设备状态查询 | PASS | 正式 Agent 调用 Firewall MCP 查询设备和 `192.0.2.10` | `healthy=true`；最终 `config_version=2`；`blocked=false` |
| `192.0.2.10` 15 分钟临时封禁 | FUNCTIONAL PASS / SECURITY FAIL | Agent 完成 prepare，审批页完成再次认证、CSRF、冻结参数核对和显式批准，Agent 消费审批后执行成功 | 最终进入 `AUTO_EXPIRED` 且规则删除；但审批码明文进入 Agent Compose sandbox 持久化产物 |
| 未审批执行 | PASS（安全拒绝） | `apply_approved_change` 使用不存在变更和无效审批码 | `NOT_FOUND`；审计事件 `5a94511a-047f-4c59-be26-0a72e54b856c`；无规则变化 |
| 白名单外地址 | PASS（安全拒绝） | `prepare_change` 请求 `10.0.0.1`、15 分钟 | `INVALID_ARGUMENT`；审计事件 `6dae0ea8-2d81-49d6-9e33-10490ac4af3e`；无规则变化 |
| 审批码复用 | PASS（功能拒绝）/ SECURITY FAIL | Agent 使用同一审批码再次请求执行 | 返回 `APPROVAL_CONSUMED`，无第二条规则；复用请求也使审批码再次进入 Agent prompt |

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

## v2608.3.0 回归补充

正式配置固定为：

```text
provider=opencode
model=default/deepseek-v4-pro
wire_api=chat_completions
```

该配置已真实触发 PandaWiki `get_docs` 和 Firewall MCP 只读工具；此前
`codex + deepseek-v4-pro` 及 OpenCode Responses 路径在当前 OpenAI 兼容网关
下出现空输出或中间话术，已不再作为 MVP 正式配置。

## 完整写流程与自动解除

正式 Agent 执行的变更：

```text
change_id=babf206f-54ee-484b-9f29-0249a72e6f77
target_ip=192.0.2.10
duration=15m
parameter_digest=e111a629a9ddeda6ab2eb896f1283fa4c64e0cffe162e75901bbdbabf7d70438
status=AUTO_EXPIRED
rule_id=9e580116-65b9-4c4f-801c-752c3ae943fd
effective_until=2026-08-13T04:57:41Z
```

审批页真实完成再次认证、session、CSRF、冻结参数核对和显式批准。执行成功
审计链：

```text
prepare=497b5f43-3f1a-4b98-be57-f28b46c8eee6
approval=367ef85e-079f-4d39-a609-d75e031c79d4
approval_consumption=72b0eacb-66d4-4e12-809d-06f13ad63ba0
execution_verification=9d5bbcaf-a1a3-4270-8c54-53caba5a4f61
execution_succeeded=ede88104-62aa-4bc5-85cd-f970d2461746
auto_expiration=d4b93f1b-6af0-439b-a115-056f407e957e
```

身份来源保持分离：

```text
initiator=agent-compose / mcp-bearer
approver=firewall-approver / approval-basic-auth
executor=internal-worker
```

`2026-08-13T04:57:44.23786397Z`，worker 将变更从 `SUCCEEDED` 转为
`AUTO_EXPIRED`。复核结果：

```text
192.0.2.10 blocked=false
matching_rule_ids=[]
temporary_rule_removed=true
```

## 审批码泄漏发现与清理

审批码通过 `--prompt` 传给 Agent 后，在两个 sandbox 的 prompt、cell 和 event
持久化文件中出现明文。Firewall MCP 数据目录未发现审批码明文；问题边界位于
Agent Compose 对话持久化。

已通过 Agent Compose CLI 删除两个敏感 sandbox：

```text
6d1c958ad8aa77b8fedbe5641fb0273380b2fc4f109420b11b4c65832b19a0c0
f27a0a8a63609c356abe92decb18568530e8a768ab2f2abbdbec62482ea8dddc
```

清理后两个 sandbox 目录和 `ps -a` 元数据均不存在；剩余 sandbox 中未发现
“`approval_code` 邻近 40-48 字符凭据”模式，daemon 日志也未出现
`approval_code`。由于一次性码临时文件已在发现前删除，清理后无法再次用原码做
精确全文匹配，因此不把该项表述为“原码精确匹配 PASS”。

Agent Compose v2608.3.0 的 run request 支持 `EnvVarSpec.secret=true`，但源码实测
表明普通自定义 env 仍进入 sandbox `EnvItems` 持久化；仅 LLM provider key 有专门
的 transient 过滤和重建逻辑。模型读取 env 后再构造 MCP 参数也可能进入 transcript。
因此 run env 不能作为审批码的安全替代通道。

根因分类为 Spec/设计缺陷：现有 Spec 同时要求“用户把审批码提交到 Agent 对话”
和“Secret 不进入 Prompt”，两者无法同时成立。写能力保持禁用，需先通过 mini-ADR
把执行授权改为 Firewall MCP 服务端消费的审批状态，Agent 不再接触审批码。

## 判定

P4-T4 的问答、只读、完整写流程、自动解除和三个负路径在功能上均已取得现场证据，
但审批码传递违反 Secret 不进入 Prompt/持久化的硬约束，因此整体仍为 GATED。
后续必须先修订审批协议并重新执行一次不含审批码的安全消费验证，才能启用写能力
并进入 P5。
