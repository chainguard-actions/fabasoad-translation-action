<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows and actions using mutable tags or branch names (@main, @v7) instead of full 40-character SHA commit digests. This exposes the workflows to supply-chain attacks where a compromised upstream repository could inject malicious code.

Failing references:
- linting.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main` and `uses: fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`
- release.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`
- security.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`
- sync-labels.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`
- test-providers.yml: `uses: actions/checkout@v7`
- test-source.yml: `uses: actions/checkout@v7`
- unit-tests.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-js-unit-tests.yml@main`
- update-license.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`

Locations:

- `.github/workflows/linting.yml:16`
- `.github/workflows/linting.yml:19`
- `.github/workflows/release.yml:12`
- `.github/workflows/security.yml:20`
- `.github/workflows/sync-labels.yml:12`
- `.github/workflows/test-providers.yml:47`
- `.github/workflows/test-source.yml:32`
- `.github/workflows/unit-tests.yml:15`
- `.github/workflows/update-license.yml:12`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside `run:` shell command strings. Before the shell executes the script, GitHub substitutes the expression value verbatim into the shell source, allowing an attacker to inject arbitrary shell commands via matrix values or step outputs.

In `.github/workflows/test-providers.yml` (lines 57-59), the `run:` block interpolates `${{ matrix.source }}`, `${{ steps.result.outputs.text }}`, and `${{ matrix.expected }}` directly into echo and test commands:
```
echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"
[ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;
```

In `.github/workflows/test-source.yml` (line 34), `${{ matrix.source }}` is interpolated directly into a shell `if` condition:
```
if [ "${{ matrix.source }}" = "file" ]; then
```

In `.github/workflows/test-source.yml` (lines 50-51), `${{ steps.translate.outputs.text }}` is interpolated directly into shell commands:
```
echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"
[ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;
```

Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g., `"$MATRIX_SOURCE"`) inside the `run:` block.

Locations:

- `.github/workflows/test-providers.yml:57`
- `.github/workflows/test-source.yml:34`
- `.github/workflows/test-source.yml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 9 unpinned action/workflow references by resolving them to full SHA digests: fabasoad/reusable-workflows@main → @10062f8186847226cb4865efbb8047795d372bae (linting.yml x2, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml) and actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (test-providers.yml, test-source.yml). Fixed all 3 script injection points by moving ${{ matrix.source }}, ${{ steps.result.outputs.text }}, ${{ matrix.expected }}, and ${{ steps.translate.outputs.text }} expressions into step-level env: blocks and referencing them as plain shell variables in the run: scripts.

