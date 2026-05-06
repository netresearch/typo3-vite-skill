# Changelog

## [1.3.1] - 2026-05-06

### Added

- **`references/vite-configuration.md`** SVG plugin now documents the
  output-mtime skip pattern. `processedFiles` only protects against
  duplicate work within a single Vite process; between builds the map is
  empty, so every SVG would be re-optimized. The new section explains
  computing `outPath` before the read/optimize call and short-circuiting
  when `output.mtime >= source.mtime`. Includes the two ordering caveats
  (compute path before `existsSync`; keep `!changedFile` guard for dev
  watch events) and a CI caveat about wiped `Resources/Public/Svg/` dirs.

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
