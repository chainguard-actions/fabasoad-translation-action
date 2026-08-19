<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable tags or branch names (@main, @v6) instead of immutable full 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the referenced ref is updated maliciously.

Failing references:
- linting.yml: fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main
- linting.yml: fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main
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
- `.github/workflows/release.yml:11`
- `.github/workflows/security.yml:20`
- `.github/workflows/sync-labels.yml:11`
- `.github/workflows/test-providers.yml:47`
- `.github/workflows/test-source.yml:33`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:11`

### script-injection (severity: high)

Sub-rule (a): The run: block in the 'Validate' step directly interpolates GitHub Actions expressions inside shell command strings. ${{ matrix.source }}, ${{ matrix.expected }}, and ${{ steps.result.outputs.text }} are substituted into the shell before execution. An attacker who can influence matrix values or step outputs could inject arbitrary shell commands. Offending lines:
  echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"
  [ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;

Locations:

- `.github/workflows/test-providers.yml:57`

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in test-source.yml directly interpolate GitHub Actions expressions inside shell command strings.

1. 'Prepare source' step: ${{ matrix.source }} is interpolated directly in a shell comparison:
   if [ "${{ matrix.source }}" = "file" ]; then
   An attacker controlling matrix.source could inject shell metacharacters.

2. 'Validate translated text' step: ${{ steps.translate.outputs.text }} is interpolated directly in shell commands:
   echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"
   [ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;
   A compromised translation provider output could inject shell commands.

Locations:

- `.github/workflows/test-source.yml:38`
- `.github/workflows/test-source.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 9 unpinned-uses findings by pinning fabasoad/reusable-workflows@main to SHA 10062f8186847226cb4865efbb8047795d372bae and actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803. Fixed 3 script-injection findings in test-providers.yml (matrix.source, steps.result.outputs.text, matrix.expected) and test-source.yml (matrix.source in Prepare source step, steps.translate.outputs.text in Validate step) by moving all ${{ }} expressions into step env: blocks and referencing them as plain shell variables.

