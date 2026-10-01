# ADR 0001: Native Host Systemd Deployment

## Status
Accepted

## Context
OmniRoute can be deployed either containerized (via Docker / Docker Compose / Podman) or natively on the target host as a systemd-supervised Node.js service. The target environment consists of Debian 12 / Ubuntu 22.04+ (Bookworm/Jammy/Noble) hosts. 

We evaluated two main installation methods:
1. **Containerized (Docker Compose)**: Encapsulates runtime and dependencies in container images, but introduces container abstraction overhead, volume permission mappings for SQLite WAL, and complexity when integrating with local host CLI tools (e.g. Claude Code, Codex, Ollama on localhost).
2. **Native Host (Systemd Service)**: Directly runs Node.js 22 LTS with `pnpm` under a dedicated `omniroute` system user. Gives direct access to system resources, local SQLite files, seamless integration with host CLI tools and local LLMs (Ollama/vLLM), and fine-grained systemd sandboxing (`ProtectSystem=full`, `ProtectHome=true`, `PrivateTmp=true`, `NoNewPrivileges=true`).

## Decision
We deploy OmniRoute directly on the native host managed by a hardened `systemd` service unit. Node.js 22 LTS is installed via the official NodeSource repository, with dependencies managed via `pnpm`.

## Consequences
### Positive
- Direct access to local host tools, UNIX sockets, and local inference engines (Ollama, LM Studio) without container networking friction.
- Robust native service lifecycle management via `systemctl` with automatic restart and systemd journal integration.
- Native filesystem performance for SQLite in WAL mode without container mount overhead.
- Strong security isolation using systemd sandboxing directives.

### Negative
- Requires target host to install NodeSource Node.js 22 LTS and build tools (`build-essential`, `python3`) on initial deployment.
- Node.js runtime updates on the host must be managed by package manager tasks.
