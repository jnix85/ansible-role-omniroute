# ADR 0003: Idempotent Secret Generation and Persistence

## Status
Accepted

## Context
OmniRoute requires multiple sensitive cryptographic keys to operate securely:
- `JWT_SECRET`: Signs and verifies dashboard authentication session tokens.
- `API_KEY_SECRET`: Encrypts provider credentials and API keys stored in SQLite at rest.
- `STORAGE_ENCRYPTION_KEY`: Encrypts the entire SQLite database at rest (optional).
- `OMNIROUTE_WS_BRIDGE_SECRET`: Secures internal WebSocket communication for browser/desktop bridges.
- `INITIAL_PASSWORD`: Initial bootstrap administrator password.

In Ansible roles, forcing operators to generate and vault 5+ secrets before initial testing creates significant friction. Conversely, regenerating secrets on every playbook run would invalidate active user sessions, rotate DB encryption keys, and lock administrators out of their encrypted SQLite databases.

## Decision
We implement an **idempotent secret generation and persistence mechanism**:
1. If secrets are explicitly defined in Ansible variables or Ansible Vault, those values are used.
2. If secrets are undefined/empty:
   - Ansible checks whether `/srv/omniroute/config/.secrets.env` already exists on the target host.
   - If the file exists, the existing keys are preserved and loaded.
   - If the file does not exist, cryptographically strong random secrets are generated using Ansible filters (`lookup('password', ...)` or python openssl generation) and written to `/srv/omniroute/config/.secrets.env` with strict permissions (`chmod 0600`, owned by `omniroute:omniroute`).

## Consequences
### Positive
- Zero-friction initial deployment with production-grade cryptographic entropy.
- Complete idempotency across multiple playbook executions without risking database key rotation or session invalidation.
- Supports gradual migration to centralized secret management (e.g. Infisical or Ansible Vault) whenever the operator chooses to override the generated defaults.

### Negative
- Local file `/srv/omniroute/config/.secrets.env` on the host becomes a critical secret asset that must be included in system backups.
