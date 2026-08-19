<!-- markdownlint-disable -->

# Hardening Report: gacts--install-hurl/v1.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-hurl/v1.3.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct GitHub Actions expression interpolation inside a `run:` shell command. In `.github/workflows/release.yml`, the step that configures git and sets the remote URL interpolates `${{ github.actor }}` directly into the shell command string twice. An attacker who can control the actor name (e.g. via a fork/PR scenario) could inject shell metacharacters. Offending lines:
  `git config --local user.name "${{ github.actor }}"`
  `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
Fix: move these values into `env:` variables and reference them as `"$VAR"` in the shell.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced action tag is moved or overwritten.

`.github/workflows/release.yml`:
  - `actions/checkout@v6` (line 16)
  - `gacts/github-slug@v1` (line 17)

`.github/workflows/tests.yml`:
  - `actions/checkout@v4` (line 22)
  - `gacts/gitleaks@v1` (line 23)
  - `actions/checkout@v6` (line 28)
  - `actions/setup-node@v4` (line 29)
  - `actions/checkout@v6` (line 37)
  - `actions/setup-node@v4` (line 38)
  - `actions/upload-artifact@v7` (line 42)
  - `actions/checkout@v6` (line 51)
  - `actions/download-artifact@v8` (line 52)
  - `stefanzweifel/git-auto-commit-action@v7` (line 53)
  - `actions/checkout@v6` (line 64)

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:17`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:28`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:37`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:42`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:53`
- `.github/workflows/tests.yml:64`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` block, and not every job has its own `permissions:` block, meaning jobs run with the default (broad) repository token permissions.

`.github/workflows/release.yml`: No top-level `permissions:` and the sole job `update-git-tag` has no `permissions:` key. This job pushes tags using `secrets.GITHUB_TOKEN` and should declare minimal permissions (e.g. `contents: write`).

`.github/workflows/tests.yml`: No top-level `permissions:` and jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` have no `permissions:` key (only `commit-and-push-fresh-dist` declares permissions). All jobs should declare explicit minimal permissions.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files. (1) script-injection: moved github.actor and secrets.GITHUB_TOKEN out of run: shell strings into env: block variables GIT_ACTOR and GITHUB_TOKEN, referenced as plain shell vars. (2) unpinned-uses: pinned all 13 action references to full 40-char SHAs with original tag preserved as comment. (3) missing-permissions: added top-level permissions: {} to both files; added job-level minimal permissions (contents: write for tag-pushing job, contents: read for read-only jobs).

