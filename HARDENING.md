<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable branch names (@main) or version tags (@v7) instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks where a compromised upstream repository could inject malicious code. Affected references:
- linting.yml: fabasoad/reusable-workflows/...wf-js-lint.yml@main, fabasoad/reusable-workflows/...wf-pre-commit.yml@main
- release.yml: fabasoad/reusable-workflows/...wf-github-release.yml@main
- security.yml: fabasoad/reusable-workflows/...wf-security-sast.yml@main
- sync-labels.yml: fabasoad/reusable-workflows/...wf-sync-labels.yml@main
- test-providers.yml: actions/checkout@v7
- test-source.yml: actions/checkout@v7
- unit-tests.yml: fabasoad/reusable-workflows/...wf-js-unit-tests.yml@main
- update-license.yml: fabasoad/reusable-workflows/...wf-update-license.yml@main

Locations:

- `.github/workflows/linting.yml:12`
- `.github/workflows/linting.yml:16`
- `.github/workflows/release.yml:11`
- `.github/workflows/security.yml:19`
- `.github/workflows/sync-labels.yml:12`
- `.github/workflows/test-providers.yml:46`
- `.github/workflows/test-source.yml:32`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:12`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings. Before the shell executes the command, GitHub substitutes the expression value as raw text, allowing an attacker who controls the value to inject arbitrary shell commands.

In test-providers.yml (Validate step, lines 56-57): `${{ matrix.source }}`, `${{ steps.result.outputs.text }}`, and `${{ matrix.expected }}` are interpolated directly into echo and test commands. While matrix values are defined in the workflow file itself, `steps.result.outputs.text` is action output that flows through untrusted translation input.

In test-source.yml (Prepare source step, line 34): `${{ matrix.source }}` is interpolated directly into an `if [ "${{ matrix.source }}" = "file" ]` shell test. Although the matrix values are ["file", "text"], the expression is still substituted before shell parsing.

In test-source.yml (Validate translated text step, lines 52-53): `${{ steps.translate.outputs.text }}` is interpolated directly into echo and test commands. This output comes from the translation action and could contain shell metacharacters.

Locations:

- `.github/workflows/test-providers.yml:56`
- `.github/workflows/test-providers.yml:57`
- `.github/workflows/test-source.yml:34`
- `.github/workflows/test-source.yml:52`
- `.github/workflows/test-source.yml:53`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 9 unpinned-uses locations by resolving full commit SHAs: fabasoad/reusable-workflows@main → @ecce8eb77aa808e37355819f2a4ac62deefd0aff (linting.yml x2, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml) and actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (test-providers.yml, test-source.yml). Fixed all 5 script-injection locations by moving ${{ }} expressions into step env: blocks and referencing them as plain shell variables: in test-providers.yml Validate step (MATRIX_SOURCE, RESULT_TEXT, MATRIX_EXPECTED) and in test-source.yml Prepare source step (MATRIX_SOURCE) and Validate translated text step (TRANSLATE_TEXT).

