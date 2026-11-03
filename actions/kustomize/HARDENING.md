<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kustomize/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kustomize/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are placed directly in the composite step's run: script. Line 18: `VERSION=${{ inputs.version }}`, so a caller-supplied version string is substituted into the shell script as-is. Line 20: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Fix: pass both values through env: variables (e.g. INPUT_VERSION, GH_TOKEN) and use them double-quoted. Rule (b): line 26 `if [[ ! $VERSION = v* ]]` expands the input-derived variable without quotes.

Locations:

- `action.yml:18`
- `action.yml:20`
- `action.yml:26`

### github-env-injection (severity: high)

The step writes KUSTOMIZE_TOOL_DIR to $GITHUB_PATH (`echo "$KUSTOMIZE_TOOL_DIR" >> "$GITHUB_PATH"`) without sanitizing it first. The value is built at line 45 from VERSION (taken from inputs.version, or from the GitHub API response) and from the inherited RUNNER_TOOL_CACHE env var. A newline in either one could inject extra PATH entries. Fix: apply `printf '%s' "$KUSTOMIZE_TOOL_DIR" | tr -d '\n\r'` right before the write.

Locations:

- `action.yml:82`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN, and the script now uses them double-quoted. Quoted $VERSION in the v* check. Newlines and carriage returns are now stripped from KUSTOMIZE_TOOL_DIR before it is written to $GITHUB_PATH.

