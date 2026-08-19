<!-- markdownlint-disable -->

# Hardening Report: gacts--install-hurl/v1.3.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-hurl/v1.3.4** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in both workflow files are pinned to mutable tags instead of full 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced actions are compromised or tags are moved. Failing references include: actions/checkout@v6, gacts/github-slug@v1, gacts/gitleaks@v1, actions/setup-node@v6, actions/upload-artifact@v7, actions/download-artifact@v8, stefanzweifel/git-auto-commit-action@v7.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:39`
- `.github/workflows/tests.yml:40`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:50`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:59`

### script-injection (severity: high)

Sub-rule (a) violation: `${{ github.actor }}` is directly interpolated inside a `run:` shell block in release.yml. `github.actor` is the username of whoever triggered the workflow and is attacker-controllable. It is embedded directly into shell commands without being routed through an env: variable and without quoting protection against shell metacharacters. Offending lines: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and its only job (`update-git-tag`) also has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level `permissions:` key, and four of its five jobs (`gitleaks`, `eslint`, `dist-built`, `run-this-action`) have no job-level `permissions:` block. Only the `commit-and-push-fresh-dist` job defines explicit permissions. The remaining jobs inherit the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all four findings across both workflow files:

1. **unpinned-uses**: Pinned all 7 action references to full 40-char SHAs with tag comments preserved: actions/checkout@df4cb1c (v6), gacts/github-slug@83cd3d9 (v1), gacts/gitleaks@c9a0338 (v1), actions/setup-node@2499707 (v6), actions/upload-artifact@043fb46 (v7), actions/download-artifact@3e5f45b (v8), stefanzweifel/git-auto-commit-action@4a55954 (v7).

2. **script-injection**: In release.yml, moved `${{ github.actor }}` out of the `run:` block into the step's `env:` block as `GIT_ACTOR`, then referenced `$GIT_ACTOR` in the shell script.

3. **missing-permissions (release.yml)**: Added `permissions: {}` at the workflow top level and `permissions: contents: write` at the job level (needed to push git tags).

4. **missing-permissions (tests.yml)**: Added `permissions: {}` at the workflow top level and explicit `permissions: {}` blocks to all four jobs that lacked them (gitleaks, eslint, dist-built, run-this-action). The commit-and-push-fresh-dist job already had appropriate permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/release.yml at line 27. Moved `${{ secrets.GITHUB_TOKEN }}` from the `run:` shell command string into the step's `env:` block as `GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}`, and updated the shell command to reference it as `$GH_TOKEN` instead of the direct template expression. This ensures the token value is passed via the environment rather than being interpolated directly into the shell command string by YAML template substitution.

