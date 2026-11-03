<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--yq/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--yq/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), expression placed directly in the script: the composite step's run: block puts expressions straight into the shell script. Line 17: `VERSION=${{ inputs.version }}`. The input is substituted into the script before bash runs, so a caller-supplied value like `1; malicious_cmd` gets executed. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression in run:. Both should go through env: variables (e.g. INPUT_VERSION, GH_TOKEN) and be used double-quoted. Rule (b), unquoted variable: line 25 `if [[ ! $VERSION = v* ]]; then` uses the input-derived VERSION without quotes. It should read "$VERSION".

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 79, `echo "$YQ_TOOL_DIR" >> "$GITHUB_PATH"`, writes YQ_TOOL_DIR to $GITHUB_PATH without the required `printf '%s' ... | tr -d '\n\r'` step. YQ_TOOL_DIR is built from the untrusted inputs.version value (VERSION) and the inherited RUNNER_TOOL_CACHE variable. A value with an embedded newline could add arbitrary directories to PATH for later steps.

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token out of the script into the step's env: block (INPUT_VERSION, GH_TOKEN), and quoted $VERSION in the v* test. The value written to $GITHUB_PATH now has \n and \r stripped (printf/tr) before it is written.

