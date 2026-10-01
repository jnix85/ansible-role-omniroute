# ADR 0014: Proactive Provider Recovery and Health Scheduling

## Status
Accepted

## Context
When upstream LLM providers (e.g. Anthropic, OpenAI, DeepSeek, Google Gemini) return rate limits (HTTP 429) or transient network errors, OmniRoute marks their connection credentials in a cooldown state (`rate_limited_until`) and switches traffic to healthy fallback models in the active combo.

Once the cooldown window expires, the gateway has two options:
1. **Lazy Request-Time Recovery**: Wait for a client request to hit the cooled-down provider, perform a synchronous health probe on the critical path, and retry if successful. This adds 200–800ms of probe latency to the user's request.
2. **Proactive Background Recovery**: A background scheduler periodically (default: 60s) validates credentials whose cooldown window has expired outside the request hot path.

## Decision
We enforce **Active Proactive Background Recovery** as the operational default:
- `OMNIROUTE_CONNECTION_RECOVERY_INTERVAL_MS=60000` (60-second recovery cadence).
- `OMNIROUTE_DISABLE_CONNECTION_RECOVERY=false`.
- `CREDENTIAL_HEALTH_CHECK_INTERVAL=300000` (5-minute comprehensive health polling).
- `CREDENTIAL_HEALTH_CACHE_TTL=300000`.

## Consequences
### Positive
- Client proxy requests never pay latency penalties for provider health probes.
- Combos restore primary model routing immediately after upstream provider limits reset.
- Degraded upstream accounts are flagged in the dashboard before client traffic attempts to use them.

### Negative
- Produces a small amount of periodic outbound HTTP probe traffic (~1 request per cooled-down provider every 60s).
