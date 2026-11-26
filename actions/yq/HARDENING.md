<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--yq/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--yq/version/v0.21.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the 'Download the binary' step, GitHub expressions are pasted straight into the run: script. Line 17: `VERSION=${{ inputs.version }}`. This is an unquoted interpolation of a caller-controlled input, so a value like `1.0; malicious_cmd` runs as a shell command. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also uses an expression inside the script. Rule (b): line 25 `[[ ! $VERSION = v* ]]` expands the input-derived VERSION without quotes. Fix: add `env: VERSION: ${{ inputs.version }}` and `GITHUB_TOKEN: ${{ github.token }}` to the step, then use "$VERSION" and "$GITHUB_TOKEN" in double quotes.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

YQ_TOOL_DIR is built from VERSION (taken from inputs.version) and from the inherited RUNNER_TOOL_CACHE. It is written with `echo "$YQ_TOOL_DIR" >> "$GITHUB_PATH"` and is not cleaned first. A newline in the input could add arbitrary PATH entries. Fix: strip line breaks before the write, e.g. `safe=$(printf '%s' "$YQ_TOOL_DIR" | tr -d '\n\r')`, then write "$safe".

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

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GITHUB_TOKEN. The script now uses them as "$INPUT_VERSION" and "$GITHUB_TOKEN" in double quotes. Quoted $VERSION in the v* check. YQ_TOOL_DIR now has CR/LF stripped before it is written to GITHUB_PATH. Nothing has been run to test this.

