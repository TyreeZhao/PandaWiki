# ADR-0002: 审批授权仅在 Firewall MCP 服务端消费

Status: accepted
Date: 2026-08-13
Supersedes: ADR-0001 中关于一次性审批码进入 Agent 对话的部分

## 背景 / 问题

ADR-0001 选择了两阶段审批，并允许审批页面生成一次性审批码，由用户将审批码
提交到 Agent，再由 Agent 调用 `apply_approved_change`。

现场全链路烟测证明该流程在功能上可用，但审批码通过 Agent prompt 传递后，会进入
Agent Compose sandbox 的 prompt、cell 和 event 持久化产物。Agent Compose 的普通
`secret` run env 也不是安全替代方案：自定义 env 会进入 sandbox 元数据，模型读取后
仍可能把值写入 transcript。

这与“Secret 不进入 Prompt、SQLite、日志或持久化对话产物”的既有安全约束冲突。
审批码虽然短期有效且只能消费一次，但它在有效期内属于可执行写操作的 bearer
credential，不能接受明文进入 Agent 边界。

## 决策

保留 `prepare -> 独立审批 -> apply` 三步交互，但把审批授权改为 Firewall MCP
服务端状态，不再生成、展示或接收审批码。

1. `prepare_change` 创建并冻结变更，记录参数摘要、发起身份和审批地址。
2. 审批人在独立页面重新认证、核对冻结参数并显式批准。
3. Firewall MCP 在同一事务中创建一次性服务端执行授权，并将变更转为
   `APPROVED`。授权绑定：
   - `change_id`
   - `parameter_digest`
   - 原始发起身份
   - 审批身份
   - `approved_at`
   - `expires_at`
4. 审批页面只返回批准结果和变更状态，不返回任何执行凭证。
5. Agent 调用：

   ```text
   apply_approved_change(change_id, idempotency_key)
   ```

6. Firewall MCP 在单个事务中校验：
   - 变更状态为 `APPROVED`
   - 服务端授权存在、未过期且未消费
   - 当前调用者与原始发起身份匹配
   - 当前参数摘要与审批绑定摘要匹配
   - 幂等键未与其他请求摘要冲突
7. 校验通过后，服务端原子消费授权并进入 `EXECUTING`。
8. 相同幂等键重试返回首次执行结果，不重复消费和执行；授权已被其他请求消费时
   返回 `APPROVAL_CONSUMED`；授权过期时返回 `APPROVAL_EXPIRED`。

`change_id` 是可审计的资源标识符，不是 Secret，也不单独形成执行权限。即使知道
`change_id`，没有服务端 `APPROVED` 授权、匹配发起身份和有效状态也不能执行。

## 备选与否决理由

### 继续使用审批码，通过 Agent Compose secret env 传递

否决。现场源码和持久化检查表明，普通自定义 env 会进入 sandbox 元数据；模型读取
env 后也可能把值写入 transcript。`secret: true` 的输出脱敏不能证明值不持久化。

### 审批页面批准后立即执行

暂不采用。它能彻底绕开 Agent 的执行调用，但会把“批准”和“执行”合并为一个动作，
改变现有交互和审计语义，也让审批页面承担触发执行的职责。服务端授权方案已能消除
凭证明文，同时保留当前三步流程。

### 将审批码放入 URL、Cookie、HTTP Header 或临时文件

否决。Agent 必须读取这些值才能构造 MCP 调用，仍可能进入模型上下文、运行时元数据
或诊断产物；同时会增加清理和失效路径。

### 接入企业审批系统服务端回调

保留为生产演进方向。它适合严格职责分离和企业 IAM，但超出当前单机 MVP 范围。
回调最终也应写入同一种服务端执行授权，而不是把 bearer credential 返回给 Agent。

## 退化 / 保留

### 退化

- 不再通过“持有审批码”证明执行请求得到批准。
- 审批后 15 分钟内，原发起 Agent 可根据服务端状态发起一次执行；授权安全依赖
  Firewall MCP 的身份绑定、事务状态机和 MCP 调用认证。
- MVP 的 MCP Bearer 仍是服务级身份，不提供最终生产所需的逐用户强身份。

### 保留

- 保留参数冻结、独立审批和明确的后续执行动作。
- 保留 15 分钟有效期、一次性消费、并发保护和幂等重试。
- 保留发起人、审批人、执行者三种逻辑身份及完整审计。
- 保留 Agent 无批准权限、无设备底层权限、无数据库权限的边界。
- 保留未来接入企业 IAM、严格双人审批和审批回调的路径。

## 架构影响 / 后果

正面后果：

- Agent prompt、transcript、sandbox、日志和 Agent Compose 数据库不再需要接触
  审批 Secret。
- 公开 MCP Schema 不再包含 `approval_code`。
- 审批授权的过期、消费、身份绑定和参数绑定集中在 Firewall MCP 的事务边界内。
- 现有状态机、执行器、验证、回滚和自动解除流程可以复用。

负面后果：

- 需要迁移既有 `approvals` 数据模型，删除或停止使用 `code_hash`。
- 需要重写审批页面响应、MCP Schema、服务层和 SQLite 原子消费测试。
- 已冻结的 Eval 和烟测用例需要从“审批码复用”改为“服务端授权重复消费”。

可逆性：

- 若生产环境要求企业审批回调，可让回调创建相同的服务端执行授权。
- 若未来引入逐用户 OAuth/mTLS，可增强发起身份绑定而不改变上层状态机。
- 不恢复任何让审批 Secret 进入 Agent 的传递方式。
