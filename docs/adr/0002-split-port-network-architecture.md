# ADR 0002: Split-Port Network Architecture

## Status
Accepted

## Context
OmniRoute can operate in two network modes:
1. **Single-Port Mode**: The Dashboard Web UI and Proxy API endpoints both listen on port `20128`.
2. **Split-Port Mode**: The Dashboard Web UI listens on `DASHBOARD_PORT=20128`, while the AI Proxy API (`/v1`, `/v1beta`, `/a2a`) listens on `API_PORT=20129`. The real-time Live WebSocket server listens separately on `LIVE_WS_PORT=20132`.

In enterprise and multi-agent infrastructure, automated coding agents and tools (such as Claude Code, Cursor, OpenCode, Aider) need unrestricted access to the proxy API, whereas the administrative dashboard contains sensitive configuration options, credential viewing, and user management.

## Decision
We configure OmniRoute to use **Split-Port Mode** by default:
- Dashboard Web UI: `DASHBOARD_PORT=20128` (Bind: `0.0.0.0`)
- API Proxy Endpoint: `API_PORT=20129` (Bind: `0.0.0.0`)
- Live WebSocket Server: `LIVE_WS_PORT=20132` (Bind: `0.0.0.0`)

## Consequences
### Positive
- Strict network segmentation: Administrators can expose port `20129` to developer workstations or agent networks while restricting port `20128` (Dashboard) behind VPN, reverse proxy authentication (e.g. Authentik/Authelia), or firewall allowlists.
- Independent rate limiting and DDoS protection policies can be applied to data plane vs control plane endpoints.

### Negative
- Requires firewall and reverse proxy administrators to map or open multiple ports instead of a single unified port.
