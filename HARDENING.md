<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside run: blocks. In test-providers.yml, the 'Validate' step interpolates ${{ matrix.source }}, ${{ steps.result.outputs.text }}, and ${{ matrix.expected }} directly into shell commands. matrix.* and steps.*.outputs.* are workflow-controllable contexts that are substituted by the YAML template engine before the shell sees them, enabling command injection. Offending lines:
  echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"
  [ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;

Locations:

- `.github/workflows/test-providers.yml:55`

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside run: blocks. In test-source.yml, the 'Prepare source' step interpolates ${{ matrix.source }} directly into a shell conditional, and the 'Validate translated text' step interpolates ${{ steps.translate.outputs.text }} directly into shell commands. Both matrix.* and steps.*.outputs.* are workflow-controllable contexts substituted before the shell executes them. Offending lines:
  if [ "${{ matrix.source }}" = "file" ]; then
  [ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;

Locations:

- `.github/workflows/test-source.yml:36`
- `.github/workflows/test-source.yml:52`

### github-env-injection (severity: high)

In test-source.yml, the 'Prepare source' step writes a value derived from ${{ matrix.source }} (a workflow-controllable matrix context) to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The variable 'source' is set based on the matrix.source expression and then written directly: echo "source=${source}" >> "$GITHUB_OUTPUT". A newline-containing matrix value could inject additional output variables.

Locations:

- `.github/workflows/test-source.yml:43`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable branch (@main) or tag (@v7) refs instead of immutable 40-character SHA commit digests. This exposes the workflows to supply-chain attacks if the referenced repositories are compromised or the refs are moved. Failing references:
- linting.yml: fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main
- linting.yml: fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main
- release.yml: fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main
- security.yml: fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main
- sync-labels.yml: fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main
- test-providers.yml: actions/checkout@v7
- test-source.yml: actions/checkout@v7
- unit-tests.yml: fabasoad/reusable-workflows/.github/workflows/wf-js-unit-tests.yml@main
- update-license.yml: fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main

Locations:

- `.github/workflows/linting.yml:13`
- `.github/workflows/linting.yml:17`
- `.github/workflows/release.yml:10`
- `.github/workflows/security.yml:22`
- `.github/workflows/sync-labels.yml:11`
- `.github/workflows/test-providers.yml:47`
- `.github/workflows/test-source.yml:30`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings across 7 workflow files:

1. script-injection (test-providers.yml): Moved matrix.source, steps.result.outputs.text, and matrix.expected from run: shell strings into an env: block, referencing them as plain shell variables.

2. script-injection (test-source.yml): Moved matrix.source into env: block in 'Prepare source' step; moved steps.translate.outputs.text into env: block in 'Validate translated text' step.

3. github-env-injection (test-source.yml): Added sanitization with `printf '%s' "${source}" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. unpinned-uses: Pinned actions/checkout@v7 to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 in test-providers.yml and test-source.yml. Pinned all fabasoad/reusable-workflows@main references to SHA ecce8eb77aa808e37355819f2a4ac62deefd0aff in linting.yml, release.yml, security.yml, sync-labels.yml, unit-tests.yml, and update-license.yml.

