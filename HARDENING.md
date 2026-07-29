<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--auth/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--auth/v3.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings, violating rule (a). This includes ${{ steps.auth-default.outputs.project_id }}, ${{ steps.auth-default.outputs.auth_token }}, ${{ vars.SECRET_NAME }}, ${{ steps.auth-access-token.outputs.project_id }}, and ${{ steps.auth-access-token.outputs.access_token }} embedded directly in curl URLs/headers and gcloud arguments. These values flow through YAML template substitution before the shell sees them, enabling script injection. Affected steps: 'oauth-federated-token' (direct WIF job), 'gcloud' (direct WIF job), 'gcloud' (WIF-through-SA job), 'oauth-token' (WIF-through-SA job), 'gcloud' (credentials_json job), 'access-token' (credentials_json job).

Locations:

- `.github/workflows/test.yml:100`
- `.github/workflows/test.yml:108`
- `.github/workflows/test.yml:138`
- `.github/workflows/test.yml:152`
- `.github/workflows/test.yml:185`
- `.github/workflows/test.yml:198`

### unpinned-uses (severity: high)

Several workflow files reference actions or reusable workflows by mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) draft-release.yml uses 'google-github-actions/.github/.github/workflows/draft-release.yml@v3' (tag, not SHA); (2) release.yml uses 'google-github-actions/.github/.github/workflows/release.yml@v3' (tag, not SHA); (3) test.yml uses 'google-github-actions/setup-gcloud@main' three times (branch name, not SHA).

Locations:

- `.github/workflows/draft-release.yml:16`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:143`
- `.github/workflows/test.yml:178`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed 6 script-injection instances in .github/workflows/test.yml by moving all ${{ }} expressions from run: shell strings into step env: blocks and referencing them as plain shell variables. Fixed 5 unpinned-uses instances: pinned google-github-actions/setup-gcloud@main to SHA 01af06c41d9aa0790d6fdc8f0b982b768ca4d102 (3 occurrences in test.yml), and pinned both google-github-actions/.github reusable workflow references (@v3) to SHA 29c6d38eeb974133b4b66401985f7c70cf4a6681 in draft-release.yml and release.yml.

