# Changelog

## [Unreleased]

## [1.8.1] - 2026-08-27

### Added

- First eval suite

## [1.8.0] - 2026-08-08

### Added

- Add Agent Plugins 1.0.0 portable manifest (manifest)

## [1.7.0] - 2026-07-30

### Added

- **`SKILL.md`** new "Stale Page Cache After a Build Trap" section: `vite build`
  mints new content hashes and deletes the previous files, while the rendered
  markup stays in TYPO3's page cache with the old names — the page then 404s on
  its entrypoints and loads with no JavaScript and no CSS while still rendering,
  so it presents as a broken feature rather than a broken asset reference.
  `vendor/bin/typo3 cache:flush` belongs to the build step.
- **`references/vite-configuration.md`** new "Flush the page cache after every
  build" subsection under Manifest Mode: the flush command, a `curl`/`ls`
  comparison to verify what is actually served, the note that hashes only change
  for entrypoints whose output changed (a SCSS comment leaves the CSS hash
  untouched, a TypeScript comment does change the JS hash), and the warning that
  a measurement asserting the *absence* of an effect cannot distinguish an
  unloaded bundle from a correctly idle one without positive proof that the code
  ran.

## [1.6.0] - 2026-07-13

### Added

- **`SKILL.md`** new "Dev Server `allowedHosts` Trap" section: without
  `allowedHosts: true` and `cors: true` in the `server:` block, the Vite
  dev server returns HTTP 403 "Blocked request" for any host header other
  than `localhost`, breaking HMR behind a reverse proxy (Traefik, nginx).
  Also warns that a second `server: { ... }` literal silently overwrites
  the first one's keys.

### Changed

- **`references/vite-configuration.md`** corrected the `allowedHosts`
  version boundary claim: host-header enforcement is not a Vite 7.3
  cutoff — it has been the default since the fix for
  [GHSA-vg6x-rcgg-rjx6](https://github.com/vitejs/vite/security/advisories/GHSA-vg6x-rcgg-rjx6)
  (CVE-2025-24010, landed in 4.5.6/5.4.12/6.0.9), so every Vite 7.x/8.x
  release enforces it.

### Removed

- **`references/bootstrap-theming.md`** cut generic Bootstrap/SCSS
  tutorial content (~200 lines); the reference now covers the SCSS
  theming flow, CI-color mapping, and selective Bootstrap imports.

## [1.5.0] - 2026-06-11

### Added

- **`references/vite-configuration.md`** new "Manifest Mode (No Dev Server)"
  section for build-time-only asset workflows. Documents forcing
  `useDevServer = '0'` (the `auto` default resolves to
  `Environment::getContext()->isDevelopment()`, so any `Development/*`
  context without a running HMR server breaks `<vite:asset>`),
  pre-populating the complete `vite_asset_collector` key set in
  `settings.php` (a missing key triggers a synchronize that writes the
  file — on sealed/read-only `settings.php` this throws core exception
  `#1346323822` and the frontend returns HTTP 500), and why
  `vite-plugin-typo3` project-mode output aligns with the extension's
  `defaultManifest` so `<vite:asset>` needs no `manifest` argument.
  Includes a deployment note: keep the toolchain at the Composer project
  root so `node_modules` stays out of copied extension paths.

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
