<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable tags or branch names instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks. Failing references:
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main`, `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`
- release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`
- security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`
- sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`
- test-providers.yml: `actions/checkout@v6`
- test-source.yml: `actions/checkout@v6`
- unit-tests.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-unit-tests.yml@main`
- update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`

Locations:

- `.github/workflows/linting.yml:13`
- `.github/workflows/linting.yml:16`
- `.github/workflows/release.yml:10`
- `.github/workflows/security.yml:20`
- `.github/workflows/sync-labels.yml:10`
- `.github/workflows/test-providers.yml:51`
- `.github/workflows/test-source.yml:33`
- `.github/workflows/unit-tests.yml:14`
- `.github/workflows/update-license.yml:10`

### script-injection (severity: high)

Sub-rule (a): Direct GitHub Actions expression interpolation inside `run:` shell command strings. In test-providers.yml, the 'Validate' step interpolates `${{ matrix.source }}`, `${{ steps.result.outputs.text }}`, and `${{ matrix.expected }}` directly into shell commands. These values flow from the matrix (which could be influenced by a PR) and from step outputs, and are expanded by the template engine before the shell ever sees them, allowing shell metacharacter injection. Offending lines:
  `echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"` 
  `[ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;`

In test-source.yml, the 'Prepare source' step interpolates `${{ matrix.source }}` directly into a shell `if` condition, and the 'Validate translated text' step interpolates `${{ steps.translate.outputs.text }}` directly into shell commands. Offending lines:
  `if [ "${{ matrix.source }}" = "file" ]; then`
  `echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"` 
  `[ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;`

Locations:

- `.github/workflows/test-providers.yml:57`
- `.github/workflows/test-providers.yml:58`
- `.github/workflows/test-source.yml:34`
- `.github/workflows/test-source.yml:52`
- `.github/workflows/test-source.yml:53`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all 9 mutable action/workflow references to full SHA digests: fabasoad/reusable-workflows@main → @4e2279474e598bee3ae8ded28899a24bbc7bf971 (linting.yml x2, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml) and actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 (test-providers.yml, test-source.yml). Fixed all script injection issues by moving ${{ matrix.source }}, ${{ steps.result.outputs.text }}, ${{ matrix.expected }}, and ${{ steps.translate.outputs.text }} out of run: shell strings and into step env: blocks, referencing them as plain environment variables in the shell scripts.

