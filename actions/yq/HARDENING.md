<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--yq/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--yq/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) broken: expressions are placed straight into the run: script. Line 17: `VERSION=${{ inputs.version }}`. This is also unquoted, so whatever the caller passes as `version` is run as shell code. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Fix: move both into env: (for example INPUT_VERSION and GITHUB_TOKEN) and use "$INPUT_VERSION" and "$GITHUB_TOKEN". Rule (b) also broken: line 25 expands the input-derived variable without quotes: `if [[ ! $VERSION = v* ]]`.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 79 writes `echo "$YQ_TOOL_DIR" >> "$GITHUB_PATH"`. YQ_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited env vars RUNNER_TOOL_CACHE, RUNNER_OS and RUNNER_ARCH. Nothing removes newline characters before the write. Fix: sanitize the value right before writing it, e.g. `safe=$(printf '%s' "$YQ_TOOL_DIR" | tr -d '\n\r')`.

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

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GITHUB_TOKEN, and the script now uses them as quoted variables. Quoted $VERSION in the [[ ]] check; the v* pattern on the right is unchanged. Removed \n and \r from YQ_TOOL_DIR before it is written to $GITHUB_PATH. Nothing else in the script was changed.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token from the download step's run: script into its env: block as INPUT_VERSION and GH_TOKEN. The script now reads them as VERSION="$INPUT_VERSION" and "Authorization: token $GH_TOKEN", so it behaves the same as before.

