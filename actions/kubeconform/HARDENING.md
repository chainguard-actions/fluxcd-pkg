<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubeconform/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubeconform/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): expressions are pasted straight into the run: script of step 'Download the binary to the runner's cache dir'. Line 17: `VERSION=${{ inputs.version }}`. This is unquoted, so an attacker-controlled input like `1.0; malicious-cmd` would run as shell code. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Sub-rule (b): line 25 `if [[ ! $VERSION = v* ]]; then` expands the input-derived variable without quotes. Fix: pass inputs.version and github.token through env: variables (e.g. INPUT_VERSION, GH_TOKEN) and always double-quote them when expanded.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 83: `echo "$KUBECONFORM_TOOL_DIR" >> "$GITHUB_PATH"` writes to $GITHUB_PATH without sanitizing first. KUBECONFORM_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited RUNNER_TOOL_CACHE env var. No `printf '%s' ... | tr -d '\n\r'` step runs before the write, so a newline in the value could inject extra PATH entries.

Locations:

- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN, and referenced them as quoted shell variables. Quoted $VERSION in the v* check. The tool dir now has newlines stripped with printf | tr -d '\n\r' before it's written to $GITHUB_PATH.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Repaired the damaged lines at the end of the download step: put back the truncated `chmod +x "$KUBECONFORM_TOOL_DIR/$KUBECONFORM_EXEC_FILE"`, the missing closing `fi` and the `echo "Adding kubeconform to path"` line. Removed the unsanitized write to GITHUB_PATH. The tool dir is now cleaned with `printf '%s' | tr -d '\n\r'`, saved as SAFE_TOOL_DIR, and only that value is written to $GITHUB_PATH. Writing a plain value (no command substitution inside another call) keeps errexit behavior the same.

