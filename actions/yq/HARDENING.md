<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--yq/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--yq/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a), direct expression interpolation in a composite run: block. Line 17 `VERSION=${{ inputs.version }}` puts caller input straight into the bash script without quotes. A value like `1; malicious_cmd` runs as a command. Line 19 also interpolates `${{github.token}}` directly (`curl -fsSL -H "Authorization: token ${{github.token}}" ...`). Sub-rule (b): line 25 `[[ ! $VERSION = v* ]]` expands the input-derived variable without quotes. Fix: pass inputs.version and github.token through env: and use them as "$INPUT_VERSION" and "$GITHUB_TOKEN", with quotes.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 79 `echo "$YQ_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from the untrusted inputs.version (VERSION goes into YQ_TOOL_DIR="${RUNNER_TOOL_CACHE}/yq/${VERSION}/${OS}/${ARCH}") to $GITHUB_PATH. The value is not sanitized with printf '%s' ... | tr -d '\n\r' first, so a newline in the input could add extra PATH entries.

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token into the step env (INPUT_VERSION, GITHUB_TOKEN) and referenced them quoted. Quoted $VERSION in the v* test. Stripped CR/LF from YQ_TOOL_DIR before appending it to $GITHUB_PATH.

