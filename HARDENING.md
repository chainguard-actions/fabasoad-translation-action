<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned refs found:
- linting.yml: `fabasoad/reusable-workflows/...@main` (two refs)
- release.yml: `fabasoad/reusable-workflows/...@main`
- security.yml: `fabasoad/reusable-workflows/...@main`
- sync-labels.yml: `fabasoad/reusable-workflows/...@main`
- test-providers.yml: `actions/checkout@v7`
- test-source.yml: `actions/checkout@v7`
- unit-tests.yml: `fabasoad/reusable-workflows/...@main`
- update-license.yml: `fabasoad/reusable-workflows/...@main`

Locations:

- `.github/workflows/linting.yml:14`
- `.github/workflows/linting.yml:18`
- `.github/workflows/release.yml:11`
- `.github/workflows/security.yml:19`
- `.github/workflows/sync-labels.yml:10`
- `.github/workflows/test-providers.yml:43`
- `.github/workflows/test-source.yml:36`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:11`

### script-injection (severity: high)

Rule (a) violation: `${{ }}` expressions are directly interpolated inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

In `test-providers.yml`, the 'Validate' step interpolates `${{ matrix.source }}`, `${{ steps.result.outputs.text }}`, and `${{ matrix.expected }}` directly in the shell script:
  `echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"` and `[ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;`

In `test-source.yml`, the 'Prepare source' step interpolates `${{ matrix.source }}` directly in the shell script:
  `if [ "${{ matrix.source }}" = "file" ]; then`
And the 'Validate translated text' step interpolates `${{ steps.translate.outputs.text }}`:
  `echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"` and `[ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;`

Locations:

- `.github/workflows/test-providers.yml:55`
- `.github/workflows/test-source.yml:46`
- `.github/workflows/test-source.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 9 unpinned action references by pinning to full commit SHAs: fabasoad/reusable-workflows@main → @ecce8eb77aa808e37355819f2a4ac62deefd0aff (linting.yml x2, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml) and actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (test-providers.yml, test-source.yml). Fixed all script injection issues by moving ${{ }} expressions out of run: shell strings into step env: blocks in test-providers.yml (Validate step: matrix.source, steps.result.outputs.text, matrix.expected) and test-source.yml (Prepare source step: matrix.source; Validate translated text step: steps.translate.outputs.text).

