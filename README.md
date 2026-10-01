<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->
# typo3-vite-skill

Vite build setup, SCSS architecture, and Bootstrap 5 theming for TYPO3 v13+ sitepackage development.

## Installation

### Claude Code Marketplace

```bash
claude install netresearch/typo3-vite-skill
```

### Composer

```bash
composer require netresearch/typo3-vite-skill
```

## References

| File | Description |
|---|---|
| `references/vite-configuration.md` | Complete vite.config.ts, entrypoints, SVG optimization plugin, CSP compliance |
| `references/scss-architecture.md` | SCSS folder structure, import chain, naming conventions, CSS units |
| `references/bootstrap-theming.md` | SCSS theming flow, CI-color mapping, selective Bootstrap imports |

## Contributing

Issues and pull requests are welcome. Every commit needs a `Signed-off-by` trailer (`git commit -s`), which the DCO check enforces, and a signature, which branch protection on `main` requires.

A change to `SKILL.md` or a reference that changes an answer the skill gives (a configuration option, a command, a warning) comes with a case in `evals/evals.json` that asserts the new answer.

### Checks

The repository ships no executable code, so it has no behavioural tests. The checks validate the skill's structure, its evaluation cases and the syntax of every file:

- **Skill Validation** (`.github/workflows/validate.yml`, calling `validate.yml` of `netresearch/skill-repo-skill`): skill structure and front matter (`validate-skill.sh`), manifest sync between `plugin.json` and `.claude-plugin/plugin.json`, plugin version format, markdownlint, yamllint, actionlint, JSON syntax, ShellCheck at severity `style`, ruff check and format, checkpoint schema. Steps that find no file of their kind (shell, Python, checkpoints) say so and pass.
- **Eval Validation** (`.github/workflows/eval-validate.yml`): checks the structure of `evals/evals.json` with `validate-evals.sh`. It does not run the prompts against a model.
- **Harness Verification** (`.github/workflows/harness-verify.yml`, pull requests only): `AGENTS.md` exists, stays under 150 lines and links only to files that exist.

Skill Validation and Eval Validation run on every pull request and on every push to `main`. To run the same checks locally:

```bash
pre-commit install --install-hooks   # once per clone
pre-commit run --all-files           # exit 0 = all hooks passed
```

The hooks in `.pre-commit-config.yaml` run the same linters, the skill validator and the version-parity check; they use `netresearch/skill-repo-skill` at the pinned `rev:`, while CI uses its `main`. The manifest-sync and eval checks have no hook; with a checkout of `netresearch/skill-repo-skill` at `<tools>`, run them from the root of this repository:

```bash
bash <tools>/skills/skill-repo/scripts/sync-plugin-manifest.sh --check
bash <tools>/skills/skill-repo/scripts/validate-evals.sh evals/evals.json
```

A failure names the file and the rule: an `MD…` rule for markdownlint, a yamllint rule name, an `ERROR:` line from `validate-skill.sh`, an `ERROR:` line followed by the drifting fields from `sync-plugin-manifest.sh`, a `FAIL:` line from `validate-evals.sh`. `WARNING:` and `WARN:` lines do not fail the check.

## Dependencies

- **Runtime:** none. The skill is text. The configuration in the references uses packages of the user's project (Vite and its plugins, SVGO, Bootstrap, `praetorius/vite-asset-collector`), which that project selects and locks.
- **Composer:** `composer.json` requires `netresearch/composer-agent-skill-plugin`, which installs the skill into a Composer project. There is no lock file; the package is consumed as a library.
- **CI:** the workflows call shared workflows in `netresearch/skill-repo-skill` and `netresearch/.github` by `@main`; those pin third-party actions by commit SHA and tools by version. The pre-commit hooks are pinned by `rev:` in `.pre-commit-config.yaml`.
- **Updates:** Renovate (`renovate.json`, extending the organisation preset `local>netresearch/renovate-config`) opens update pull requests, for example for the pre-commit hooks. `auto-merge-deps.yml` passes pull requests from Renovate and Dependabot to the shared auto-merge workflow in `netresearch/.github`.
- **Selection:** a new dependency is added only when the skill or its checks cannot work without it, and is declared where its consumer reads it (`composer.json` for Composer, `.pre-commit-config.yaml` for hooks).

## Governance and policies

This repository follows the Netresearch organisation policies:

- [Governance](https://github.com/netresearch/.github/blob/main/GOVERNANCE.md): ownership, roles, and how decisions are made and disputes resolved.
- [Roadmap](https://github.com/netresearch/.github/blob/main/ROADMAP.md): planned and explicitly excluded work for the coming year.
- [Handling of dependency and code analysis findings](https://github.com/netresearch/.github/blob/main/SECURITY.md#handling-of-dependency-and-code-analysis-findings): thresholds, deadlines and the exception process for dependency (SCA) and static analysis (SAST) findings.
- [Secret management](https://github.com/netresearch/.github/blob/main/SECURITY.md#secret-management): how CI and release credentials are stored, accessed and rotated.
- [Access roster](https://github.com/netresearch/.github/blob/main/docs/access-roster.md): who holds administrative and write access to this repository and the organisation.

The security assurance case for this skill (threat model, trust boundaries, countermeasures and limits) is in [`docs/SECURITY-ASSURANCE.md`](docs/SECURITY-ASSURANCE.md).

Checks that run on pull requests in this repository:

- Skill Validation (`validate.yml`), Eval Validation (`eval-validate.yml`) and Harness Verification (`harness-verify.yml`), described under [Checks](#checks).
- DCO: every commit carries a `Signed-off-by` trailer.
- CodeQL default setup (a repository setting, not a workflow file) analyses the GitHub Actions workflows with the extended query suite.
- PR Quality Gates (`pr-quality.yml`, on `pull_request_target`): approves pull requests whose author has write, maintain or admin access.
- Auto-merge dependency PRs (`auto-merge-deps.yml`, on `pull_request_target`): approves and merges pull requests from Renovate and Dependabot through the shared workflow in `netresearch/.github`, and is skipped for every other author.
- A repository ruleset requests a Copilot code review when a pull request is opened for review or leaves draft; it does not require the review to pass.
- CodeRabbit (a GitHub App configured for the organisation, not a workflow file) reviews pull requests and reports a `CodeRabbit` status; it is not a required check.
- Branch protection on `main` requires Skill Validation, Eval Validation, CodeQL `Analyze (actions)` and DCO to pass on a branch that is up to date with `main`, and requires signed commits; it is not enforced for repository admins.
- No workflow here runs dependency review, a dependency audit, Opengrep or Betterleaks. Secret detection is GitHub secret scanning with push protection, which is enabled for this repository.

## License

This project uses split licensing:

- Code (`scripts/**`, `.github/workflows/**`, config files) is licensed under the [MIT License](LICENSE-MIT).
- Documentation and skill content (`skills/**`, `references/**`, `README.md`) is licensed under [CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0).

SPDX expression: `(MIT AND CC-BY-SA-4.0)`.

## Maintainer

Maintained by [Netresearch DTT GmbH](https://www.netresearch.de).
