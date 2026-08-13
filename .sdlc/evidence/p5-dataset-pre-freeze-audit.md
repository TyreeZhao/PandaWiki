# P5 数据集预冻结审计

日期：2026-08-13
状态：GATED
范围：Firewall MCP `evals/dataset.jsonl`、`evals/rubric.yaml`、初始设备 fixture、
知识证据 fixture、冻结检查器和 Eval 聚合器

## 结论

30 条数据集草案的数量、分类、字段、知识引用、OCR 限定、工具契约、模拟目标、
摘要和 Secret 边界均已通过机器审计。审计中发现 3 个冻结前安全缺口，均按
TDD 修复并提交为 Firewall MCP commit `22d4dfe`。

数据集仍不能冻结，原因是：

1. 30 条样本的 `annotator` 均为空，`review_status` 均为 `draft`。
2. 尚无真实复核人签署的 `freeze-review.json`。
3. 已批准计划要求写样本覆盖“发起身份不匹配”，当前 30 条中的 5 个审批场景
   未直接包含该场景；需在冻结前确认是调整数据集，还是明确作为 P5-T3 独立
   对抗用例执行。

## 机器审计

| 检查项 | 结果 | 证据 |
|---|---|---|
| 样本总数 | PASS | 30 |
| ID 唯一性 | PASS | 30/30 唯一 |
| 分类配比 | PASS | `10/5/5/5/5` |
| Spec 7.2 必填字段 | PASS | `LoadDataset` |
| 初始状态 fixture | PASS | 30 条统一指向 `fixtures/initial-device-state.json` |
| 知识证据引用 | PASS | 14 个被引用 ID 全部存在于 15 条 fixture |
| OCR 限定 | PASS | 6 个引用 GB/T 36627 OCR 证据的样本均要求限定结论 |
| 期望工具白名单 | PASS | 仅 PandaWiki `get_docs` 和 Firewall MCP 公开工具 |
| expected/forbidden 互斥 | PASS | 无自相矛盾工具 |
| 受控写目标 | PASS | 均为 `192.0.2.1-254` 内单主机 |
| 封禁时长 | PASS | 15 分钟，处于 5-60 分钟范围 |
| Rubric 评分值 | PASS | 聚合器仅接受 `1/3/5` |
| Secret 契约 | PASS | 四项 `*_allowed` 均为 false，无凭据值形态命中 |
| README 摘要 | PASS | dataset、rubric、fixture 三个 SHA-256 一致 |
| 冻结 fail closed | PASS | 当前在 `SQA-01` 空 annotator 处拒绝 |

当前摘要：

```text
dataset.jsonl
c2e01825cda73308520539ff0ce91fff2613e8294a0ce7e9db81b73d2dabbdfd

rubric.yaml
59abbba953ba04c62430c60a5e0968a9d08841cbaeb0969b2b69d71318f34342

fixtures/initial-device-state.json
735c43c256b2149dd29d8033bbc3bd5fd9b202e289ee232662b842c1a2ba8f63
```

## 修复项

### F-01 受控目标未复用执行策略

严重度：HIGH

RED：

```text
target_ip "10.0.0.1" accepted; want RFC 5737 host-range rejection
```

根因：Eval 数据集只检查 `target_ip` 非空，未使用 Firewall MCP 的执行策略。

修复：解析 IPv4 后复用 `policy.ValidateTemporaryBlock`，拒绝生产网段、网络地址、
广播地址、IPv6 和非法地址。数据集与实际执行共享同一允许范围事实源。

### F-02 数据集可声明任意期望工具

严重度：HIGH

RED：

```text
LoadDataset() error = <nil>, want unsupported expected tool failure
```

根因：`expected_tools` 只检查字段存在，不校验公开工具契约，也不检查
expected/forbidden 集合冲突。

修复：期望工具限制为 PandaWiki `get_docs` 加 Firewall MCP 8 个公开工具；同一
工具不能同时标为 expected 和 forbidden。`forbidden_tools` 仍允许记录未公开的
危险工具名用于对抗检测。

### F-03 聚合器接受 Rubric 外分值

严重度：HIGH

RED：

```text
result passed with scores outside the 1/3/5 rubric scale
```

根因：原实现只检查分值不低于 3 和平均分不低于 4，五项全为 4 会通过。

修复：单样本判定和最终聚合均强制分值属于 `1/3/5`；其他整数直接形成维度失败。

## 覆盖观察

- 知识 fixture 共 15 条，当前数据集引用 14 条；`K10`（GB/T 36627-2018
  第 5.1.6 条密码检查）未被使用。这不是 Spec 的硬性失败，但领域复核人应确认
  当前 30 条覆盖取舍是否合理。
- 5 个受控写样本分别覆盖：服务端批准、未审批、授权过期、同幂等键重放、
  参数篡改或不同幂等键重放。
- P5-T3 另有 Prompt Injection、发起身份不匹配、并发消费、响应丢失重试、
  服务重启和自动解除失败注入等对抗任务，不能因 30 条数据集通过而省略。

## 待确认决策

### 发起身份不匹配的归属

1. 调整 30 条数据集：扩展 WR-05 和采集协议，在一个样本中同时执行参数篡改、
   不同幂等键和发起身份不匹配攻击。
2. 保持当前 30 条数据集，将发起身份不匹配明确为 P5-T3 的独立确定性对抗用例，
   并修订 P5-T1 的 action 描述。

推荐选项 2。MVP 的发起身份是 MCP Bearer 认证得到的 `agent-compose` 服务级
身份，正常 Agent 会话不能切换调用身份；身份不匹配应由独立攻击客户端使用
不同认证上下文验证，和 Agent 对话质量样本分开更可重复，也不会把多个失败原因
混入 WR-05。

## 人工冻结清单

- 标注人逐条核对 prompt、human reference、标准编号、年份、条款和摘要。
- 对 6 个 GB/T 36627 OCR 样本确认必须提示人工复核。
- 核对 5 个比较样本没有把不同标准合并成同一要求。
- 核对 5 个拒答样本的拒答边界和允许的限定回答。
- 核对 5 个只读样本的工具、参数和初始状态预期。
- 核对 5 个写样本的审批场景、终态、无副作用要求和审计要求。
- 填写真实 `annotator`，将复核通过的样本改为 `frozen`。
- 将 Rubric 改为 `status=frozen`、`review_status=approved`。
- 由真实复核人创建并签署 `freeze-review.json`。
- 重新计算摘要并运行 `eval-freeze-check`；不得覆盖旧冻结版本。
