# ADR 0006: Systemd Service Execution and Environment Sourcing

## Status
Accepted

## Context
OmniRoute requires environment variables for runtime configuration (`PORT`, `API_PORT`, `DATA_DIR`, rate limits, CORS origins) and sensitive secrets (`JWT_SECRET`, `API_KEY_SECRET`, `STORAGE_ENCRYPTION_KEY`). 

We evaluated two execution paradigms for the systemd unit:
1. **Direct Systemd Execution with EnvironmentFiles**: Native systemd directives (`EnvironmentFile=-/srv/omniroute/config/.env` and `EnvironmentFile=-/srv/omniroute/config/.secrets.env`) loading environment files directly before invoking `pnpm start` or `pnpm run serve`.
2. **Intermediate Shell Wrapper Script**: An entrypoint bash script (`omniroute-service.sh`) that sources `.env` files and handles pre-flight validation before `exec node`.

## Decision
We implement **Direct Systemd Execution with Dual EnvironmentFiles**:
- The unit defines `WorkingDirectory=/srv/omniroute/app`.
- Configuration is loaded cleanly using two separate files:
  - `/srv/omniroute/config/.env` (Non-sensitive general configuration, chmod 0644)
  - `/srv/omniroute/config/.secrets.env` (Cryptographic keys and passwords, chmod 0600)
- Execution is handled directly via `ExecStart=/usr/bin/pnpm run serve` (or `pnpm start`).
- Sandboxing directives (`ProtectSystem=full`, `ProtectHome=true`, `PrivateTmp=true`, `NoNewPrivileges=true`) are enforced without shell interpreter leakage.

## Consequences
### Positive
- Clear separation between non-sensitive configuration parameters and cryptographic secrets on the filesystem.
- No intermediary bash processes in the process tree, enabling clean SIGTERM propagation and process management by systemd.
- Standard systemd observability (`systemctl status omniroute`, `journalctl -u omniroute`).

### Negative
- Any dynamic environment variable expansion must be handled by systemd or Node.js runtime rather than bash shell expansion.
