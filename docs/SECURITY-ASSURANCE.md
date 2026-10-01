<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# Security Assurance Case

This document states what users of the typo3-vite skill can and cannot expect in terms of security, and argues why the expectations hold. Every claim names the file that supports it. Vulnerabilities are reported privately as described in the [organisation security policy](https://github.com/netresearch/.github/blob/main/SECURITY.md).

## What the project ships

| Part | Files | Runs code? |
|------|-------|------------|
| Skill instructions | `skills/typo3-vite/SKILL.md` | No. Text an AI agent loads. |
| References | `skills/typo3-vite/references/*.md` | No. Text an AI agent loads. They contain example configuration (`vite.config.ts`, `svgo.config.js`, a `settings.php` excerpt, SCSS) and four shell commands that the agent or the user runs in their own TYPO3 project (see below). |
| Evaluation cases | `evals/evals.json` | No. Prompts and expected answer patterns, validated in CI; not loaded by the skill. |
| Manifests | `plugin.json`, `.claude-plugin/plugin.json`, `composer.json` | No. Package metadata. |

The skill contains no scripts and no executable program, so a user of the skill runs no code from this repository. Untrusted input is parsed only in CI: on every pull request the workflows here call shared workflows of `netresearch/skill-repo-skill`, which check out the pull request and run linters and validators over its Markdown, YAML and JSON files with `contents: read` (see requirement 5).

## Actors and trust boundaries

- **User**: asks for help with the Vite build of a TYPO3 sitepackage and decides which suggested changes to apply. Trusted.
- **AI agent**: loads `SKILL.md` and the references and writes or edits files in the user's project. It acts with the user's permissions and the tools the user's agent platform grants.
- **User's project**: the TYPO3 project with its `package.json`, `vite.config.ts`, `settings.php` and frontend sources. The configuration from the references ends up here.
- **Package registries**: npm and Packagist, from which the user's project obtains Vite, its plugins and `praetorius/vite-asset-collector`. This repository does not install anything from them.
- **Maintainers and CI**: change and release this repository.

Boundary 1 lies between the skill text and the user's project: the agent turns the examples into files there, and the user's build tools execute them. Boundary 2 lies between this repository and the user's machine: releases are built and signed in CI.

## Security requirements

1. The skill contains no executable code of its own; every command and configuration it proposes is visible in the reference text.
2. The skill asks for, stores and transmits no credentials.
3. The frontend guidance keeps the Content Security Policy of the site intact.
4. The skill and its releases are delivered unmodified from this repository.
5. A pull request into `main` merges only with signed commits and with the checks that branch protection requires passing; repository admins can bypass this.

## Argument per requirement

### 1. No executable code of its own

`git ls-files` lists Markdown, JSON, JSONC, YAML and licence files and `.gitignore`; the Skill Validation job finds no shell script and no Python file to lint (`.github/workflows/validate.yml`). The commands the agent may run are in `references/vite-configuration.md`: the fenced `bash` blocks in "Flush the page cache after every build" (`vendor/bin/typo3 cache:flush`, a `curl` of the site's start page piped into `grep`, and `ls` of the build output) and the inline example `npm run hmr` in "HMR (Hot Module Replacement)", which starts the project's own dev-server script. The TypeScript, JavaScript and PHP blocks in the same file are configuration for the user's project, not code this repository runs.

### 2. No credentials

`SKILL.md` and the references ask for no token, password or key, and none of the commands takes one. The skill has no storage of its own.

### 3. Content Security Policy

- `SKILL.md` ("CSP Compliance") and `references/vite-configuration.md` ("CSP Compliance") tell the agent to load assets through the `<vite:asset>` ViewHelper, which adds the CSP nonce, so that no inline `<script>` or `<style>` tags are needed.
- The same section points CSP headers to TYPO3's Content-Security-Policy API or the web server configuration.

### 4. Delivered content is the reviewed content

- Releases are built by `.github/workflows/release.yml`, which calls the `netresearch/skill-repo-skill` release workflow with `id-token: write` and `attestations: write`. That workflow signs `SHA256SUMS.txt` keyless with `cosign sign-blob` and attests the release archives and checksums with `actions/attest-build-provenance`.
- The Skill Validation job checks that `plugin.json` and `.claude-plugin/plugin.json` agree and that the plugin version has a valid format.
- Branch protection on `main` requires signed commits.

### 5. Pull requests pass the required checks

Branch protection on `main` (a repository setting) requires a pull request, signed commits, and passing Skill Validation, Eval Validation, CodeQL `Analyze (actions)` and DCO checks on a branch that is up to date with `main`. It is not enforced for repository admins, so an admin can merge without them. Harness Verification runs on every pull request but is not a required check. A repository ruleset blocks deleting and force-pushing `main` and requests a Copilot review when a pull request is opened for review or leaves draft; it does not require that review to pass.

`validate.yml`, `eval-validate.yml` and `harness-verify.yml` grant `contents: read` only. Two workflows run on `pull_request_target` and call shared workflows that contain no checkout step, so they run no pull request code: `auto-merge-deps.yml` (shared workflow in `netresearch/.github`, acts only on pull requests opened by Renovate or Dependabot) and `pr-quality.yml` (shared workflow in `netresearch/skill-repo-skill`, approves pull requests whose author has write access, with `pull-requests: write`). The checks themselves are listed in [README.md](../README.md#governance-and-policies).

## Common weaknesses

| Weakness | Where it could arise | Countermeasure |
|----------|---------------------|----------------|
| CWE-79 cross-site scripting | Frontend assets of the user's site | The guidance loads every asset through `<vite:asset>` with a CSP nonce and uses no inline scripts or styles (`references/vite-configuration.md`, "CSP Compliance"). |
| CWE-942 permissive cross-domain policy | Vite dev server configuration in the user's project | `allowedHosts` and `cors` list the DDEV domain instead of `true`, so other websites cannot reach the dev server through DNS rebinding or cross-origin requests (`references/vite-configuration.md`, `SKILL.md` "Dev Server `allowedHosts` Trap"). |
| CWE-78 OS command injection | Shell commands in the references | The commands take no input from files or the network except the `<host>` placeholder the user fills in; no command string is evaluated. `npm run hmr` runs a script the user's own `package.json` defines. |
| CWE-798 credential exposure | Commits to this repository | GitHub secret scanning with push protection is enabled for the repository. No file reads or stores credentials. |
| CWE-829 inclusion of functionality from an untrusted source | CI workflows | The workflows call shared workflows inside the `netresearch` organisation; those pin third-party actions by commit SHA. |
| CWE-1104 unmaintained third-party components | Composer dependency and pinned tools | The only package dependency is `netresearch/composer-agent-skill-plugin`, required as `*` in `composer.json`, so every release satisfies the constraint and there is no version for Renovate to raise. Renovate opens update pull requests for the pre-commit hooks pinned in `.pre-commit-config.yaml` (`renovate.json`). |

## What the skill does not protect against

- **The user's dependencies.** The references name npm packages (Vite and its plugins, SVGO, Bootstrap) and `praetorius/vite-asset-collector` without pinning versions. The user's project selects, locks and audits them.
- **The development server.** The `server:` block in `references/vite-configuration.md` configures a local development server behind a reverse proxy. It restricts `allowedHosts` and `cors` to the DDEV domain and tells the agent never to use `allowedHosts: true` or `cors: true`; `evals/evals.json` asserts that answer. Beyond that, the skill does not cover hardening the server or exposing it outside a development environment.
- **Instructions inside project files.** The agent reads the project's files. The skill does not defend against text in those files that tries to steer the agent; that is the agent platform's responsibility.
- **`allowed-tools`.** `SKILL.md` declares none. Where a skill declares `allowed-tools`, it only pre-approves tools; it does not remove tools the agent already has.
