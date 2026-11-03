<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--helm/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--helm/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) is violated in the step 'Download the binary to the runner's cache dir': expressions are placed directly into the run: script. Line 17: `VERSION=${{ inputs.version }}` (attacker-controllable input, also unquoted). Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Sub-rule (b) is also violated: the untrusted VERSION value is expanded unquoted on line 25, `if [[ ! $VERSION = v* ]]; then`. Fix: pass inputs.version and github.token in through env: and double-quote every expansion.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

The step 'Download the binary to the runner's cache dir' writes HELM_TOOL_DIR to $GITHUB_PATH without sanitizing it (`echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"`). HELM_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited env var RUNNER_TOOL_CACHE. The required `printf '%s' ... | tr -d '\n\r'` step is not applied before the write, so a newline in the value can add extra PATH entries.

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

Moved inputs.version and github.token into env (INPUT_VERSION, GH_TOKEN) and the script now refers to them as quoted variables. Quoted the $VERSION comparison. Newlines are now stripped from HELM_TOOL_DIR with printf | tr -d '\n\r' before it is written to $GITHUB_PATH. Upstream used no pipe-to-shell or other risky constructs, so I left the rest as it was.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Put back the missing `fi` that closes the `if [[ ! -x ... ]]` block. Removed the leftover text ('h.') after "$GITHUB_PATH" and took out the write of the unsanitized HELM_TOOL_DIR. The step now writes only SAFE_HELM_TOOL_DIR to $GITHUB_PATH, after stripping newlines with printf '%s' | tr -d '\n\r'.

