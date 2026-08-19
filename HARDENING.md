<!-- markdownlint-disable -->

# Hardening Report: gacts--install-hurl/v1.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-hurl/v1.3.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In release.yml, the step that configures git interpolates ${{ github.actor }} directly into the shell command string: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`. Any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value, allowing shell metacharacters to be injected.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:20`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files use mutable tag or version refs instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references in release.yml: `actions/checkout@v5`, `gacts/github-slug@v1`. Failing references in tests.yml: `actions/checkout@v4`, `gacts/gitleaks@v1`, `actions/checkout@v5` (multiple), `actions/setup-node@v4` (multiple), `actions/upload-artifact@v5`, `actions/download-artifact@v6`, `stefanzweifel/git-auto-commit-action@v7`.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:15`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:39`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:50`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:59`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and the only job (`update-git-tag`) also has no `permissions:` block. This means the workflow runs with the default (broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level `permissions:` key, and the jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` each have no job-level `permissions:` block. Only the `commit-and-push-fresh-dist` job has explicit permissions. The remaining jobs run with default (broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across release.yml and tests.yml:

1. script-injection (release.yml): Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline run: shell strings into the step's env: block as GIT_ACTOR and GH_TOKEN, then referenced them as plain shell variables.

2. unpinned-uses: Pinned all 8 action references to full 40-char commit SHAs with tag comments: actions/checkout@v4, actions/checkout@v5, gacts/github-slug@v1, gacts/gitleaks@v1, actions/setup-node@v4, actions/upload-artifact@v5, actions/download-artifact@v6, stefanzweifel/git-auto-commit-action@v7.

3. missing-permissions (release.yml): Added top-level `permissions: {}` and job-level `permissions: { contents: write }` for the update-git-tag job.

4. missing-permissions (tests.yml): Added top-level `permissions: {}` and per-job `permissions: { contents: read }` for gitleaks, eslint, dist-built, and run-this-action jobs. The commit-and-push-fresh-dist job already had appropriate permissions.

