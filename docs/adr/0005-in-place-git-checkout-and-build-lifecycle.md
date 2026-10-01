# ADR 0005: In-place Git Checkout and Change-Detected Build Lifecycle

## Status
Accepted

## Context
OmniRoute is an actively evolving Node.js/Next.js full-stack gateway. We need an efficient, idempotent mechanism to deploy and upgrade the application on target hosts from its upstream Git repository (`https://github.com/diegosouzapw/OmniRoute.git`) at specified tags or branches (`omniroute_version`).

Building Next.js applications (`pnpm build`) requires significant CPU and memory resources. Executing full compilation on every Ansible playbook execution when no code has changed creates unnecessary overhead and extends playbook execution time.

## Decision
We implement an **in-place Git checkout with change-detected build lifecycle**:
1. The repository is cloned and checked out to `/srv/omniroute/app` using `ansible.builtin.git` targeting `omniroute_version`.
2. The role registers the git checkout result (`omniroute_git_checkout.changed`).
3. Dependency installation (`pnpm install --frozen-lockfile`) and production build compilation (`pnpm build`) are triggered **only** when:
   - The git checkout reports changes (new commit/tag checked out), or
   - The operator explicitly sets `omniroute_force_rebuild: true`.
4. Successful build completion notifies the `restart omniroute` handler to restart the systemd service.

## Consequences
### Positive
- Playbook executions are fast and fully idempotent when code is unchanged (~few seconds).
- Builds only run when necessary, conserving CPU and disk I/O on production hosts.
- `omniroute_force_rebuild: true` provides a manual escape hatch if build artifacts need regeneration.

### Negative
- A broken build during an in-place upgrade could leave `/srv/omniroute/app` in an intermediate state until the build error is resolved.
