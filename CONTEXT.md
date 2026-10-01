# Domain Context & Ubiquitous Language: OmniRoute Gateway

This document establishes the ubiquitous language and domain concepts for the OmniRoute deployment and operations.

## Core Domain Concepts

### OmniRoute Core Gateway
The unified, open-source AI routing and API mediation service built on Node.js/Next.js. It standardizes communication across hundreds of LLM providers into standard OpenAI, Anthropic, and Google AI compatible endpoints (`/v1/chat/completions`, `/v1/models`, `/v1/responses`, `/v1beta`, `/a2a`).

### Split-Port Architecture
An operational topology separating the administrative control plane (Dashboard UI on port `20128`) from the AI proxy data plane (API endpoints on port `20129`). This separation permits fine-grained firewalling, binding policies, and reverse proxy routing between administrative users and automated client agents.

### Combo
A configured routing chain of LLM models and provider accounts that OmniRoute traverses sequentially or via load-balanced round-robin. If a provider in the chain encounters a rate limit (HTTP 429), quota exhaustion, or upstream failure (HTTP 5xx), the gateway automatically falls back to the next healthy model in the combo.

### Virtual Combo (`auto`)
A dynamic routing mode (`model: "auto"` or `auto/coding`) where OmniRoute dynamically builds and scores a virtual combo across all connected and healthy provider accounts, avoiding manual combo creation.

### Heavyweight Admission Gate
A multi-tier memory and backpressure controller that intercepts large request payloads (high byte size, extensive message counts, or high token estimates). It bounds concurrent V8 heap allocations by admitting requests into bounded queues or shedding excess load with HTTP 503 (`Retry-After`) before parsing inflates memory.

### Live WebSocket Server
A standalone real-time WebSocket daemon (default port `20132`) that broadcasts live request telemetry, token consumption rates, provider health status, and live connection logs to connected dashboard clients.

### Ecosystem Sidecars
Independent companion daemons that augment OmniRoute's core capabilities:
- **Redis Sidecar**: Backing store for distributed rate limiting, quota tracking, and multi-worker session caching.
- **Qdrant Sidecar**: Vector database enabling semantic retrieval and long-term memory offload.
- **Bifrost Sidecar**: High-throughput Go-based Tier-1 router for latency-critical upstream forwarding.
- **Headroom Sidecar**: Specialized token compression proxy sitting in front of upstream calls to reduce prompt token spend.

### SQLite Online Backup
A non-blocking database backup routine utilizing SQLite's atomic `.backup` protocol. It produces consistent point-in-time snapshots of the database file (`omniroute.db`) and WAL journals without interrupting live transactions or taking the gateway offline.

### Pre-Upgrade Database Snapshot
An atomic snapshot hook executed by Ansible immediately before checking out updated source code or applying version upgrades, ensuring guaranteed point-in-time rollback protection against schema migration regressions.

### Proactive Connection Recovery
An out-of-band background polling mechanism that periodically re-validates upstream credentials whose cooldown windows have elapsed, ensuring primary model routing restores instantly without adding latency to client proxy requests.
