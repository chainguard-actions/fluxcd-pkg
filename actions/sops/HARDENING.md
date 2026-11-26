<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--sops/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--sops/version/v0.21.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are placed straight into the run: script of the 'Download the binary to the runner's cache dir' step. Offending lines: `VERSION=${{ inputs.version }}` (line 17) and `curl -fsSL -H "Authorization: token ${{github.token}}" ...` (line 19). The inputs.version value is template-substituted unquoted into a shell assignment, so a crafted version string can run arbitrary commands. Fix: move inputs.version and github.token into the step's env: block (e.g. INPUT_VERSION, GH_TOKEN) and reference them as "$INPUT_VERSION" / "$GH_TOKEN". Related rule (b): the derived variable is also expanded unquoted in `[[ ! $VERSION = v* ]]` (line 25). Quote it as "$VERSION".

Locations:

- `action.yaml:17`
- `action.yaml:19`
- `action.yaml:25`

### github-env-injection (severity: high)

`echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from inputs.version (via VERSION) and the inherited RUNNER_TOOL_CACHE env var, with no `printf '%s' ... | tr -d '\n\r'` step before the write. A version input containing a newline could add arbitrary entries to PATH. Fix: safe=$(printf '%s' "$SOPS_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH".

Locations:

- `action.yaml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

I fixed all three findings in hardened/action/action.yaml. Nothing has been run or tested yet.

- **script-injection / static-inline-injection:** `inputs.version` and `github.token` now go into the step's `env:` block as `INPUT_VERSION` and `GH_TOKEN`. The script reads them as `VERSION="$INPUT_VERSION"` and in the curl header as `Authorization: token $GH_TOKEN`. `$VERSION` is also quoted in the `v*` check now; the `v*` pattern itself stays unquoted so it still works as a pattern.
- **github-env-injection:** newlines and carriage returns are removed from `SOPS_TOOL_DIR` with `printf '%s' | tr -d '\n\r'` before it is written to `$GITHUB_PATH`.

One small behaviour change: upstream pasted the version in without quotes, so a value with spaces would have broken the assignment. It is now kept as a single string. Normal version values work exactly as before.

