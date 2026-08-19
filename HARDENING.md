<!-- markdownlint-disable -->

# Hardening Report: fabasoad--translation-action/v4.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--translation-action/v4.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable tags or branch names (@main, @v6) instead of pinned full 40-character SHA commit digests. This exposes the workflows to supply-chain attacks where a compromised upstream repository could inject malicious code.

Failing references:
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main`
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`
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
- `.github/workflows/release.yml:11`
- `.github/workflows/security.yml:21`
- `.github/workflows/sync-labels.yml:11`
- `.github/workflows/test-providers.yml:38`
- `.github/workflows/test-source.yml:35`
- `.github/workflows/unit-tests.yml:16`
- `.github/workflows/update-license.yml:11`

### script-injection (severity: high)

GitHub Actions expression values are interpolated directly inside `run:` shell command strings (sub-rule a). Before the shell executes the script, the Actions runner performs template substitution of `${{ ... }}` expressions, allowing an attacker who can influence those values to inject arbitrary shell commands.

**test-providers.yml** — The 'Validate' step interpolates `${{ matrix.source }}`, `${{ steps.result.outputs.text }}`, and `${{ matrix.expected }}` directly into the shell script. `steps.result.outputs.text` is action output that could contain shell metacharacters.

Offending lines:
```
echo "'${{ matrix.source }}' has been translated to '${{ steps.result.outputs.text }}'"
[ "${{ steps.result.outputs.text }}" = "${{ matrix.expected }}" ] || exit 1;
```

**test-source.yml** — The 'Prepare source' step interpolates `${{ matrix.source }}` directly into a shell `if` condition, and the 'Validate translated text' step interpolates `${{ steps.translate.outputs.text }}` directly into the shell script.

Offending lines:
```
if [ "${{ matrix.source }}" = "file" ]; then
echo "'${TEXT}' has been translated to '${{ steps.translate.outputs.text }}'"
[ "${{ steps.translate.outputs.text }}" = "${EXPECTED}" ] || exit 1;
```

Locations:

- `.github/workflows/test-providers.yml:55`
- `.github/workflows/test-providers.yml:56`
- `.github/workflows/test-source.yml:42`
- `.github/workflows/test-source.yml:55`
- `.github/workflows/test-source.yml:56`

### github-env-injection (severity: high)

In test-source.yml, the 'Prepare source' step writes to $GITHUB_OUTPUT using a shell variable (`source`) whose value is derived from the directly-interpolated expression `${{ matrix.source }}`. Because `${{ matrix.source }}` is substituted into the shell script before execution (the script-injection vector), the resulting value of the `source` variable — and therefore the value written to $GITHUB_OUTPUT — can contain newlines or other control characters injected by an attacker who controls `matrix.source`. The write `echo "source=${source}" >> "$GITHUB_OUTPUT"` is not preceded by the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

Offending code:
```sh
if [ "${{ matrix.source }}" = "file" ]; then
  source="source.txt"
else
  source="${TEXT}"
fi
echo "source=${source}" >> "$GITHUB_OUTPUT"
```

Locations:

- `.github/workflows/test-source.yml:42`
- `.github/workflows/test-source.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 9 unpinned action references: fabasoad/reusable-workflows@main pinned to SHA 10062f8186847226cb4865efbb8047795d372bae in linting.yml (×2), release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml; actions/checkout@v6 pinned to SHA d23441a48e516b6c34aea4fa41551a30e30af803 in test-providers.yml and test-source.yml. Fixed script injection in test-providers.yml Validate step by moving matrix.source, steps.result.outputs.text, and matrix.expected into env vars. Fixed script injection in test-source.yml by moving matrix.source into MATRIX_SOURCE env var in Prepare source step, and steps.translate.outputs.text into TRANSLATE_TEXT env var in Validate step. Fixed github-env-injection in test-source.yml by sanitizing the source value with printf/tr before writing to GITHUB_OUTPUT.

