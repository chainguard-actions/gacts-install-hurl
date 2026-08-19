<!-- markdownlint-disable -->

# Hardening Report: gacts--install-hurl/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-hurl/v1.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or version refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved.

In `.github/workflows/release.yml`:
- `uses: actions/checkout@v4` (line 15)
- `uses: gacts/github-slug@v1` (line 16)

In `.github/workflows/tests.yml`:
- `uses: actions/checkout@v4` (line 22)
- `uses: gacts/gitleaks@v1` (line 23)
- `uses: actions/checkout@v4` (line 27)
- `uses: actions/setup-node@v4` (line 28)
- `uses: actions/checkout@v4` (line 34)
- `uses: actions/setup-node@v4` (line 35)
- `uses: actions/upload-artifact@v4` (line 38)
- `uses: actions/checkout@v4` (line 47)
- `uses: actions/download-artifact@v4` (line 48)
- `uses: stefanzweifel/git-auto-commit-action@v5` (line 49)
- `uses: actions/checkout@v4` (line 57)

All should be pinned to full 40-character SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:27`
- `.github/workflows/tests.yml:28`
- `.github/workflows/tests.yml:34`
- `.github/workflows/tests.yml:35`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:48`
- `.github/workflows/tests.yml:49`
- `.github/workflows/tests.yml:57`

### script-injection (severity: high)

In `.github/workflows/release.yml`, the `run:` block of the `update-git-tag` job directly interpolates `${{ github.actor }}` into shell commands (sub-rule a: direct expression interpolation). Although `github.actor` is GitHub-controlled, any `${{ ... }}` expression inside a `run:` block is processed by the YAML template engine before the shell sees it, allowing injection of shell metacharacters if the value is attacker-influenced. The offending lines are:

```
git config --local user.name "${{ github.actor }}"
git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"
```

These should be moved to an `env:` block and referenced as `"$ACTOR"` in the shell script.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and several jobs also lack job-level `permissions:` blocks, meaning they run with the default (overly broad) repository token permissions.

- `.github/workflows/release.yml`: No top-level `permissions:` key and the single job `update-git-tag` has no job-level `permissions:` key.
- `.github/workflows/tests.yml`: No top-level `permissions:` key, and jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` all lack job-level `permissions:` blocks (only `commit-and-push-fresh-dist` has explicit permissions).

Each workflow should declare minimal required permissions at the top level or on every job.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in both workflow files:

1. **unpinned-uses**: Pinned all 7 unique action references to full 40-char SHA digests with original tags as comments. actions/checkout→11d5960a, gacts/github-slug→83cd3d95, gacts/gitleaks→c9a0338, actions/setup-node→49933ea5, actions/upload-artifact→ea165f8d, actions/download-artifact→d3f86a10, stefanzweifel/git-auto-commit-action→b863ae19.

2. **script-injection**: In release.yml, moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from the `run:` shell block into the step's `env:` block as `ACTOR` and `GITHUB_TOKEN`, then referenced them as plain env vars `$ACTOR` and `$GITHUB_TOKEN` in the shell script.

3. **missing-permissions**: Added `permissions: {}` at the top level of both workflow files. Added minimal job-level permissions: `contents: write` for the release job (needs to push tags), `contents: read` for gitleaks/eslint/dist-built/run-this-action jobs, and preserved the existing `contents: write` + `pull-requests: write` for commit-and-push-fresh-dist.

