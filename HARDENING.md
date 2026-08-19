<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions and reusable workflows using mutable tags or branch names (@main, @v6) instead of pinned 40-character commit SHAs. This exposes the workflows to supply-chain attacks if the referenced repository is compromised or the tag is moved.

Failing references:
- linting.yml: fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main, fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main
- release.yml: fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main
- security.yml: fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main
- sync-labels.yml: fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main
- test-providers.yml: actions/checkout@v6
- test-source.yml: actions/checkout@v6
- unit-tests.yml: fabasoad/reusable-workflows/.github/workflows/wf-js-unit-tests.yml@main
- update-license.yml: fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main

Locations:

- `.github/workflows/linting.yml:13`
- `.github/workflows/linting.yml:16`
- `.github/workflows/release.yml:10`
- `.github/workflows/security.yml:20`
- `.github/workflows/sync-labels.yml:11`
- `.github/workflows/test-providers.yml:42`
- `.github/workflows/test-source.yml:33`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:11`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings. The contexts matrix.* and steps.*.outputs.* flow through YAML template substitution before the shell processes them, allowing shell metacharacter injection.

In test-providers.yml (Validate step run: block):
  echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"
  [ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;

In test-source.yml (Prepare source step run: block):
  if [ "${{ matrix.source }}" = "file" ]; then

In test-source.yml (Validate translated text step run: block):
  echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"
  [ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;

These should be moved to env: variables and referenced as quoted shell variables (e.g., "$MATRIX_SOURCE") instead.

Locations:

- `.github/workflows/test-providers.yml:55`
- `.github/workflows/test-source.yml:40`
- `.github/workflows/test-source.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by resolving to full 40-char commit SHAs: fabasoad/reusable-workflows@main → @10062f8186847226cb4865efbb8047795d372bae (linting.yml ×2, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml) and actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (test-providers.yml, test-source.yml). Fixed script injection in test-providers.yml (Validate step: moved matrix.source, steps.result.outputs.text, matrix.expected to env vars MATRIX_SOURCE, RESULT_TEXT, MATRIX_EXPECTED) and test-source.yml (Prepare source step: moved matrix.source to MATRIX_SOURCE; Validate translated text step: moved steps.translate.outputs.text to TRANSLATE_TEXT).

