<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--auth/v2.1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--auth/v2.1.13** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate GitHub Actions expressions (${{ ... }}) inside shell commands, violating rule (a). The expressions ${{ steps.auth-default.outputs.project_id }}, ${{ steps.auth-default.outputs.auth_token }}, ${{ vars.SECRET_NAME }}, ${{ steps.auth-access-token.outputs.project_id }}, and ${{ steps.auth-access-token.outputs.access_token }} are embedded directly in curl URL paths and Authorization headers. These values flow through YAML template substitution before the shell sees them, enabling script injection if any value contains shell metacharacters.

Locations:

- `.github/workflows/test.yml:97`
- `.github/workflows/test.yml:152`
- `.github/workflows/test.yml:213`

### unpinned-uses (severity: high)

Several workflow files reference actions/reusable workflows by mutable tags or branch names instead of full 40-character commit SHAs. Unpinned references are vulnerable to supply-chain attacks if the referenced tag or branch is updated with malicious code. Failing references: 'google-github-actions/.github/.github/workflows/draft-release.yml@v3' (draft-release.yml), 'google-github-actions/.github/.github/workflows/release.yml@v3' (release.yml), and 'google-github-actions/setup-gcloud@main' (test.yml, three occurrences).

Locations:

- `.github/workflows/draft-release.yml:17`
- `.github/workflows/release.yml:8`
- `.github/workflows/test.yml:103`
- `.github/workflows/test.yml:155`
- `.github/workflows/test.yml:216`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection in test.yml by moving all ${{ ... }} expressions from run: blocks into env: blocks and referencing them as plain shell variables (${VAR_NAME}). This affects the oauth-federated-token step (line 97), oauth-token step (line 152), access-token step (line 213), and also the gcloud steps that used ${{ vars.SECRET_NAME }} directly. Fixed unpinned-uses by pinning google-github-actions/setup-gcloud@main to full SHA 01af06c41d9aa0790d6fdc8f0b982b768ca4d102 (3 occurrences in test.yml), and pinning the reusable workflow references in draft-release.yml and release.yml from @v3 to full SHA 29c6d38eeb974133b4b66401985f7c70cf4a6681.

