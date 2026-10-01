# ADR 0013: Journald and Logrotate Management

## Status
Accepted

## Context
High-volume AI proxy routing and real-time WebSocket telemetry produce substantial log lines (request routing, provider fallback events, rate-limit warnings, token usage summaries). Proper log aggregation, identifier tagging, and retention policies prevent log files from exhausting disk space.

## Decision
We implement a **dual-tier logging and rotation strategy**:
1. **Primary Stream**: OmniRoute logs directly to systemd journald with `SyslogIdentifier=omniroute`, enabling real-time inspection via `journalctl -u omniroute -f` and standard forwarding to centralized SIEM/Loki pipelines.
2. **File Rotation**: For environments that configure log files in `/var/log/omniroute/`, the role deploys `/etc/logrotate.d/omniroute`:
   - Daily rotation (`daily`)
   - 14 days retention (`rotate 14`)
   - Gzip compression with delayed compression (`compress`, `delaycompress`)
   - Non-disruptive rotation (`copytruncate`, `missingok`, `notifempty`)

## Consequences
### Positive
- Unified journald queryability across all operating system tools.
- Guaranteed disk space protection against unbounded log file growth.
- Zero restart requirement during log rotation due to `copytruncate`.

### Negative
- Logrotate configuration file must be managed alongside the role.
