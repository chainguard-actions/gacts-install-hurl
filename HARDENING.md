<!-- markdownlint-disable -->

# Hardening Report: gacts--install-hurl/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-hurl/v1.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are directly interpolated inside run: shell commands in release.yml. Line 20: `git config --local user.name "${{ github.actor }}"` — github.actor is attacker-controllable (e.g. via a crafted username) and is injected directly into the shell command without quoting through an env var. Line 21: `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"` — same issue with github.actor. These expressions are expanded by the Actions template engine before the shell ever sees them, allowing shell metacharacter injection.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files use mutable tag refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. In release.yml: `actions/checkout@v4` (line 15), `gacts/github-slug@v1` (line 16). In tests.yml: `actions/checkout@v4` (lines 24, 29, 40, 57), `gacts/gitleaks@v1` (line 25), `actions/setup-node@v4` (lines 30, 41), `actions/upload-artifact@v4` (line 43), `actions/download-artifact@v4` (line 58), `stefanzweifel/git-auto-commit-action@v6` (line 59).

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:40`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:57`
- `.github/workflows/tests.yml:58`
- `.github/workflows/tests.yml:59`

### missing-permissions (severity: medium)

release.yml has no top-level permissions: key and the only job (update-git-tag) also has no job-level permissions: key. This means the workflow runs with the default (broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level permissions: key, and the jobs gitleaks (line 20), eslint (line 27), dist-built (line 34), and run-this-action (line 62) each have no job-level permissions: key. Only the commit-and-push-fresh-dist job defines its own permissions. The remaining jobs run with default (broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across release.yml and tests.yml:

1. script-injection (release.yml): Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline shell commands into the step's `env:` block as GIT_ACTOR and GITHUB_TOKEN. Shell script now references them as plain env vars.

2. unpinned-uses: Pinned all 7 action references to full 40-char commit SHAs with original tags preserved as comments: actions/checkout@11d5960a, gacts/github-slug@83cd3d9, gacts/gitleaks@c9a0338, actions/setup-node@49933ea, actions/upload-artifact@ea165f8, actions/download-artifact@d3f86a1, stefanzweifel/git-auto-commit-action@778341a.

3. missing-permissions (release.yml): Added top-level `permissions: {}` and job-level `permissions: contents: write` (needed for git push/tag operations).

4. missing-permissions (tests.yml): Added top-level `permissions: {}` and job-level `permissions: contents: read` for gitleaks, eslint, dist-built, and run-this-action jobs. The commit-and-push-fresh-dist job already had appropriate permissions (contents: write, pull-requests: write) which were preserved.

