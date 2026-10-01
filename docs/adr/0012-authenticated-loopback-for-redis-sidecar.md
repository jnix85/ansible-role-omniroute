# ADR 0012: Authenticated Loopback for Redis Sidecar

## Status
Accepted

## Context
OmniRoute can use a local Redis instance for distributed rate-limiting, authentication token caching, and quota tracking. In multi-tenant Linux hosts or shared server environments, unauthenticated Redis instances listening on `127.0.0.1:6379` expose sensitive rate-limit keys and auth cache entries to any local unprivileged process or container running in the host network namespace.

## Decision
We enforce **authenticated loopback security** for local Redis sidecar instances:
1. Redis is bound strictly to `127.0.0.1` (`omniroute_redis_bind_host`).
2. An alphanumeric password (`omniroute_redis_password`) is generated idempotently on initial run if not provided and persisted to `.secrets.env`.
3. Redis configuration enforces `requirepass <password>`.
4. OmniRoute is configured with `REDIS_URL=redis://:<password>@{{ omniroute_redis_bind_host }}:{{ omniroute_redis_port }}`.

## Consequences
### Positive
- Prevents cross-process credential/cache inspection on multi-user and shared agent hosts.
- Consistent URL schema for both local and external Redis instances.
- Fully automated, zero-friction secret generation.

### Negative
- Local debugging via `redis-cli` requires supplying the `-a <password>` authentication argument.
