<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--auth/v2.1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--auth/v2.1.13** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate GitHub Actions expressions inside shell commands (sub-rule a). The `${{ steps.*.outputs.* }}` and `${{ vars.* }}` values are substituted into shell strings before the shell parses them, allowing an attacker who controls those values to inject arbitrary shell commands.

Affected steps and offending lines:
- 'oauth-federated-token' step: `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-default.outputs.project_id }}/secrets/${{ vars.SECRET_NAME }}/versions/latest:access` and `--header "Authorization: Bearer ${{ steps.auth-default.outputs.auth_token }}"`
- 'gcloud' step (direct_workload_identity_federation job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`
- 'oauth-token' step: `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-access-token.outputs.project_id }}/...` and `--header "Authorization: Bearer ${{ steps.auth-access-token.outputs.access_token }}"`
- 'gcloud' step (workload_identity_federation_through_service_account job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`
- 'access-token' step: `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-access-token.outputs.project_id }}/...` and `--header "Authorization: Bearer ${{ steps.auth-access-token.outputs.access_token }}"`
- 'gcloud' step (credentials_json job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`

Fix: Move all expression values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$VAR"`) inside the `run:` block.

Locations:

- `.github/workflows/test.yml:91`
- `.github/workflows/test.yml:101`
- `.github/workflows/test.yml:140`
- `.github/workflows/test.yml:155`
- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:213`

### unpinned-uses (severity: high)

Several workflow files reference external actions or reusable workflows using mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks where the referenced action could be silently replaced with malicious code.

Failing references:
- `.github/workflows/test.yml`: `uses: 'google-github-actions/setup-gcloud@main'` (3 occurrences — marked `# ratchet:exclude` but still unpinned)
- `.github/workflows/release.yml`: `uses: 'google-github-actions/.github/.github/workflows/release.yml@v3'`
- `.github/workflows/draft-release.yml`: `uses: 'google-github-actions/.github/.github/workflows/draft-release.yml@v3'`

Fix: Pin each reference to a full 40-character commit SHA, e.g. `uses: google-github-actions/setup-gcloud@<sha> # v<tag>`.

Locations:

- `.github/workflows/test.yml:97`
- `.github/workflows/test.yml:151`
- `.github/workflows/test.yml:209`
- `.github/workflows/release.yml:9`
- `.github/workflows/draft-release.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed 6 script-injection instances in .github/workflows/test.yml by moving all ${{ steps.*.outputs.* }} and ${{ vars.* }} expressions into step-level env: blocks and referencing them as plain shell variables (${VAR}) in run: scripts. Fixed 5 unpinned-uses instances: pinned 3 occurrences of google-github-actions/setup-gcloud@main to @01af06c41d9aa0790d6fdc8f0b982b768ca4d102 in test.yml, pinned google-github-actions/.github release.yml@v3 to @29c6d38eeb974133b4b66401985f7c70cf4a6681 in release.yml, and pinned google-github-actions/.github draft-release.yml@v3 to @29c6d38eeb974133b4b66401985f7c70cf4a6681 in draft-release.yml.

