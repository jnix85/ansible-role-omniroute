# ADR 0009: Playwright and Browser Automation Strategy

## Status
Accepted

## Context
OmniRoute features optional web-cookie providers (`gemini-web`, `claude-web`, `claude-turnstile`) which simulate user session authentication via headless browser automation. This functionality requires Microsoft Playwright and headless Chromium with supporting native X11/Wayland/DRM system libraries.

Because headless browsers consume significant disk (~300MB) and memory resources, browser dependencies must only be installed when explicitly enabled by the operator.

## Decision
We implement an **on-demand Playwright and Chromium provisioning task**:
1. Guarded by `omniroute_enable_playwright: true` (default `false`).
2. When enabled:
   - APT task installs required system shared libraries (`libnss3`, `libatk1.0-0`, `libcups2`, `libdrm2`, `libxcomposite1`, `libxdamage1`, `libxrandr2`, `libgbm1`, `libpango-1.0-0`, `libasound2`).
   - Executes `pnpm exec playwright install chromium` within `/srv/omniroute/app`, installing browser binaries to `/srv/omniroute/.cache/ms-playwright`.
   - The systemd unit sets `PLAYWRIGHT_BROWSERS_PATH=/srv/omniroute/.cache/ms-playwright`.

## Consequences
### Positive
- Zero browser overhead on standard deployments using standard API keys and direct provider endpoints.
- Isolated, self-contained browser runtime within the `omniroute` user directory.
- Enables web-cookie and session-based providers without external container or browser-pool dependencies.

### Negative
- Initial Playwright download and dependency installation takes several minutes when `omniroute_enable_playwright` is first toggled to `true`.
