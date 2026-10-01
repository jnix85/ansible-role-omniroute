# ADR 0011: Systemd Cgroup Resource and Memory Limits

## Status
Accepted

## Context
OmniRoute processes high-throughput AI API proxy traffic and large LLM context windows. While Node.js limits its V8 JavaScript heap via `NODE_OPTIONS=--max-old-space-size=2048` (2 GB), substantial memory is consumed outside the V8 heap in native buffers:
- SQLite WAL page cache via `better-sqlite3` native C++ bindings.
- Gzip/brotli compression stream buffers.
- Native TLS connection buffers and WebSocket telemetry sockets.
- Optional headless Chromium Playwright processes.

Without host-level cgroup boundaries, unexpected native memory expansion could trigger the Linux kernel's global Out-Of-Memory (OOM) killer, potentially terminating critical system daemons or sibling services.

## Decision
We configure **systemd cgroup resource boundaries** in `omniroute.service`:
- `MemoryHigh=2.5G` (default, configurable via `omniroute_systemd_memory_high`): Triggers kernel page-reclaim pressure before reaching hard allocation ceilings.
- `MemoryMax=3G` (default, configurable via `omniroute_systemd_memory_max`): Enforces a hard process group limit, cleanly containing OmniRoute and any child processes within a 3 GB boundary.
- `MemoryAccounting=true` and `CPUAccounting=true` for precise metrics visibility via `systemctl status omniroute`.

## Consequences
### Positive
- Strict host protection against memory leaks or runaway native buffer allocations.
- Clear separation between the 2 GB V8 heap allocation and the 1 GB native buffer headroom.
- Transparent memory and CPU metrics available natively through `systemd-cgtop` and Prometheus node_exporter.

### Negative
- Hosts running OmniRoute must have at least 3.5–4 GB of total system RAM to avoid hitting cgroup memory caps during heavy burst periods.
