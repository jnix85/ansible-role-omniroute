# ADR 0007: Node.js 24.x LTS Runtime and Corepack-Managed pnpm

## Status
Accepted

## Context
OmniRoute is built on modern Next.js and V8 features, leveraging fast native crypto, ES modules, and async streaming protocols. Node.js 24.x LTS provides the latest V8 engine optimizations, reduced memory overhead for large buffer transformations, and native Corepack integration.

Package manager management must be idempotent, reproducible, and avoid cluttering global NPM package directories.

## Decision
We standardize on **NodeSource Node.js 24.x LTS** managed via official repository packaging:
- NodeSource APT repository configured for `deb.nodesource.com/node_24.x`.
- Node.js 24.x and `build-essential` installed via package manager.
- Package manager `pnpm` is provisioned using official Node Corepack:
  ```bash
  corepack enable
  corepack prepare pnpm@latest --activate
  ```

## Consequences
### Positive
- Latest V8 memory management performance, especially for streaming SSE transformations and WebSocket traffic.
- Clean Corepack activation eliminates version mismatch issues with upstream repository lockfiles (`pnpm-lock.yaml`).
- Native support across Debian 12 (Bookworm) and Ubuntu 22.04 / 24.04 (Jammy / Noble).

### Negative
- Requires target operating systems to support glibc versions compatible with Node.js 24.
