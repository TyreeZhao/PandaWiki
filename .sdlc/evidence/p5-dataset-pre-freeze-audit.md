# P5 数据集预冻结审计

日期：2026-08-13
状态：GATED
范围：Firewall MCP `evals/dataset.jsonl`、`evals/rubric.yaml`、初始设备 fixture、
知识证据 fixture、冻结检查器和 Eval 聚合器

## 结论

30 条数据集草案的数量、分类、字段、知识引用、OCR 限定、工具契约、模拟目标、
摘要和 Secret 边界均已通过机器审计。审计中发现 3 个冻结前安全缺口，均按
TDD 修复并提交为 Firewall MCP commit `22d4dfe`。

数据集已于 2026-08-13 完成人工复核并通过冻结门。冻结版本：

1. 30 条样本的 `annotator` 均为 `tong.zhao`，`review_status` 均为 `frozen`。
2. Rubric 为 `status=frozen`、`review_status=approved`。
3. `evals/freeze-review.json` 由 `tong.zhao` 以“防火墙方案技术负责人”
   角色签署；MVP 允许同一自然人承担标注人与 reviewer 两个逻辑角色。
4. `eval-freeze-check` 返回 PASS，数据集、Rubric 和初始 fixture 摘要匹配。

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
| 冻结 fail closed | PASS | draft 状态会被拒绝；完整冻结包已通过校验 |

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

## 已确认决策

### 发起身份不匹配的归属

2026-08-13 用户确认：保持当前 30 条数据集，将发起身份不匹配作为 P5-T3 的
独立确定性对抗用例，不混入 `WR-05`。

原因：MVP 的发起身份是 MCP Bearer 认证得到的 `agent-compose` 服务级身份，
正常 Agent 会话不能切换调用身份；身份不匹配应由独立攻击客户端使用不同认证
上下文验证。该测试必须证明请求被稳定拒绝、服务端授权未消费、规则和配置版本
不变，并保留认证来源、拒绝错误码、审计事件和前后状态摘要。这样比在一个 Agent
质量样本中混合参数篡改、幂等攻击和身份攻击更可重复，也能明确失败根因。

## 人工身份状态

- 2026-08-13：用户指定 `tong.zhao` 为领域标注人。
- 2026-08-13：用户确认 `tong.zhao` 的真实领域角色为“防火墙方案技术负责人”。
- 2026-08-13：用户确认 `tong.zhao` 已完成 30 条样本的逐条复核。
- 2026-08-13：用户确认由 `tong.zhao` 同时担任 reviewer，角色仍为“防火墙方案
  技术负责人”；生产阶段再升级为严格职责分离。
- 数据集、Rubric 和 `freeze-review.json` 已更新，冻结门禁通过。
- Firewall MCP 冻结提交：`d4d9ad8`。

冻结摘要：

```text
dataset.jsonl                         fb551f491ccd18d32a2ae3dd9996eeb89e5deab5de4d4d1208559e9f19db696f
rubric.yaml                           014c55faeee2be488c659d6b43d2bfd9b89a0c29078003391e948ee55229a7ce
fixtures/initial-device-state.json   735c43c256b2149dd29d8033bbc3bd5fd9b202e289ee232662b842c1a2ba8f63
```

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
