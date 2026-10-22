<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubectl/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubectl/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), expressions placed directly in a run: script. Line 17 has `VERSION=${{ inputs.version }}`, an unquoted input the caller controls, which allows command injection (e.g. `1; malicious_cmd`). Line 19 has `curl -fsSL -H "Authorization: token ${{github.token}}" ...`, which puts the token into the script text. Rule (b), unquoted untrusted variable: line 25 has `if [[ ! $VERSION = v* ]]; then`. Fix: move both values into an env: block (e.g. INPUT_VERSION, GH_TOKEN) and always quote them.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 74, `echo "$KUBECTL_TOOL_DIR" >> "$GITHUB_PATH"`, writes KUBECTL_TOOL_DIR to $GITHUB_PATH without the `printf '%s' ... | tr -d '\n\r'` sanitization step. KUBECTL_TOOL_DIR is built from VERSION, which comes from the untrusted inputs.version, and from the inherited RUNNER_TOOL_CACHE. If the value contains a newline, extra entries can be added to PATH.

Locations:

- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN, and the script now uses the quoted variables. Quoted $VERSION in the [[ ]] test and also quoted the openssl and cat arguments. The GITHUB_PATH write now strips newlines and carriage returns with printf '%s' | tr -d '\n\r'. Nothing was run to test these changes.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into step env (INPUT_VERSION, GH_TOKEN), referenced them as quoted shell variables, and quoted $VERSION in the v* glob test (pattern left unquoted to keep glob matching).

