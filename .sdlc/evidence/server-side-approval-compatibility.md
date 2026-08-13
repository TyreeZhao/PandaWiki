# P4R 服务端审批授权兼容性证据

> 核验日期：2026-08-13
> 代码仓库：`/Users/zhaotong/Documents/chaitin/code/firewall-mcp`
> Firewall MCP 提交：`87b97b2`, `a50b9f7`, `8d91225`, `419c4a3`
> PandaWiki SDLC/ADR 提交：`8f34d4b6`

## 本地验证

- `go test -race ./...`：PASS
- `go vet ./...`：PASS
- `git diff --check`：PASS
- `sh deploy/agent-compose-config-test.sh`：PASS
- `FIREWALL_APPROVAL_BASE_URL=... sh deploy/config_test.sh`：PASS
- `FIREWALL_APPROVAL_BASE_URL=... docker compose -f deploy/compose.yaml config`：PASS
- Agent 配置、系统提示词和 README 静态扫描：未发现 `approval_code`、`ApprovalCode` 或“一次性审批码”。
- 正式 Agent 模板固定为已验证的 `provider=opencode`、`model=default/deepseek-v4-pro`。

## 迁移验证

- 旧 SQLite `approvals.code_hash` 在 migration 3 后已移除。
- 迁移前存在的审批统一转换为 `LEGACY_INVALID`，不能继续执行。
- 变更的发起身份从最早的 `prepare_change` 审计回填。
- 重复打开数据库不会重复记录 migration 3。
- `PRAGMA foreign_key_check` 结果为空。
- 服务端授权按 `change_id`、参数摘要、发起身份、审批身份和有效期绑定。
- 并发消费测试证明只有一个请求进入执行，其他请求返回已消费错误；同幂等键重试返回已保存结果。

## 当前结论

P4R-T1、P4R-T2 及 P4R-T3 的代码、配置和升级兼容性门通过。

## 远端升级与全链路烟测

- 目标机：`root@10.2.138.74`
- 数据库备份：`/data/firewall-mcp/backups/20260813T063643Z/firewall.db`
- Firewall MCP 镜像：`firewall-mcp:mvp-server-auth`
- 镜像摘要：`sha256:f42f8eb9dc2bddd9150456a691b9911bbba80b132d388ba604d7865859a48e2f`
- 变更 ID：`0b485a01-d490-4eba-9887-268dc736d04c`
- 目标：`192.0.2.10`
- 时长：5 分钟
- 执行规则：`324637fd-c981-429b-9783-f93c0cf3eabf`
- 最终状态：`AUTO_EXPIRED`
- 最终 IP 状态：`blocked=false`，无匹配规则

审批页面真实完成 Basic Auth、独立 session、CSRF 和显式批准，批准响应只包含
`change_id/status/approved_at/expires_at`，没有执行 Secret。随后
`apply_approved_change` 仅使用 `change_id + idempotency_key` 进入 `SUCCEEDED`。
相同幂等键重放返回同一规则和终态，不同幂等键返回 `APPROVAL_CONSUMED`。

远端 SQLite 实测：

```text
integrity_check=ok
foreign_key_violations=0
migrations=1,2,3
approval_secret_columns=[]
```

Agent Compose 正式配置升级为 revision `2`，spec hash 为
`sha256:2c4f358f4eec7b98e2fff0dd71dbd391c6788f7efbf18551db094bad96dfab28`。
新 sandbox 明确回答用户无需提供审批码或执行 Secret，apply 只需原
`change_id + idempotency_key`。历史 sandbox 中仍保留旧流程文字，但不包含本轮
执行 Secret；它们作为历史审计记录保留，不作为当前 Agent 行为来源。

当前容器均 `restart=0`，Firewall MCP 与专用 DinD 均 healthy。
P4R 安全整改结论：`SECURITY PASS`。

## 仓库提交状态

- PandaWiki：`feature/firewall-agent-mvp` 已推送至个人 fork
  `TyreeZhao/PandaWiki`；官方 `upstream` 的 push URL 保持 `DISABLED`。
- Firewall MCP：`feature/two-phase-approved-changes` 工作区 clean，上述代码提交均已
  本地落库。当前仓库未配置 remote，且本机 GitHub CLI 未登录，因此尚未推送。
- PandaWiki 工作区中与本特性无关的 backend 修改和测试文件未纳入本次提交。
