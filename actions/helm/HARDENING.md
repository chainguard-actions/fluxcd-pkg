<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--helm/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--helm/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download the binary to the runner's cache dir' breaks rule (a): expressions are placed straight into the run: script. Line 17: `VERSION=${{ inputs.version }}`. This is unquoted, so a version string chosen by the caller can run arbitrary shell commands. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. It also breaks rule (b): line 25 `if [[ ! $VERSION = v* ]]` leaves the untrusted variable unquoted. Fix: pass inputs.version and github.token in through env: and always use them double-quoted.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes HELM_TOOL_DIR to $GITHUB_PATH without sanitizing it: `echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"`. HELM_TOOL_DIR is built from VERSION, which comes from the untrusted inputs.version, and from the inherited RUNNER_TOOL_CACHE env var. No `printf '%s' ... | tr -d '\n\r'` step runs before the write, so a newline in the value could inject extra PATH entries.

Locations:

- `action.yml:60`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

inputs.version and github.token now come in through the step's env: block as INPUT_VERSION and GH_TOKEN. Both are always used double-quoted, including in the `v*` check. HELM_TOOL_DIR now has newlines stripped with `printf '%s' | tr -d '\n\r'` before it is written to $GITHUB_PATH. I haven't run the action after these changes.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Put back the missing closing 'fi' for the install block, removed the broken partial edit with stray 'h.' text, and changed the unsafe 'echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"' into a write that strips newlines first (printf '%s' | tr -d '\n\r'). This matches the upstream logic.

