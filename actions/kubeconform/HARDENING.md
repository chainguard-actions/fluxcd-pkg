<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubeconform/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubeconform/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), expressions written straight into the run: script. Line 17: `VERSION=${{ inputs.version }}` lets the caller's input be pasted into the bash script as code. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` interpolates an expression into the shell command. Rule (b), unquoted variable holding input data. Line 25: `if [[ ! $VERSION = v* ]]` leaves $VERSION unquoted. Fix: pass inputs.version and github.token in through env: and always write the variables as "$VAR".

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 83: `echo "$KUBECONFORM_TOOL_DIR" >> "$GITHUB_PATH"`. KUBECONFORM_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited RUNNER_TOOL_CACHE. It is written to $GITHUB_PATH without first running `printf '%s' ... | tr -d '\n\r'`, so a newline in the value could add extra PATH entries.

Locations:

- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step env (INPUT_VERSION, GH_TOKEN) and referenced them as quoted shell variables; quoted $VERSION in the [[ ]] test; sanitized KUBECONFORM_TOOL_DIR with tr -d '\n\r' before writing to $GITHUB_PATH.

