# ADR 0008: Native Systemd Supervision for Ecosystem Sidecars

## Status
Accepted

## Context
OmniRoute optionally integrates with specialized companion services:
- **Redis**: Distributed rate limiting, auth cache, quota store.
- **Qdrant**: High-performance vector database for semantic memory retrieval.
- **Bifrost**: Ultra-low-latency Go-based LLM proxy and Tier-1 routing daemon.
- **Headroom**: Local prompt token compression proxy.

When OmniRoute is deployed as a native host systemd service, running sidecars as containers would introduce Docker daemon dependencies, network namespace bridging, and mixed lifecycle management.

## Decision
We provision and supervise optional sidecars as **pure native host processes with dedicated systemd units**:
1. **Redis**: When `omniroute_redis_enabled: true` and using local mode, installed via native OS package (`redis-server`), bound to `127.0.0.1:6379`, and managed via `redis.service`.
2. **Qdrant**: When `omniroute_qdrant_enabled: true`, downloads the official release binary to `/srv/omniroute/bin/qdrant` and installs `omniroute-qdrant.service`.
3. **Bifrost**: When `omniroute_bifrost_enabled: true`, downloads the official Bifrost release binary to `/srv/omniroute/bin/bifrost` and installs `omniroute-bifrost.service`.
4. **Headroom**: When `omniroute_headroom_enabled: true`, provisions the Headroom binary to `/srv/omniroute/bin/headroom` and installs `omniroute-headroom.service`.

OmniRoute connects to these local sidecars over loopback (`127.0.0.1:<port>`) or via explicitly configured external URLs.

## Consequences
### Positive
- Fully containerless, lightweight footprint with zero Docker daemon dependency.
- Uniform operational tooling: every component is inspected and controlled via `systemctl` and `journalctl`.
- Minimal memory and CPU overhead.

### Negative
- Role tasks must handle architecture-specific binary downloads (`x86_64` vs `aarch64`) for Qdrant, Bifrost, and Headroom.
