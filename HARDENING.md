<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--auth/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--auth/v3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate ${{ ... }} expressions inside shell commands, violating sub-rule (a). The following steps embed GitHub Actions expressions directly in shell strings:

1. Step 'oauth-federated-token' (direct_workload_identity_federation job): `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-default.outputs.project_id }}/secrets/${{ vars.SECRET_NAME }}/versions/latest:access ... --header "Authorization: Bearer ${{ steps.auth-default.outputs.auth_token }}"`

2. Step 'gcloud' (direct_workload_identity_federation job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`

3. Step 'gcloud' (workload_identity_federation_through_service_account job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`

4. Step 'oauth-token' (workload_identity_federation_through_service_account job): `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-access-token.outputs.project_id }}/secrets/${{ vars.SECRET_NAME }}/versions/latest:access ... --header "Authorization: Bearer ${{ steps.auth-access-token.outputs.access_token }}"`

5. Step 'gcloud' (credentials_json job): `gcloud secrets versions access "latest" --secret "${{ vars.SECRET_NAME }}"`

6. Step 'access-token' (credentials_json job): `curl https://secretmanager.googleapis.com/v1/projects/${{ steps.auth-access-token.outputs.project_id }}/secrets/${{ vars.SECRET_NAME }}/versions/latest:access ... --header "Authorization: Bearer ${{ steps.auth-access-token.outputs.access_token }}"`

All these expressions should be moved to env: variables and the env vars should be double-quoted in the shell commands.

Locations:

- `.github/workflows/test.yml:88`
- `.github/workflows/test.yml:97`
- `.github/workflows/test.yml:131`
- `.github/workflows/test.yml:148`
- `.github/workflows/test.yml:168`
- `.github/workflows/test.yml:196`

### unpinned-uses (severity: high)

Several workflow files reference actions or reusable workflows by mutable tags or branch names instead of full 40-character commit SHAs:

- draft-release.yml: `uses: 'google-github-actions/.github/.github/workflows/draft-release.yml@v3'` — pinned to tag `v3`
- release.yml: `uses: 'google-github-actions/.github/.github/workflows/release.yml@v3'` — pinned to tag `v3`
- test.yml (3 occurrences): `uses: 'google-github-actions/setup-gcloud@main'` — pinned to branch `main`

These mutable refs can be silently updated to point to different (potentially malicious) commits. Each should be replaced with a full SHA pin (e.g. `uses: google-github-actions/setup-gcloud@<40-hex-sha> # vX.Y.Z`).

Locations:

- `.github/workflows/draft-release.yml:16`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:101`
- `.github/workflows/test.yml:135`
- `.github/workflows/test.yml:172`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed 6 script-injection instances in test.yml by moving all ${{ }} expressions from run: blocks into env: blocks and referencing them as plain shell variables. Fixed 5 unpinned-uses: google-github-actions/setup-gcloud@main pinned to SHA 01af06c41d9aa0790d6fdc8f0b982b768ca4d102 (3 occurrences in test.yml), and google-github-actions/.github workflows pinned to SHA 29c6d38eeb974133b4b66401985f7c70cf4a6681 in both draft-release.yml and release.yml.

