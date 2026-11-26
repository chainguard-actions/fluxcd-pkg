<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--helm/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--helm/version/v0.21.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): expressions are placed straight into the composite step's run: script. Line 17 has `VERSION=${{ inputs.version }}`, so a crafted version input such as `1; malicious-cmd` runs as shell code. Line 19 has `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Rule (b): line 25 has the unquoted expansion `if [[ ! $VERSION = v* ]]; then`, and VERSION comes from the input. Fix: add `env: INPUT_VERSION: ${{ inputs.version }}` and `GH_TOKEN: ${{ github.token }}`, then use `VERSION="$INPUT_VERSION"`, `-H "Authorization: token $GH_TOKEN"` and `[[ ! "$VERSION" = v* ]]`.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 60, `echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"`, writes to $GITHUB_PATH without sanitizing first. HELM_TOOL_DIR is built from VERSION, which comes from `inputs.version`. A version containing newlines could add extra entries to PATH. Clean it right before the write: `safe=$(printf '%s' "$HELM_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:60`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN, and the script now references them as quoted shell variables. Quoted $VERSION in the v* check. HELM_TOOL_DIR now has newline and carriage-return characters stripped before it is written to $GITHUB_PATH. No tests were run, and the action's behaviour is otherwise unchanged.

