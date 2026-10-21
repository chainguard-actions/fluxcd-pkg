<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--helm/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--helm/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), direct expression interpolation in the 'Download the binary' run block. Line 17: `VERSION=${{ inputs.version }}` puts an attacker-controllable input straight into the bash script, unquoted. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression directly inside run:. Fix: move both values into env: (for example INPUT_VERSION and GITHUB_TOKEN) and refer to them as "$INPUT_VERSION" and "$GITHUB_TOKEN", always in double quotes.

Locations:

- `action.yml:17`
- `action.yml:19`

### github-env-injection (severity: high)

Line 60: `echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from untrusted sources without sanitizing it first. HELM_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited env var RUNNER_TOOL_CACHE. Fix: run `safe=$(printf '%s' "$HELM_TOOL_DIR" | tr -d '\n\r')` right before the write, then append "$safe" to "$GITHUB_PATH".

Locations:

- `action.yml:60`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GITHUB_TOKEN, and referenced them double-quoted. The value written to GITHUB_PATH now has newlines stripped with tr -d '\n\r'.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed the malformed if-block by removing the stray 'path.' token and adding the missing 'fi'. Removed the unsanitized echo to GITHUB_PATH, so the only write left is the sanitized one (tr -d '\n\r'). It runs after the if-block, as upstream did. Quoted $VERSION in the [[ ]] v* check.

