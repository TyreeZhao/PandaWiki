# Latest MVP Deployment Evidence

- Deployment time: `2026-08-14T16:14:19+08:00`
- Target: `root@10.2.138.74`
- Scope: Firewall MCP latest committed code from `720f14f`
- Agent Compose: retained `v2608.3.0-mvp-docker`
- PandaWiki: retained `v3.86.4`; no new committed PandaWiki runtime code was deployed

## Firewall MCP

- Image: `firewall-mcp:mvp-eval-v2-720f14f`
- Image digest: `sha256:16a547cf33166ce55ea11f13b648468ba5a6b98e9a7077f47c7eabb9f0ace6ca`
- Container: `firewall-mcp-mvp-firewall-mcp-1`
- Health: `healthy`
- Restart count: `0`
- In-container healthcheck: `PASS`
- SQLite pre-deploy backup:
  `/opt/firewall-mcp/backups/20260814T081333Z-pre-deploy-720f14f`

## Runtime Smoke

- Agent Compose read-only run: `succeeded`, exit code `0`
- Device result: `healthy=true`, `config_version=0`, `write_locked=false`
- Prompt explicitly prohibited `prepare_change` and `apply_approved_change`
- Agent sandbox was started with `--rm` and verified absent after the run
- Agent Compose restart count: `0`
- Eval fixture `ACTIVE` marker: absent
- Agent UI local SNI route: HTTP `200`
- Unauthenticated approval route: HTTP `401`
- Agent Compose runtime has no `/var/run/docker.sock`; it uses TLS DinD at
  `tcp://docker:2376`

## Current Test Readiness

The deployment is ready for controlled human QA/UAT of Firewall MCP read-only
queries and supervised test-network change flows. PandaWiki license was
manually revoked before this deployment, so knowledge-base Q&A cannot be
accepted as passing until the license is reactivated and the PandaWiki MCP
chain is retested.

This is not a production-readiness declaration. The v2 Eval still requires
human five-dimension scoring and the independent adversarial suite.
