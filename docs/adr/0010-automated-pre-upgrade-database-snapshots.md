# ADR 0010: Automated Pre-Upgrade Database Snapshots

## Status
Accepted

## Context
When updating OmniRoute to a newer release version (`omniroute_version`), upstream database schema migrations execute automatically during application bootstrap. If a migration encounters an incompatibility, SQLite schema lock, or corruption, rolling back the application code is insufficient if the database state was partially modified.

Relying solely on periodic daily backup timers risks losing data generated since the last scheduled snapshot.

## Decision
We implement an **automated pre-upgrade database snapshot hook** in the Ansible role:
1. Before checking out updated source code or running `pnpm build`, if an existing `omniroute.db` database is present and git changes are detected:
   - Ansible triggers an immediate non-blocking online SQLite `.backup` to `/srv/omniroute/backups/pre-upgrade_{{ omniroute_current_version }}_{{ timestamp }}.db.gz`.
2. This creates a guaranteed point-in-time recovery point immediately prior to code checkout and schema migration execution.

## Consequences
### Positive
- Zero data loss risk during version upgrades.
- Immediate rollback capability if new version schema migrations fail.
- Non-blocking online execution ensures active proxy traffic is not interrupted during the snapshot.

### Negative
- Adds a few seconds of execution time to upgrade runs when version changes occur.
