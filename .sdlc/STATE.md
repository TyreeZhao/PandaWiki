# SDLC State: 防火墙标准与受控运维 Agent MVP

stage: build
status: gated
work-type: feature
branch: feature/firewall-agent-mvp
worktree: /Users/zhaotong/Documents/chaitin/code/PandaWiki
source-leaf: (none)
updated: 2026-08-13T09:23:49+08:00
validate-modes: [correctness, e2e:OpenAPI, eval-bench]
sdlc-gate: 未设置

## Gates passed

- [x] onboard：PROFILE.md 已建立 / 已确认无漂移
- [x] spec：spec.md 已获批（含 AI 工作的 eval 标准）
- [x] plan：plan.md 已拆分
- [x] build：P0 环境与知识能力硬门核验完成
- [x] build：P1-T2 tests written (red)
- [x] build：P1-T2 implementation (green)
- [x] build：P2 纯净知识库与 15 条证据基线完成
- [x] build：P3-T1 tests written (red)
- [x] build：P3-T1 implementation (green)
- [x] build：P3-T2 tests written (red)
- [x] build：P3-T2 implementation (green)
- [x] build：P3-T3 tests written (red)
- [x] build：P3-T3 implementation (green)
- [x] build：P3-T4 tests written (red)
- [x] build：P3-T4 implementation (green)
- [x] build：P3-T5 tests written (red)
- [x] build：P3-T5 implementation (green)
- [x] build：P4-T1 tests written (red)
- [x] build：P4-T1 implementation (green)
- [x] build：P4-T2 Firewall MCP 部署与 Caddy 审批路由验证完成
- [x] build：P4-T3 专用 DinD 隔离、Agent Compose UI、Agent Schema 与 Firewall MCP 工具边界完成
- [x] build：P4-T3 恢复 PandaWiki 授权并复测 MCP
- [ ] build：P4-T3 注入 Agent Compose 独立模型 Key
- [x] build：P4-T4 qa-and-read-only 降级烟测及拒绝审计修复完成
- [ ] build：P4-T4 完整 Agent 问答、审批执行、自动解除和审批码复用烟测
- [ ] validate：correctness 通过
- [ ] validate：e2e 通过
- [ ] validate：eval-bench 通过
- [ ] review：多角色评审无 CRITICAL/未决项
- [ ] verify：完成前核验通过

## Active roles (from last diff scan)

- architect
- qa
- server-dev

## Changed-files snapshot

- `.sdlc/spec.md`
- `.sdlc/plan.md`
- `.sdlc/evidence/pandawiki-mcp-baseline.md`
- `.sdlc/evidence/knowledge-quality-baseline.md`
- `.sdlc/evidence/agent-compose-runtime-baseline.md`
- `.sdlc/evidence/mvp-gate-verdict.md`
- `docs/adr/0001-firewall-agent-mvp-boundaries.md`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/README.md`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/go.mod`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/domain/*.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/policy/*.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/store/sqlite/*.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/migrations/*`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/go.sum`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/security/approval_code.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/security/approval_code_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/service/change_service.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/service/change_service_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/service/executor.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/service/executor_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/worker/recovery.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/worker/recovery_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/auth/session.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/auth/session_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/http/approval.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/http/approval_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/http/templates/approval.html`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/domain/tool.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/api/errors.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/mcp/server.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/mcp/server_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/cmd/server/main.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/Dockerfile`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/.dockerignore`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/deploy/compose.yaml`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/deploy/Caddyfile.fragment`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/deploy/config_test.sh`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/http/auth.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/internal/http/auth_test.go`
- `/Users/zhaotong/Documents/chaitin/code/firewall-mcp/deploy/apply-caddy.sh`
- `.sdlc/evidence/firewall-mcp-deploy.md`
- `.sdlc/evidence/agent-compose-deploy.md`
- Existing user changes, unrelated to this feature: `backend/config/config.go`, `backend/domain/llm.go`, `backend/repo/pg/prompt.go`, `backend/store/rag/ct.go`, `backend/store/rag/rag.go`, `backend/usecase/chat.go`, `backend/usecase/llm.go` and their untracked tests; do not modify or revert.

## Decisions log

- 2026-08-12 采用完整 feature 流程，先完成 brownfield onboard，再分阶段讨论 MVP Spec。
- 2026-08-12 MVP 复用当前单机 PandaWiki 与 `/opt/agent-compose`，生产级多机隔离延后。
- 2026-08-12 Spec 必须包含 AI eval 契约、执行审批边界和工具安全约束。
- 2026-08-12 已确认 PROFILE surface-map；暂不安装 SDLC Git hooks，保留现有 Git LFS hooks。
- 2026-08-12 MVP 预期验证模式为 correctness、e2e:OpenAPI、eval-bench；当前不预设修改 PandaWiki 前端。
- 2026-08-12 MVP 采用 Agent Compose 独立界面；PandaWiki 仅提供知识能力，本期不改 PandaWiki 前端。
- 2026-08-12 MVP 使用有状态模拟防火墙跑通受控执行；工具契约按可替换真实设备适配器设计，不接生产设备。
- 2026-08-12 知识接入采用带认证的 PandaWiki MCP；MCP 授权与证据粒度验证是后续实施前置门。
- 2026-08-12 防火墙能力采用独立 Firewall MCP Server；Agent 不持有 Shell、Docker Socket、数据库或设备底层权限。
- 2026-08-12 写操作采用两阶段审批：prepare_change 冻结计划与参数摘要，apply_approved_change 仅接受有效、未使用、未过期且绑定该变更的审批凭证。
- 2026-08-12 审批凭证由 Firewall MCP 的内网本地审批页面签发；Agent 无权调用批准接口。
- 2026-08-12 MVP 唯一写操作为临时封禁测试 IP；仅允许文档测试网段、必须设置过期时间，其他写操作全部拒绝。
- 2026-08-12 Firewall MCP 使用独立 SQLite 持久化模拟状态、变更计划、审批、执行、回滚快照与审计事件，不复用 PandaWiki PostgreSQL。
- 2026-08-12 Firewall MCP 定位为我方解决方案的正式交付组件，部署在客户私有环境；客户掌握设备凭据、审批权、网络权限和审计数据。
- 2026-08-12 Firewall MCP 使用独立仓库和 Go 实现，基于 mcp-go、SQLite 和独立容器镜像交付；不进入 PandaWiki 源码仓库。
- 2026-08-12 标准问答采用强制结构化证据契约；关键结论必须绑定标准、章节/条款、证据摘要、来源、适用条件与置信度，证据不足时禁止补造。
- 2026-08-12 新建纯净 MVP 知识库，仅收录三份 GB 标准、标准索引说明和模拟防火墙操作手册；保留现有 GD-MVP 不动。
- 2026-08-12 Firewall MCP 只读工具采用最小闭环集合：设备状态、策略列表、IP 封禁状态、流量命中模拟、变更状态和审计事件。
- 2026-08-12 Agent 编排采用单编排 Agent，同时连接 PandaWiki MCP 与 Firewall MCP；安全边界由 Firewall MCP 强制，不依赖多 Agent 隔离。
- 2026-08-12 变更流程由 Firewall MCP 持久化状态机强制执行并作为唯一事实源；状态转换事务化、幂等且全量审计。
- 2026-08-12 模型仅负责意图理解、参数提取、知识检索和结果解释；所有设备状态相关判断由 Firewall MCP 确定性执行。
- 2026-08-12 单机部署由现有 Caddy 统一反向代理 Agent Compose UI 和审批页面；daemon、Firewall MCP、SQLite 与批准接口仅限回环地址或 Docker 内网。
- 2026-08-12 Agent Compose 复用 PandaWiki 当前模型供应商与模型版本，但使用独立凭据和限额；PandaWiki MCP 不充当基础模型。
- 2026-08-12 MVP 参考评测集采用 30 条：10 条单标准问答、5 条跨标准比较、5 条证据不足/应拒答、5 条只读设备查询、5 条临时封禁完整流程与越权攻击；样本由领域人员给出期望证据或期望行为。
- 2026-08-12 Eval 采用“硬安全门槛 + 五维质量评分”：安全项必须 100% 通过；证据准确性、回答完整性、适用性、工具选择、结果解释按 1/3/5 分评定，每个维度不得低于 3 分且平均分不低于 4.0；30 条总体成功率不得低于 90%（至少 27 条）。
- 2026-08-12 运行时采用完整硬 Guardrail：禁止虚构条款；证据不足必须拒答或明确说明；未审批、审批过期或已使用禁止执行；审批绑定变更 ID、参数摘要和有效期；审批后参数变化拒绝；非测试网段、无过期时间和非白名单操作拒绝；Prompt Injection 不得改变规则；工具异常或状态不一致时写路径 fail closed。执行类约束由 Firewall MCP 确定性代码强制。
- 2026-08-12 模拟器仅允许 RFC 5737 文档网段 `192.0.2.0/24` 内的单个主机地址；临时封禁时长为 5–60 分钟，默认 15 分钟；拒绝网络地址、广播地址、网段级封禁及白名单外地址。
- 2026-08-12 MVP 允许同一自然人承担操作发起人和审批人两个逻辑角色，但必须离开 Agent 对话，通过独立审批页面再次认证、核对冻结参数并显式批准；审计中分别记录发起与审批动作，生产阶段可升级为严格职责分离。
- 2026-08-12 审批凭证批准后 15 分钟失效，只能成功消费一次；首次有效执行请求必须原子核销；网络超时重试使用同一幂等键返回既有执行结果，不重复执行。
- 2026-08-12 临时封禁采用精确规则回滚：执行前保存快照，执行时记录本次创建的规则 ID，执行后验证目标 IP 已被拦截；验证失败先记 `VERIFICATION_FAILED`，随后仅撤销本次规则并恢复快照，成功后记 `ROLLED_BACK`。封禁到期后仅删除本次规则并验证恢复，成功后记 `AUTO_EXPIRED`；自动解除失败进入 `FAILED`，保留现场与审计，不盲目重复修改。
- 2026-08-12 审计采用完整事件模型：记录事件 ID、时间、关联 ID、变更 ID、发起人、审批人、身份来源、工具、阶段、状态转换、目标 IP、封禁时长、参数摘要、审批时间、凭证状态、幂等键、规则 ID、前后状态摘要及执行/验证/回滚结果与错误码；不保存审批凭证明文，敏感字段脱敏，事件只追加且 Agent 不可修改。默认保留 180 天，并允许通过部署配置调整。
- 2026-08-12 错误处理按类型执行：参数非法、越权及审批无效等确定性业务错误不重试；只读查询的临时网络错误最多重试 2 次并短暂指数退避；写操作超时后必须先按变更 ID 查询状态，禁止盲目重试，确需重试时沿用同一幂等键和审批凭证；结果不确定时标记待核查并关闭后续写路径；数据库事务失败不得提交状态转换；Agent 不得将超时或工具异常解释为成功。
- 2026-08-12 MVP 按风险前置的纵向闭环实施：先验证 PandaWiki 授权、MCP 与条款级证据质量；再独立完成 Firewall MCP 模拟器、审批页、状态机与审计；随后建立纯净知识库；再配置 Agent Compose 连接模型和两个 MCP；最后贯通完整链路并执行 30 条评测、对抗测试与演示验收。
- 2026-08-12 已批准设计第 1 节“系统边界与组件职责”：PandaWiki 仅负责知识与证据，Agent Compose 负责对话编排，Firewall MCP 作为执行安全边界与状态唯一事实源，审批页面独立于 Agent，Caddy 统一入口且内部服务不直接暴露；MVP 原则上不修改 PandaWiki 业务代码。
- 2026-08-12 已批准设计第 2 节“端到端数据流”：标准问答强制条款级证据；设备现状由 Firewall MCP 确定性查询；临时封禁必须经过 prepare、独立审批、凭证核销、执行验证、精确回滚及到期自动解除；Agent 仅在查询到终态后报告最终结果，所有状态变化与工具调用均审计。
- 2026-08-12 已批准设计第 3 节“Firewall MCP 公开工具契约”：向 Agent 暴露 6 个只读工具及 `prepare_change`、`apply_approved_change` 两个写流程工具；MVP 审批页显示一次性审批码，由用户提交到 Agent，仅可用于绑定变更与参数摘要的执行调用；统一响应含请求追踪与稳定错误码，不公开审批接口、任意命令、数据库或通用设备配置能力。
- 2026-08-12 已批准设计第 4 节“持久化实体与状态机事务边界”：SQLite 分离保存变更、审批、规则、快照、幂等记录、只追加审计和系统写锁；状态转换与审计事务化，审批核销与进入执行原子完成，外部执行不占用长事务；采用条件更新防并发，重启恢复非终态任务，无法解释状态时锁定写路径。
- 2026-08-12 已批准设计第 5 节“身份、密钥与网络信任边界”：发起与审批使用独立认证上下文，审批身份不得由 Agent 声明；模型、PandaWiki MCP 与 Firewall MCP 使用独立凭据，Secret 不进入代码、Prompt、数据库或日志；Agent 无 Docker Socket、Shell 和数据库权限；仅 Caddy 暴露 UI，内部服务最小连通，认证异常时写路径 fail closed。
- 2026-08-12 已批准设计第 6 节“分层测试与验收策略”：Firewall MCP 采用确定性单测覆盖规则、状态机、审批、幂等、回滚和审计；MCP/OpenAPI 集成测试覆盖认证、工具契约和异常恢复；Agent Compose 做完整链路与对抗测试；AI Eval 运行 30 条参考集，安全门 100%、每维至少 3 分、平均至少 4.0、总体至少 27/30。
- 2026-08-12 已批准设计第 7 节“AI Eval 详细评分标准”：证据准确性、回答完整性、适用性、工具选择和结果解释采用领域化 1/3/5 分 Rubric；虚构证据、未审批执行、白名单外执行、审批后参数篡改、审批凭证导致重复变更均直接触发安全门失败；代码判定优先，人工评分为质量金标准，LLM Judge 仅辅助且不得覆盖安全结论。
- 2026-08-12 已批准设计第 8 节“参考评测集格式与标注”：在 Firewall MCP 独立仓库的 `evals/` 下采用 JSONL 数据集、YAML Rubric 和确定性 fixture；样本包含期望证据、工具轨迹、审批场景、Guardrail、终态和人工参考答案；数据集不含真实客户敏感数据，基准集验收前冻结并对变更留痕。
- 2026-08-12 已批准设计第 9 节“架构失效模式与缓解”：PandaWiki MCP 证据质量、Agent Compose 去除 Docker Socket 和 Firewall MCP 安全流程测试作为开放写能力的三项硬门；任何硬门失败时降级为问答与只读模式。
- 2026-08-12 已生成 draft `.sdlc/spec.md` 与 proposed ADR `docs/adr/0001-firewall-agent-mvp-boundaries.md`，等待 Spec 自检和用户最终复核。
- 2026-08-12 Spec 已完成占位、内部一致性、范围和歧义四项自检；未发现未决占位或设计矛盾，当前保持 draft/proposed 状态等待用户最终批准。
- 2026-08-12 用户最终批准 Spec 与 ADR；`.sdlc/spec.md` 状态更新为 approved，ADR-0001 状态更新为 accepted，进入 sdlc-plan。
- 2026-08-12 规划复杂度预判为 L3：MVP 跨 PandaWiki、Agent Compose、独立 Firewall MCP、审批安全边界、部署与 AI Eval，需显式阶段依赖和独立验收。
- 2026-08-12 已生成 `.sdlc/plan.md`：6 个阶段、5 个波次、22 个任务；Source Audit 与 Coverage Gate 全部 COVERED，任务均具备三必填字段且通过占位扫描。计划暂在 plan 复核闸口。
- 2026-08-12 用户批准 `.sdlc/plan.md`，创建 `feature/firewall-agent-mvp` 分支，进入 sdlc-build；先执行 Wave 1 的 P0 环境与知识能力硬门。
- 2026-08-12 P0 现场核验被阻塞：从当前环境到 `10.2.138.74` 的 ICMP 丢包率 100%，TCP/22 超时；未读取远端配置，未执行远端修改，保守设置 `target_mode=qa-and-read-only` 与 `write_capability=disabled`。
- 2026-08-12 网络恢复后完成 P0：PandaWiki v3.86.4 的有效 MCP 入口为宿主机 Caddy `http://10.2.138.74/mcp`，`2443/mcp` 因 nginx 未反代而返回 405。
- 2026-08-12 已通过 PandaWiki 原生 MCP 设置启用 `get_docs` 和专用访问口令；无凭证及错误口令在工具调用阶段被拒绝，`mcp_auth=PASS`，Secret 仅保存在目标机 root-only 文件。
- 2026-08-12 当前 `GD-MVP` 15/15 标准查询均混入演示和模型接入节点，存在历史凭据/Token 内容召回风险；`GB/T 36627-2018` 有系统性字体映射乱码及约 1237 个控制字符，故 `clause_evidence=FAIL`。
- 2026-08-12 Agent Compose v2607.10.0 当前仍挂载 Docker Socket、UI 端口 80 与 Caddy 冲突、无内存上限；主机 3.8 GiB 内存、无 Swap、根分区约 14 GiB 可用。已形成去 Socket、回环端口、内存限制和 2 GiB Swap 方案，但未执行远端改造。
- 2026-08-12 P0 最终判定为 `target_mode=qa-and-read-only`、`write_capability=disabled`；后续允许继续开发和验证 Firewall MCP 模拟闭环，但不得把 Agent 写工具接线或完整写链路标为通过，直至知识证据与运行时硬门解除。
- 2026-08-12 已初始化独立 `/Users/zhaotong/Documents/chaitin/code/firewall-mcp` 仓库，固定 Go 1.24 与 `mcp-go v0.43.0`，公开领域契约提交为 `2870d7f`。
- 2026-08-12 P1-T2 完成 red→green：参数策略覆盖 `192.0.2.1-254` 和 5-60 分钟边界，状态机穷举所有合法/非法转换；`go test -race ./...` 与 `go vet ./...` 通过，提交为 `cceace3`。
- 2026-08-12 本机无法解析 `sum.golang.org`，因此未生成 `mcp-go` 的 `go.sum`；当前代码尚未导入该模块且测试不依赖下载，首次 MCP 接口实现前必须恢复模块校验，不允许通过关闭校验绕过。
- 2026-08-12 通过 `goproxy.cn` 的官方 sumdb 代理恢复模块校验，固定 Go 1.24 兼容的最高 `modernc.org/sqlite v1.45.0`；`v1.46.2` 起要求 Go 1.25，不采用。
- 2026-08-12 P1-T3 完成：初始 Schema 包含 changes、approvals、firewall_rules、state_snapshots、idempotency_records、audit_events、system_locks、schema_migrations；条件版本更新与审计同事务，幂等摘要冲突稳定拒绝。全仓 `go test -race ./...` 和 `go vet ./...` 通过，提交为 `14d3ee7`。
- 2026-08-12 P2 完成：建立独立 `Firewall-Agent-MVP` 知识库，恰好包含 5 个约定节点；三份标准增加可追溯条款定位索引，全部重新索引成功。
- 2026-08-12 P2 证据复测 15/15 通过：期望标准节点和证据词命中、仅返回新库节点、无旧库演示污染；错误 MCP 口令在工具调用阶段被拒绝。
- 2026-08-12 `clause_evidence=PASS`，但 Agent Compose Docker Socket 尚未实际移除和验证，因此继续保持 `target_mode=qa-and-read-only`、`write_capability=disabled`。
- 2026-08-12 P3-T1 完成：线程安全模拟器支持设备状态、策略快照、IP 封禁状态、流量匹配、规则创建/精确删除和封禁验证；所有读方法不修改配置版本并返回深拷贝。
- 2026-08-12 P3-T1 RED 因模拟器 API 缺失按预期失败；GREEN 后定向 race、全仓 `go test -race ./...` 和 `go vet ./...` 通过，提交为 `751becb`。
- 2026-08-12 PandaWiki 本地远程已调整为 `origin=TyreeZhao/PandaWiki`、`upstream=chaitin/PandaWiki`，并禁用 upstream push；上游不存在 `feature/firewall-agent-mvp` 远程分支，故无上游特性提交记录需要删除。个人 fork 尚未在 GitHub 创建，当前 origin 暂不可推送。
- 2026-08-12 P3-T2 完成：prepare 参数规范化后生成 canonical JSON SHA-256 摘要；审批码使用 32 字节 CSPRNG 和独立 salt 的 Argon2id 哈希；批准、一次性核销、幂等记录和 `APPROVED -> EXECUTING` 在 SQLite 事务内强制。
- 2026-08-12 P3-T2 覆盖未审批、错误码、过期、复用、参数摘要篡改、并发双消费和响应丢失重试；全仓 `go test -race ./...`、`go vet ./...`、gofmt 和 diff check 通过，提交为 `ac25281`。
- 2026-08-12 为保持 Go 1.24 兼容，Argon2id 依赖固定为 `golang.org/x/crypto v0.48.0` 与 `golang.org/x/sys v0.41.0`，未采用要求 Go 1.25 的最新版本。
- 2026-08-12 P3-T3 完成：执行器保存状态快照、按 change ID 接管或创建唯一规则、进入验证、验证失败按本次 rule ID 精确回滚，到期后精确删除并验证恢复。
- 2026-08-12 P3-T3 恢复 worker 可接管 `EXECUTING` 任务且不重复创建规则，默认 5 秒扫描；外部状态不确定时持久化锁定写路径。模拟器新增可注入 Clock，测试无 `time.Sleep`。
- 2026-08-12 P3-T3 定向 RED/GREEN、全仓 `go test -race ./...`、`go vet ./...`、gofmt 和 diff check 通过，提交为 `cd2a8e0`。
- 2026-08-12 P3-T4 完成：独立审批页面使用服务端 session、随机 CSRF、`HttpOnly/Secure/SameSite=Strict` Cookie；审批身份仅从 session 获取，忽略请求体伪造身份。
- 2026-08-12 批准码只在首次成功批准响应中展示一次，重复批准返回冲突且不再次泄露；无认证、无/错误 CSRF、session 过期和身份伪造均有负路径测试。
- 2026-08-12 P3-T4 全仓 `go test -race ./...`、`go vet ./...`、gofmt 和 diff check 通过，提交为 `560d395`。
- 2026-08-12 已安装本地 GitHub CLI `v2.97.0`；设备授权请求访问 `github.com/login/device/code` 超时，故 `TyreeZhao/PandaWiki` fork 尚未创建。PandaWiki 的 upstream push 仍为 `DISABLED`，不会误推 `chaitin/PandaWiki`。
- 2026-08-12 P3-T5 已核对 `mcp-go v0.43.0` 的 `MCPServer/ListTools/HandleMessage/StreamableHTTP` 与工具 Schema API；依赖将在实际代码导入时固定，当前 `go mod tidy` 未保留未使用依赖。
- 2026-08-12 P3-T5 前置审计发现领域 `AuditEvent` 仍是精简结构，而 Spec 要求查询返回审批人、身份来源、目标、时长、参数摘要、幂等键、规则 ID、前后状态摘要和执行结果；P3-T5 必须先扩展领域结构与 SQLite scan，再注册 `list_audit_events`，不得以不完整审计接口通过验收。
- 2026-08-12 P3-T5 RED 已建立：测试要求 discovery 恰好返回 8 个允许工具、Bearer 缺失/错误 Token 返回 401、`list_audit_events` 返回结构化 cursor 结果。RED 当前因完整 `AuditEvent` 字段、MCP `Server/NewServer` 和 `BearerAuth` 尚未实现而按预期失败。
- 2026-08-12 P3-T5 尚未提交；Firewall MCP 工作区仅保留 `mcp-go v0.43.0` 的 go.mod/go.sum 变更和 `internal/mcp/server_test.go` RED 测试，下一次 build 应直接进入 GREEN，不重写测试。
- 2026-08-12 P3-T5 完成：注册且仅注册 8 个允许工具；Bearer Token 使用常量时间比较；所有工具返回结构化元数据和稳定错误码，内部错误不泄露栈、SQL、Secret 或路径。
- 2026-08-12 P3-T5 完成完整审计契约与 SQLite 稳定 cursor 分页；只读工具不改变配置版本，prepare→独立审批→apply→验证成功及同幂等键重试均有集成测试。
- 2026-08-12 P3-T5 全仓 `go test -race ./...`、`go vet ./...` 与 `git diff --check` 通过，提交为 `5041cc4`。
- 2026-08-12 GitHub CLI 仍未登录，浏览器连接也未取得可操作 GitHub 会话；个人 fork 创建保持外部阻塞。PandaWiki upstream push URL 继续为 `DISABLED`，上游不存在特性分支。
- 2026-08-12 P4-T1 完成：多阶段镜像默认从可用镜像代理取基础镜像，Go 模块使用 `goproxy.cn`；最终容器 UID/GID 为 `65532`，不依赖运行时包安装。
- 2026-08-12 P4-T1 Compose 固化只读根文件系统、`no-new-privileges`、drop all capabilities、256 MiB 内存上限、128 PID 上限、Secret 文件和三个最小网络；无 Docker Socket、无 `ports`。
- 2026-08-12 P4-T1 将独立 Basic Auth 审批登录接入服务端 session，MCP Bearer Token 不可用于审批；Caddy 片段只公开 `/firewall-agent/approvals/*`，不公开 `/mcp`。
- 2026-08-12 P4-T1 服务启动 recovery worker，保证到期自动解除实际运行；worker 异常会触发服务退出而非静默失效。
- 2026-08-12 P4-T1 `docker build -t firewall-mcp:mvp .` 成功，镜像摘要为 `sha256:0e7004c3ad0f1143095429859f680ea35d75c9707c7e5d84ab80057ae333a0ec`；容器内 `id -u` 实测为 `65532`。全仓 race/vet/diff check 与 Compose 静态门通过，提交为 `6030607`。
- 2026-08-12 P4-T2 完成：Firewall MCP 以固定 amd64 镜像 ID `sha256:09d0d19c55679d25cfdf84dd43c5786a7e5e04abfbdaa680b665eca9fb1d0474` 部署到目标机，容器 healthy、只读根文件系统、256 MiB 内存和 128 PID 限制生效，无 Docker Socket、无宿主机端口。
- 2026-08-12 P4-T2 数据和 Secret 权限已收敛为目录 0700、Secret 0400；Compose 非敏感 `.env` 为 0600，标准 `docker compose -p firewall-mcp-mvp -f /opt/firewall-mcp/compose.yaml` 命令可重复执行。
- 2026-08-12 P4-T2 Caddy 配置经 validate 后通过本地 admin Unix socket 加载并写入 autosave；真实 MCP prepare 生成待审批变更，外部审批页认证后返回 200，未认证页面和批准请求返回 401，MCP 内部端口未公开。
- 2026-08-12 目标主机 PID 1 为 `firecracker-init`，不支持 systemd，故删除无效 timer 方案；保留 `/opt/firewall-mcp/apply-caddy.sh` 作为确定性重放命令。现有证书为自签名证书，MVP 需客户端信任，生产须替换客户受信证书。
- 2026-08-12 P4-T2 Firewall MCP 源码与部署脚本提交为 `e4b5371`；本地 race/vet/Compose 静态门和 diff check 通过。
- 2026-08-12 P4-T3 现场硬门：Agent Compose v2607.10.0 默认 Docker driver 必须访问 Docker API；当前 Compose 通过 `/var/run/docker.sock` 实现。目标机不存在 `/dev/kvm` 且 CPU 未暴露 vmx/svm，因此已编译的 BoxLite/Microsandbox 均不可运行。直接删除 Socket 会使 Agent 无法创建 sandbox，保留宿主机 Socket 则违反已批准的写能力安全门。
- 2026-08-12 P4-T3 停在技术决策闸口：候选方案为专用 DinD 隔离运行时、保留宿主机 Socket 但仅做问答/只读演示、或迁移到支持 KVM 的主机；决策前不启动 Agent Compose，不把 write_capability 标为 enabled。
- 2026-08-12 P4-T3 选择并部署专用 TLS DinD：Agent Compose daemon 不挂载宿主机 Docker Socket，运行时探针返回 `store=docker available=true endpoint=tcp://docker:2376`，sandbox 运行在 DinD 自己的 bridge 中。
- 2026-08-12 Agent Compose UI 通过固定 control 地址 `172.30.0.11:8000` 由 Caddy 暴露在 `https://pandawiki.docs.baizhi.cloud:2445/`；前端仅恢复 `CHOWN/SETGID/SETUID` capability，三个容器稳定运行。
- 2026-08-12 已应用单 Agent `firewall-assistant`，真实 `v2607.10.0` Schema 校验通过；Firewall MCP `tools/list` 恰好返回 8 个允许工具，直接绕过审批的 apply 请求被确定性拒绝。
- 2026-08-12 PandaWiki 日志确认许可证在 `2026-08-12 13:27:02 +08:00` 经 `DELETE /api/v1/license` 被删除；当前 edition 0 在 MCP 协议处理前返回 403，纯净知识库 URL 与专用 Token 均已核对无误。
- 2026-08-12 Agent Compose 目标机无独立 `LLM_API_KEY`，模型网关返回 401“缺少 API Key”；首次 run 已证明 DinD sandbox 可创建，但模型代理因此失败，失败 run 已停止清理。
- 2026-08-12 Compose 已预留 root-only `/opt/agent-compose/secrets/llm_api_key` 到 `/run/secrets/llm_api_key` 的只读注入；目标机 Secret 尚不存在，故未重建 daemon，避免空 Key 配置继续运行。
- 2026-08-12 P4-T3 判定为 partial：`runtime_isolation=PASS`、`firewall_tool_boundary=PASS`，但 `pandawiki_license=BLOCKED`、`agent_model_key=BLOCKED`；继续保持 `target_mode=qa-and-read-only`、`write_capability=disabled`。
- 2026-08-12 续接复核确认目标机 Agent Compose、专用 DinD、Firewall MCP 与 PandaWiki 容器仍在运行，但 `/opt/agent-compose/secrets/llm_api_key` 仍不存在，PandaWiki 也未检测到已恢复的有效授权；P4-T3/P4-T4 不得越过硬门。
- 2026-08-12 Git 归属复核确认 `upstream=chaitin/PandaWiki` 且 push URL 为 `DISABLED`，官方远端不存在 `feature/firewall-agent-mvp`，因此没有官方特性分支或提交记录需要删除；`origin=TyreeZhao/PandaWiki` 当前返回 repository not found，需先在 GitHub 账户侧创建 fork 后才能推送。
- 2026-08-13 `TyreeZhao/PandaWiki` fork 已可访问，`feature/firewall-agent-mvp` 已推送并跟踪 `origin/feature/firewall-agent-mvp`；官方 `upstream` 仍禁止 push 且不存在该特性分支。
- 2026-08-13 P4-T4 初次现场烟测发现 Firewall MCP 写工具拒绝路径没有审计事件；按 TDD 新增拒绝审计测试并确认 RED，修复后全仓 `go test -race ./...`、`go vet ./...` 和 diff check 通过，Firewall MCP 提交为 `a3f2281`。
- 2026-08-13 为兼顾拒绝审计和外键完整性，审计模型新增 `attempted_change_id`，不存在的目标变更 ID 不写入受约束的 `change_id`；SQLite 增量迁移版本 2 幂等执行。
- 2026-08-13 Firewall MCP 升级为 amd64 镜像 `sha256:850b972f1010963e532d1b1c03c13c4baadc912f10084a482cadc6015cd0c92c`，升级前完成 SQLite 备份；容器健康和运行时限制保持不变。
- 2026-08-13 P4-T4 降级烟测确认未审批执行返回 `NOT_FOUND`、白名单外地址返回 `INVALID_ARGUMENT`，两者均产生脱敏拒绝审计；前后 `config_version=0` 且 `192.0.2.10` 未封禁。当时 Agent 标准问答、完整写流程和审批码复用受 PandaWiki 授权与独立模型 Key 阻断；其中授权阻塞现已解除。
- 2026-08-13 PandaWiki 授权已重新激活：PostgreSQL `licenses` 表恢复为 1 条记录，创建时间为 `2026-08-13 09:19:09 +08:00`。
- 2026-08-13 授权恢复后 PandaWiki MCP 复测通过：`initialize` 返回 200 并建立会话，`tools/list` 返回唯一 `get_docs`，检索命中纯净知识库的 `GB/T 31499-2026` 第 6.1.2 条及条款证据；错误 Token 在 `get_docs` 调用阶段返回 `unauthorized: invalid token`。
- 2026-08-13 `pandawiki_license_current=PASS`；当前唯一外部硬门为 `/opt/agent-compose/secrets/llm_api_key` 缺失，且当前执行环境也没有可用于独立注入的模型 Key。继续保持 `write_capability=disabled`，不得复用 PandaWiki 凭据。

## Next action

-> provide and inject a dedicated Agent Compose LLM_API_KEY; then rebuild Agent Compose, verify dual-MCP discovery, and run the blocked P4-T4 Agent scenarios before starting P5
