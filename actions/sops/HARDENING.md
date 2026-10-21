<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--sops/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--sops/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are placed directly into the `run:` script of the 'Download the binary to the runner's cache dir' step. Line 17: `VERSION=${{ inputs.version }}`. An attacker-controlled input can inject shell commands here. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` puts the github.token expression straight into the script. Fix: move both into the step's `env:` block (e.g. INPUT_VERSION, GH_TOKEN) and refer to them as double-quoted variables. Rule (b), minor: line 25 `[[ ! $VERSION = v* ]]` leaves the input-derived variable unquoted; it should be `"$VERSION"`.

Locations:

- `action.yaml:17`
- `action.yaml:19`
- `action.yaml:25`

### github-env-injection (severity: high)

The 'Download the binary to the runner's cache dir' step runs `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` (line 78) without the `printf '%s' ... | tr -d '\n\r'` sanitization step. SOPS_TOOL_DIR is built from VERSION, which comes from the untrusted `inputs.version`, and from inherited env vars (RUNNER_TOOL_CACHE, RUNNER_OS, RUNNER_ARCH). A newline in the value could inject extra PATH entries. Fix: first set `safe=$(printf '%s' "$SOPS_TOOL_DIR" | tr -d '\n\r')`, then write "$safe" to $GITHUB_PATH.

Locations:

- `action.yaml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

All three findings are fixed in action.yaml. The file wasn't run after editing, so these fixes are untested.

- **script-injection and static-inline-injection:** `inputs.version` and `github.token` now go through the step's `env:` block as INPUT_VERSION and GH_TOKEN. The script reads them as `VERSION="$INPUT_VERSION"` and `Authorization: token $GH_TOKEN`, inside the double-quoted curl header. I also quoted `$VERSION` in `[[ ! "$VERSION" = v* ]]`. The `v*` pattern is still unquoted on purpose, because quoting it would turn it into a literal string match instead of a prefix check.
- **github-env-injection:** Newlines and carriage returns are now stripped from SOPS_TOOL_DIR with `printf '%s' | tr -d '\n\r'` before it is written to `$GITHUB_PATH`.

The install logic is otherwise unchanged.

