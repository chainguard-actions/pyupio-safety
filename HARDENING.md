<!-- markdownlint-disable -->

# Hardening Report: pyupio--safety/3.12.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyupio--safety/3.12.13** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a mutable Docker image tag instead of a SHA digest: `image: 'docker://pyupio/safety-v2-beta:latest'`. This is vulnerable to supply-chain attacks because the image content can change without notice.

Locations:

- `action.yml:56`

### unpinned-uses (severity: high)

build.yml uses tag-based (non-SHA) `uses:` references: `actions/checkout@v4`, `actions/setup-python@v4`, `docker/setup-buildx-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v4`. These should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/build.yml:10`
- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:22`
- `.github/workflows/build.yml:30`
- `.github/workflows/build.yml:43`

### unpinned-uses (severity: high)

issue_responder.yml uses a tag-based (non-SHA) `uses:` reference: `actions/checkout@v3`. This should be pinned to a full 40-character commit SHA.

Locations:

- `.github/workflows/issue_responder.yml:12`

### unpinned-uses (severity: high)

main.yml uses multiple tag-based (non-SHA) `uses:` references: `actions/checkout@v3`, `actions/setup-python@v4`, `slackapi/slack-github-action@v1.23.0`, `actions/upload-artifact@v3` (×4), `actions/checkout@v4`, `pypa/gh-action-pypi-publish@release/v1`, `actions/checkout@v2`, `ncipollo/release-action@v1`. All should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/main.yml:8`
- `.github/workflows/main.yml:10`
- `.github/workflows/main.yml:30`
- `.github/workflows/main.yml:40`
- `.github/workflows/main.yml:42`
- `.github/workflows/main.yml:55`
- `.github/workflows/main.yml:62`
- `.github/workflows/main.yml:69`
- `.github/workflows/main.yml:76`
- `.github/workflows/main.yml:88`
- `.github/workflows/main.yml:96`
- `.github/workflows/main.yml:103`
- `.github/workflows/main.yml:104`

### permissions (severity: medium)

build.yml has no top-level `permissions:` key and the single job `build-and-push` has no job-level `permissions:` key either. This means the job runs with the default (broad) token permissions.

Locations:

- `.github/workflows/build.yml:1`

### permissions (severity: medium)

main.yml has no top-level `permissions:` key. The jobs `test`, `notify`, and `build-binaries` have no job-level `permissions:` key, so they run with default (broad) token permissions. Only `deploy-pypi` and `create-gh-release` have job-level permissions.

Locations:

- `.github/workflows/main.yml:1`

### script-injection (severity: high)

Sub-rule (a): build.yml interpolates GitHub Actions expressions directly inside a `run:` shell command. The step `Log into registry` uses `${{ secrets.GITHUB_TOKEN }}` and `${{ github.actor }}` directly in the shell string: `echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u "${{ github.actor }}" --password-stdin`. Any expression inside `${{ }}` in a `run:` block is a script-injection risk.

Locations:

- `.github/workflows/build.yml:28`

### script-injection (severity: high)

Sub-rule (a): issue_responder.yml interpolates attacker-controlled GitHub context expressions directly inside `run:` shell commands. The step `Respond to new issues` uses `${{ github.event.issue.number }}` directly in the shell command: `gh issue comment ${{ github.event.issue.number }} --body "$BODY"`. Additionally, `${{ github.event.issue.user.login }}` is interpolated into the `BODY` env var value. Issue numbers and user logins are attacker-controlled via the `issues: opened` trigger.

Locations:

- `.github/workflows/issue_responder.yml:17`

### github-env-injection (severity: high)

build.yml writes `$package_version` (read from `safety/VERSION` via `cat safety/VERSION`) to `$GITHUB_ENV` without sanitization: `echo "SAFETY_VERSION=$package_version" >> $GITHUB_ENV`. The value is not passed through `printf '%s' ... | tr -d '\n\r'` before the write, allowing newline injection into the environment file if the VERSION file contains newlines.

Locations:

- `.github/workflows/build.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across 4 files:

1. **action.yml** (unpinned-uses): Pinned `docker://pyupio/safety-v2-beta:latest` to `docker://pyupio/safety-v2-beta:latest@sha256:a911aa6afee02c3cc4ff6a6a51c7bada392df8b775d32d22c2bdb3ef70040ec7`.

2. **build.yml** (unpinned-uses, permissions, script-injection, github-env-injection):
   - Pinned all 5 actions to full SHAs (checkout@v4, setup-python@v4, setup-buildx-action@v3, metadata-action@v5, build-push-action@v4)
   - Added top-level `permissions: {}` and job-level `permissions: {contents: read, packages: write}`
   - Fixed script injection in 'Log into registry' step by moving `secrets.GITHUB_TOKEN` and `github.actor` to env vars
   - Fixed github-env-injection by sanitizing VERSION with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV

3. **issue_responder.yml** (unpinned-uses, script-injection):
   - Pinned `actions/checkout@v3` to full SHA
   - Fixed script injection by moving `github.event.issue.number` and `github.event.issue.user.login` to env vars, then referencing them as `$ISSUE_NUMBER` and `$ISSUE_USER_LOGIN` in the shell script

4. **main.yml** (unpinned-uses, permissions):
   - Pinned all 9 action references to full SHAs (checkout@v3 ×3, setup-python@v4 ×3, slack-github-action@v1.23.0, upload-artifact@v3 ×4, checkout@v4, gh-action-pypi-publish@release/v1, checkout@v2, release-action@v1)
   - Added top-level `permissions: {}` and job-level permissions for `test` (contents: read), `notify` ({}), and `build-binaries` (contents: read) jobs

