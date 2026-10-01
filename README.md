# Ansible Role: OmniRoute AI Gateway

[![Ansible Lint](https://img.shields.io/badge/ansible--lint-production-green.svg)](https://ansible.readthedocs.io/projects/lint/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A comprehensive, production-hardened, fully-featured Ansible role for deploying and managing **[OmniRoute](https://github.com/diegosouzapw/OmniRoute)**, the open-source AI gateway providing unified OpenAI-compatible routing across 350+ LLM providers with quota-aware auto-fallback, RTK/Caveman token compression, live WebSocket monitoring, and MCP integration.

---

## Features

- **Hardened Native Host Systemd Execution**: Runs under a dedicated `omniroute` system user with production sandboxing (`ProtectSystem=full`, `ProtectHome=true`, `PrivateTmp=true`, `NoNewPrivileges=true`) and cgroup resource bounding (`MemoryHigh=2.5G`, `MemoryMax=3G`).
- **NodeSource Node.js 24 LTS & Corepack**: Manages official Node.js repository and activates `pnpm` via Corepack.
- **Split-Port Network Architecture**: Segregates control plane (Dashboard UI on port `20128`) from data plane (API proxy on port `20129`) and Live WebSocket telemetry on port `20132`.
- **Zero-Friction Crypto Secrets**: Idempotently generates cryptographically secure keys (`JWT_SECRET`, `API_KEY_SECRET`, `OMNIROUTE_WS_BRIDGE_SECRET`, `INITIAL_PASSWORD`, `STORAGE_ENCRYPTION_KEY`, `OMNIROUTE_REDIS_PASSWORD`) on first run into `/srv/omniroute/config/.secrets.env` (chmod 0600) with vault override support.
- **SQLite Concurrency & WAL Tuning**: Enforces `PRAGMA journal_mode=WAL;` and busy-timeout pragmas for high concurrency.
- **Automated Online Backups & Pre-Upgrade Snapshots**: Includes scheduled `omniroute-backup.timer` utilizing SQLite online `.backup` with automatic retention pruning, plus pre-upgrade database snapshots.
- **Toggleable Sidecars**: Native systemd-managed sidecars for Redis (authenticated rate limiter), Qdrant (vector memory), Bifrost (Go router), Headroom (token saver), and Playwright (web-cookie providers).
- **100% Configuration Coverage**: Complete support for all 100+ OmniRoute environment variables and payload manipulation rules.

---

## Requirements

- **Ansible**: `2.14` or later
- **Target OS**:
  - Debian 12 (Bookworm) / Debian 13 (Trixie)
  - Ubuntu 22.04 LTS (Jammy) / Ubuntu 24.04 LTS (Noble)
- **Privileges**: Root access (`become: true`)

---

## Role Variables

Every configuration setting is documented in [`roles/omniroute/defaults/main.yml`](roles/omniroute/defaults/main.yml). Key variables include:

```yaml
# ── Version & Core Settings ──
omniroute_version: "main"                 # Git release tag or branch
omniroute_home: "/srv/omniroute"
omniroute_port_mode: "split"              # "split" or "single"
omniroute_dashboard_port: 20128
omniroute_api_port: 20129
omniroute_live_ws_port: 20132

# ── Secrets (Auto-generated if empty) ──
omniroute_jwt_secret: ""
omniroute_api_key_secret: ""
omniroute_initial_password: ""

# ── Optional Sidecars (Toggleable) ──
omniroute_redis_enabled: false            # Distributed rate-limiting
omniroute_qdrant_enabled: false           # Semantic vector memory (:6333)
omniroute_bifrost_enabled: false          # Tier-1 Go router (:8080)
omniroute_headroom_enabled: false         # Token compression proxy (:8787)
omniroute_enable_playwright: false        # Headless Chromium for web-cookie providers

# ── Automated SQLite Backups ──
omniroute_backup_enabled: true
omniroute_backup_schedule: "*-*-* 02:00:00"
omniroute_backup_retention_days: 7
```

---

## Quickstart Playbook

```yaml
---
- name: Deploy OmniRoute AI Gateway
  hosts: ai_gateways
  become: true
  roles:
    - role: omniroute
      vars:
        omniroute_port_mode: "split"
        omniroute_api_port: 20129
        omniroute_dashboard_port: 20128
        omniroute_backup_retention_days: 14
```

---

## Architecture Decision Records (ADRs)

Key architectural decisions are documented under [`docs/adr/`](docs/adr/):
- [ADR 0001: Native Host Systemd Deployment](docs/adr/0001-native-host-systemd-deployment.md)
- [ADR 0002: Split-Port Network Architecture](docs/adr/0002-split-port-network-architecture.md)
- [ADR 0003: Idempotent Secret Generation and Persistence](docs/adr/0003-idempotent-secret-generation-and-persistence.md)
- [ADR 0004: SQLite WAL Mode and Concurrency Tuning](docs/adr/0004-sqlite-wal-mode-and-concurrency-tuning.md)
- [ADR 0005: In-place Git Checkout and Build Lifecycle](docs/adr/0005-in-place-git-checkout-and-build-lifecycle.md)
- [ADR 0006: Systemd Service Execution and Environment Sourcing](docs/adr/0006-systemd-service-execution-and-environment-sourcing.md)
- [ADR 0007: Node.js 24.x LTS Runtime and Corepack pnpm](docs/adr/0007-nodejs-24-lts-runtime-and-corepack-pnpm.md)
- [ADR 0008: Native Systemd Supervision for Ecosystem Sidecars](docs/adr/0008-native-systemd-supervision-for-sidecars.md)
- [ADR 0009: Playwright and Browser Automation Strategy](docs/adr/0009-playwright-and-browser-automation-strategy.md)
- [ADR 0010: Automated Pre-Upgrade Database Snapshots](docs/adr/0010-automated-pre-upgrade-database-snapshots.md)
- [ADR 0011: Systemd Cgroup Resource and Memory Limits](docs/adr/0011-systemd-cgroup-resource-and-memory-limits.md)
- [ADR 0012: Authenticated Loopback for Redis Sidecar](docs/adr/0012-authenticated-loopback-for-redis-sidecar.md)
- [ADR 0013: Journald and Logrotate Management](docs/adr/0013-journald-and-logrotate-management.md)
- [ADR 0014: Proactive Provider Recovery and Health Scheduling](docs/adr/0014-proactive-provider-recovery-and-health-scheduling.md)

---

## License

MIT © [Jason Parks](https://github.com/diegosouzapw/OmniRoute)
