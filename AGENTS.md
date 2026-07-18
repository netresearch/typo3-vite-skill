# typo3-vite-skill

Vite 7 build configuration for TYPO3 v13/v14 LTS sitepackage development with `praetorius/vite-asset-collector`, distributed as a Claude Code skill. Covers SCSS architecture, Bootstrap 5.3 theming, SVG optimization, PostCSS, code splitting, and CSP compliance.

## Repo Structure

```
typo3-vite-skill/
├── skills/typo3-vite/          # Skill definition and references
│   ├── SKILL.md                # Skill metadata, trigger description, inline guidance
│   └── references/             # Extended documentation (vite config, SCSS, Bootstrap theming)
├── .claude-plugin/
│   └── plugin.json             # Plugin metadata (name, version, license)
├── .github/workflows/          # CI caller workflows (all call netresearch/skill-repo-skill reusable workflows)
├── composer.json               # Composer package definition (type: ai-agent-skill)
├── LICENSE-MIT                 # MIT license (applies to code, configs, CI workflows)
├── LICENSE-CC-BY-SA-4.0        # CC-BY-SA-4.0 (applies to skill content, docs, references)
└── README.md                   # Human-facing documentation
```

## Commands

No Makefile. Key operations:

- Install PHP dependencies: `composer install`
- Validate skill repo structure: run `skill-repo-skill`'s `validate-skill.sh` against repo root
- Release: bump `.claude-plugin/plugin.json` version, open PR, merge, signed tag `vX.Y.Z`, push tag — the `release.yml` caller builds the GitHub release

## Rules

- Skill behavior is defined by [skills/typo3-vite/SKILL.md](skills/typo3-vite/SKILL.md) — its `description` field must begin with `Use when` to activate correctly in Claude Code.
- Licensing follows the Netresearch split model: code under [LICENSE-MIT](LICENSE-MIT), documentation and skill content under [LICENSE-CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0). SPDX expression: `(MIT AND CC-BY-SA-4.0)`.
- Do not add a `version` field to [composer.json](composer.json) — versions are derived from git tags by the Release workflow.
- All CI workflows in [.github/workflows/](.github/workflows/) must remain thin callers of `netresearch/skill-repo-skill` reusable workflows — never inline actions in this repo.
- Release discipline: tag only **after** the bump PR is merged to `main`; tag-before-bump runs the release against the wrong version and produces a locked broken release.
- Vite is mandatory for TYPO3 v14 frontend assets: core asset concat/compression was removed in Breaking [#108055](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-108055-RemovedFrontendAssetConcatenationAndCompression.html). Skill content must reflect this as a non-optional build step.
- Conformance with the `skill-repo-skill` structural standard is enforced by the `validate.yml` caller on every PR.

## References

- [README.md](README.md) — human-facing documentation
- [skills/typo3-vite/SKILL.md](skills/typo3-vite/SKILL.md) — skill content and triggers
- `netresearch/skill-repo-skill` (external) — source of truth for reusable CI workflows and structural conventions
- `netresearch/typo3-frontend-patterns-skill` (external) — complementary frontend patterns that pair with the Vite build
