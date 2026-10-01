# Project Context: ansible-role-omniroute

**Type:** Ansible Role / Infrastructure Automation  
**Target:** Debian 12 (Bookworm), Debian 13 (Trixie), Ubuntu 22.04 (Jammy), Ubuntu 24.04 (Noble)  
**Service:** OmniRoute AI Gateway (Unified AI API Proxy, Combos, Token Compression, and Live Monitoring)  
**Upstream Repository:** [diegosouzapw/OmniRoute](https://github.com/diegosouzapw/OmniRoute)

---

## Architecture Summary

- **Supervision**: Production-hardened native host `systemd` service (`omniroute.service`) with sandboxing (`ProtectSystem=full`, `ProtectHome=true`, `PrivateTmp=true`, `NoNewPrivileges=true`) and cgroup limits (`MemoryHigh=2.5G`, `MemoryMax=3G`).
- **Runtime**: NodeSource Node.js 24 LTS with Corepack-managed `pnpm` in `/srv/omniroute/app`.
- **Network Layout (Split-Port Mode)**:
  - Dashboard Web UI: `DASHBOARD_PORT=20128`
  - OpenAI/Anthropic/Gemini Proxy API: `API_PORT=20129`
  - Live WebSocket Telemetry Server: `LIVE_WS_PORT=20132`
- **Secrets Management**: Idempotent generation and persistence into `/srv/omniroute/config/.secrets.env` (`chmod 0600`).
- **Database & Backups**: SQLite database at `/srv/omniroute/data/omniroute.db` tuned with WAL mode (`PRAGMA journal_mode=WAL`), paired with automated online SQLite backup timer (`omniroute-backup.timer`) and pre-upgrade snapshot hooks.
- **Toggleable Sidecars**: Native systemd-managed Redis (`redis-server`), Qdrant (`omniroute-qdrant`), Bifrost (`omniroute-bifrost`), and Headroom (`omniroute-headroom`).

---

## Directory Structure

```
ansible-role-omniroute/
├── .ansible-lint
├── .yamllint
├── .gitignore
├── ansible.cfg
├── meta/main.yml
├── inventory/hosts.yml
├── playbooks/site.yml
├── README.md
├── AGENTS.md
├── CONTEXT.md
├── PLAN.md
├── docs/adr/
│   ├── 0001-native-host-systemd-deployment.md
│   ├── 0002-split-port-network-architecture.md
│   ├── 0003-idempotent-secret-generation-and-persistence.md
│   ├── 0004-sqlite-wal-mode-and-concurrency-tuning.md
│   ├── 0005-in-place-git-checkout-and-build-lifecycle.md
│   ├── 0006-systemd-service-execution-and-environment-sourcing.md
│   ├── 0007-nodejs-24-lts-runtime-and-corepack-pnpm.md
│   ├── 0008-native-systemd-supervision-for-sidecars.md
│   ├── 0009-playwright-and-browser-automation-strategy.md
│   ├── 0010-automated-pre-upgrade-database-snapshots.md
│   ├── 0011-systemd-cgroup-resource-and-memory-limits.md
│   ├── 0012-authenticated-loopback-for-redis-sidecar.md
│   ├── 0013-journald-and-logrotate-management.md
│   └── 0014-proactive-provider-recovery-and-health-scheduling.md
└── roles/omniroute/
    ├── defaults/main.yml
    ├── vars/
    ├── handlers/main.yml
    ├── tasks/
    └── templates/
```

---

## Common Commands

```bash
# Syntax verification
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory/hosts.yml playbooks/site.yml --syntax-check

# Linting
yamllint .

# Dry-run check
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory/hosts.yml playbooks/site.yml --check --diff

# Full deployment
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory/hosts.yml playbooks/site.yml

# Target specific tags
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory/hosts.yml playbooks/site.yml --tags omniroute_config
ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory/hosts.yml playbooks/site.yml --tags omniroute_backup
```
