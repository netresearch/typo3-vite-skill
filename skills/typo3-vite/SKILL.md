---
name: typo3-vite
description: "Vite build setup for TYPO3 v13+ (v14.3 LTS current target; v14.0 removed core asset concat/compression #108055 — external build tool is now mandatory) with vite-asset-collector, SCSS architecture, Bootstrap 5.3 theming (v14 backend bundles Bootstrap 5.3.2), SVG optimization, code splitting, CSP compliance. Use when configuring Vite for TYPO3 projects, setting up SCSS with Bootstrap, creating entrypoints per CE, optimizing SVGs, configuring PostCSS (autoprefixer, cssnano), loading local fonts, setting up CSS units, customizing Bootstrap variables. Also triggers for: asset hashing, Gzip/Brotli compression, SCSS import chain, global-basics.scss, selective Bootstrap imports, post-v14 frontend build pipelines."
---

# TYPO3 Vite Skill

Vite 7 build configuration for TYPO3 v13 and **v14.3 LTS** sitepackage development with `praetorius/vite-asset-collector`.

> **v14 context**: TYPO3 v14.0 **removed** the core's built-in frontend CSS/JS concatenation and compression ([Breaking #108055](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-108055-RemovedFrontendAssetConcatenationAndCompression.html)) and CSS comment/whitespace stripping ([Breaking #107944](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-107944-FrontendCSSFileProcessingNoLongerRemovesCommentsAndWhitespaces.html)). `config.concatenateCss`/`compressCss`/`concatenateJs`/`compressJs` no longer have any effect. An external build tool (Vite / webpack / esbuild) is **required** for production-grade asset handling on v14.

## Key Concepts

### Entrypoint-per-CE Pattern

Each content element gets its own Vite entrypoint (`*.entry.ts`) that imports its SCSS and TypeScript. This enables automatic code splitting -- only the CSS/JS needed for visible content elements is loaded.

### Selective Bootstrap Imports

Never import Bootstrap as a whole. Import only the components you use (`bootstrap/scss/grid`, `bootstrap/scss/buttons`, etc.) to minimize CSS bundle size.

### SVG Optimization

Custom `SvgCopyOptimizePlugin` processes SVGs from `Resources/Private/Svg/` through SVGO and writes optimized files to `Resources/Public/Svg/`. Supports dev-mode file watching.

### CSP Compliance

Assets loaded via `<vite:asset>` ViewHelper automatically get nonce attributes for Content Security Policy compliance. No inline `<script>` or `<style>` tags needed.

## Technology Stack

| Layer | Technology |
|---|---|
| Build | Vite 7+ with `praetorius/vite-asset-collector` |
| CSS | Bootstrap 5.3+ (selective imports, custom theming) |
| PostCSS | autoprefixer + cssnano (production) |
| SCSS | Modern Compiler API (`api: 'modern-compiler'`) |
| SVG | Custom SVGO plugin (`SvgCopyOptimizePlugin`) |
| Compression | Gzip + Brotli (production) |
| Package Manager | npm, pnpm, or yarn |

## References

- `references/vite-configuration.md` -- Complete vite.config.ts, entrypoints, SVG plugin, CSP
- `references/scss-architecture.md` -- SCSS folder structure, import chain, naming conventions, CSS units
- `references/bootstrap-theming.md` -- Bootstrap variable customization per project
