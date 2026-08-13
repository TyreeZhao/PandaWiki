# Firewall MCP P4-T2 Deployment Evidence

Date: 2026-08-12
Target: `root@10.2.138.74`
Result: PASS WITH CONCERN

## Deployed Artifact

- Source repository: `/Users/zhaotong/Documents/chaitin/code/firewall-mcp`
- Source commit: `e4b5371`
- Target compose file: `/opt/firewall-mcp/compose.yaml`
- Pinned image ID: `sha256:09d0d19c55679d25cfdf84dd43c5786a7e5e04abfbdaa680b665eca9fb1d0474`
- Runtime user: `65532:65532`
- Container health: `healthy`

## Runtime Hardening

- Root filesystem is read-only.
- All Linux capabilities are dropped.
- `no-new-privileges` is enabled.
- Memory limit is 256 MiB and PID limit is 128.
- No Docker Socket is mounted.
- No host port is published by Firewall MCP.
- MCP is reachable only through `agent-compose_default`.
- The approval upstream uses the fixed address `169.254.15.20:8080` on `pandawiki_panda-wiki`.
- `/data/firewall-mcp`, `secrets`, and `backup` are mode `0700`.
- Secret files are mode `0400`; values were not printed or copied into this evidence.
- `/opt/firewall-mcp/.env` is mode `0600` and contains only non-secret deployment parameters.

## Caddy And Approval Page

- Caddy configuration was validated before applying it through the local admin Unix socket.
- The live and autosaved Caddy configuration contains only
  `/firewall-agent/approvals/*` for this component.
- Requests to other paths on port `2444` return `404`; `/mcp` is not exposed there.
- A real `prepare_change` MCP call created change
  `c81bb9bc-7e60-452b-8c50-fa0da327a402`.
- Unauthenticated approval page request returned `401`.
- Authenticated approval page request returned `200` and displayed the frozen
  change ID and target `192.0.2.210`.
- Unauthenticated approval POST returned `401`.
- The rendered page did not contain the approval password.

## Verification Commands

- `docker compose -p firewall-mcp-mvp -f /opt/firewall-mcp/compose.yaml ps`
- `docker inspect firewall-mcp-mvp-firewall-mcp-1`
- `ss -lntp`
- `/opt/firewall-mcp/apply-caddy.sh`
- Authenticated and unauthenticated HTTPS requests through Caddy
- Local source verification:
  `go test -race ./...`, `go vet ./...`, `./deploy/config_test.sh`,
  and `git diff --check`

## Concern

The existing PandaWiki certificate for `pandawiki.docs.baizhi.cloud` is
self-signed. Hostname and SAN match, but clients that do not trust the
customer-local CA fail normal certificate verification. MVP browser testing
therefore requires importing the certificate into the client trust store or
explicitly accepting it. Production deployment must replace it with a
customer-trusted certificate.

The target host uses `firecracker-init`, not systemd. No unsupported systemd
timer was installed. The applied route is present in Caddy's persisted
`autosave.json`; `/opt/firewall-mcp/apply-caddy.sh` remains the deterministic
manual reconciliation command.

## Rejected-Write Audit Upgrade

Date: 2026-08-13
Result: PASS

- Source commit: `a3f2281`
- Deployed image: `firewall-mcp:mvp-a3f2281`
- Image ID: `sha256:850b972f1010963e532d1b1c03c13c4baadc912f10084a482cadc6015cd0c92c`
- Platform: `linux/amd64`
- Pre-upgrade database backup:
  `/data/firewall-mcp/backup/firewall.db.pre-a3f2281.20260812T235942Z`
- SQLite migration versions after restart: `1`, `2`
- New audit field: `attempted_change_id`

The upgrade fixes a smoke-test finding where rejected `prepare_change` and
`apply_approved_change` calls returned stable errors but were not audited.
Rejected calls now produce append-only, redacted events without storing an
approval code. An attempted ID that does not reference an existing change is
stored separately from the foreign-key constrained `change_id`.

Remote verification:

- nonexistent change apply -> `NOT_FOUND`
- outside-allowlist prepare -> `INVALID_ARGUMENT`
- both calls produced `result=rejected` audit events
- device `config_version` remained `0`
- `192.0.2.10` remained unblocked
- container remained healthy, read-only, limited to 256 MiB and 128 PIDs
