# ADR 0004: SQLite WAL Mode and Concurrency Tuning

## Status
Accepted

## Context
OmniRoute utilizes SQLite via the native `better-sqlite3` Node addon located at `/srv/omniroute/data/omniroute.db`. Under high-concurrency multi-agent environments, concurrent writes from API routing requests, live WebSocket metrics streaming, periodic quota sync tasks, and online backup jobs compete for database access.

Under standard SQLite rollback journal modes (`DELETE`, `TRUNCATE`), writers lock the entire database file, causing concurrent readers or writers to fail with `SQLITE_BUSY` errors during traffic bursts.

## Decision
The Ansible role executes an idempotent pre-flight database initialization during role deployment to enforce:
- `PRAGMA journal_mode=WAL;` (Write-Ahead Logging for concurrent non-blocking reads during active writes)
- `PRAGMA busy_timeout=5000;` (5-second lock acquisition timeout to prevent immediate failure on lock contention)
- `PRAGMA synchronous=NORMAL;` (Optimal durability-performance balance for WAL mode)

## Consequences
### Positive
- Prevents database locking (`SQLITE_BUSY`) errors during concurrent coding agent bursts and telemetry updates.
- Enables safe, zero-downtime online backups using the SQLite `.backup` API while active transactions continue.
- Maximizes I/O throughput on NVMe/SSD storage.

### Negative
- Produces shared-memory (`omniroute.db-shm`) and write-ahead log (`omniroute.db-wal`) companion files alongside the main database file in `/srv/omniroute/data/`.
