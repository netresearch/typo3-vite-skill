# Changelog

## [1.2.0] - 2026-04-30

### Fixed

- **`references/vite-configuration.md`** `server:` block now includes
  `allowedHosts: true` and `cors: true`. Without these, Vite 7.3+ and 8.x
  return HTTP 403 "Blocked request" for any host header other than
  `localhost` when accessed via reverse proxy (Traefik, nginx, etc.).
  Vite 7.1.x worked "by accident"; upgrades break HMR silently otherwise.

### Added

- Anti-pattern note: duplicate `server:` blocks in `vite.config.ts` are
  silently overwritten by JavaScript object literal semantics. Always merge
  `allowedHosts`/`cors` into the existing `server` block.
- Vite 7.1.x quirk note: explains why missing `allowedHosts` "worked"
  before 7.3.

## [1.1.0]

Previous release (no changelog kept).
