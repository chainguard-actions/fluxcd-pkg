<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kustomize/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kustomize/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download the binary to the runner's cache dir' breaks rule (a), direct expression interpolation, in two places: line 18 `VERSION=${{ inputs.version }}` and line 20 `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. It also breaks rule (b), unquoted expansion of untrusted data, at line 26: `if [[ ! $VERSION = v* ]]; then`. $VERSION comes from inputs.version. Fix: pass the values through env: (e.g. VERSION: ${{ inputs.version }} and GH_TOKEN: ${{ github.token }}) and always double-quote them.

Locations:

- `action.yml:18`
- `action.yml:20`
- `action.yml:26`

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' runs `echo "$KUSTOMIZE_TOOL_DIR" >> "$GITHUB_PATH"` (line 82). KUSTOMIZE_TOOL_DIR is built at line 45 from VERSION (from inputs.version) and from the inherited RUNNER_TOOL_CACHE env var. Nothing removes newlines before the write. Fix: run `printf '%s' ... | tr -d '\n\r'` on the value first.

Locations:

- `action.yml:82`
- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

I've fixed all three findings in action.yml. Nothing was run to check the changes.

- **script-injection and static-inline-injection:** `inputs.version` and `github.token` now go through the step's env as INPUT_VERSION and GH_TOKEN, so no `${{ }}` expression is left in the run script. VERSION is now taken from INPUT_VERSION with all whitespace removed (`tr -d '[:space:]'`). The `v*` prefix check now quotes `$VERSION`, written `[[ ! "$VERSION" == v* ]]`.
- **github-env-injection:** the folder path has newlines and carriage returns removed (`printf '%s' | tr -d '\n\r'`) before it is written to `$GITHUB_PATH`.

One behaviour change: upstream's unquoted `VERSION=${{ inputs.version }}` would fail on a version containing a space. Now the spaces are removed instead, so a value like `5. 3.0` becomes `5.3.0` and installs.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN. The script now reads them as double-quoted shell variables. Also quoted $VERSION in the v* prefix check. The script behaves the same as before.

