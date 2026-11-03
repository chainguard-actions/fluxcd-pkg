<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kustomize/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kustomize/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), direct expression interpolation in the 'Download the binary to the runner's cache dir' run: block. Line 18 `VERSION=${{ inputs.version }}` puts the caller-supplied input straight into the shell script, so a value like `1; malicious_cmd` runs as a command. Line 20 `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also interpolates an expression inside run:. Both values should be passed through env: (e.g. INPUT_VERSION, GITHUB_TOKEN) and referenced as double-quoted shell variables. Rule (b) is also violated: line 26 `if [[ ! $VERSION = v* ]]` expands the input-derived variable without quotes.

Locations:

- `action.yml:18`
- `action.yml:20`
- `action.yml:26`

### github-env-injection (severity: high)

Line 82 `echo "$KUSTOMIZE_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from the untrusted input inputs.version (via $VERSION) and the inherited $RUNNER_TOOL_CACHE, set on line 45 as `KUSTOMIZE_TOOL_DIR="${RUNNER_TOOL_CACHE}/kustomize/${VERSION}/${OS}/${ARCH}"`. The `printf '%s' ... | tr -d '\n\r'` sanitization step is not applied first, so a newline in the input could add arbitrary entries to PATH.

Locations:

- `action.yml:82`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version into INPUT_VERSION and github.token into GH_TOKEN_VALUE via env:. Quoted $VERSION in the v* check. Stripped newlines from KUSTOMIZE_TOOL_DIR (printf | tr -d '\n\r') before writing it to GITHUB_PATH.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step env (VERSION_INPUT, GH_TOKEN) and referenced them as quoted shell variables in action.yml.

