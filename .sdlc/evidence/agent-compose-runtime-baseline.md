# Agent Compose 运行基线

日期：2026-08-12
状态：PASS（基线已采集；安全改造尚未执行）

## 现场

- 目录：`/opt/agent-compose`
- Agent Compose 镜像：`ghcr.io/chaitin/agent-compose:v2607.10.0`
- 前端镜像：`ghcr.io/chaitin/agent-compose-ui:latest`
- 当前 Compose 未运行，未启动 Agent Compose。
- 当前 PandaWiki Caddy 已占用宿主机 `80` 和 `2443`。

## 当前 Compose 风险

```yaml
agent-compose:
  ports:
    - 127.0.0.1:7410:7410
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./data:/data
    - ./.env:/data/work/.env:ro

agent-compose-frontend:
  ports:
    - "80:8000/tcp"
```

- `docker_socket_current=present`
- `docker_socket_target=absent`
- UI 端口 `80` 与 PandaWiki Caddy 冲突，目标端口必须改为仅回环的未占用端口，建议 `127.0.0.1:8088:8000`，由 Caddy 统一反代。
- daemon 当前目标为 `127.0.0.1:7410`；Compose 内部环境还声明 `HTTP_LISTEN=0.0.0.0:7410`、`CAP_GRPC_LISTEN=0.0.0.0:7420`，后续应限制在 Compose 内网，不对宿主机发布能力端口。
- 运行时驱动当前为 `RUNTIME_DRIVER=docker`。在移除 Docker Socket 后必须验证该版本的运行模式是否仍支持 MVP；若不支持，Agent Compose 只能保留停用或问答/只读能力，不能绕过安全硬门。

## 资源

- 主机内存：3.8 GiB
- 可用内存：约 2.7 GiB
- Swap：0
- 根分区可用：约 14 GiB
- 当前未设置 `mem_limit` 或 `deploy.resources.limits.memory`

建议方案：

- 创建 2 GiB Swap，必须在目标机变更窗口执行并验证 `swapon --show`。
- Agent Compose daemon：建议先限 `768m`。
- Agent Compose UI：建议先限 `256m`。
- Firewall MCP：建议限 `256m`。
- 变更后保留 PandaWiki 现有容器资源余量，观察 OOM、Swap 使用率和响应延迟。

## 回滚方案

1. 保留原始 `/opt/agent-compose/docker-compose.yml` 和 `.env` 的 root-only 备份。
2. 若去 Socket 或端口调整导致启动失败，停止新配置并恢复原 Compose 文件。
3. 恢复前先确认 Agent Compose 容器未获得设备、数据库或其他宿主机敏感挂载。
4. Caddy 路由变更采用独立 JSON 配置备份，健康检查失败时恢复上一份配置。
5. Swap 回滚仅在确认没有内存压力时执行 `swapoff` 并删除对应 fstab 行。

本阶段结论：`docker_socket_target=absent`；`write_capability=disabled_until_P4`。本阶段未修改 Compose、Caddy、Swap 或 Agent Compose 运行状态。
