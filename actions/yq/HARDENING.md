<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--yq/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--yq/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): expressions are placed directly inside the run: script of the 'Download the binary to the runner's cache dir' step. Line 17: `VERSION=${{ inputs.version }}`. The caller controls inputs.version, and the expression is substituted into the script before bash parses it, so a value like `1; malicious_cmd` runs as a command. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also interpolates an expression directly. Rule (b): line 25 `[[ ! $VERSION = v* ]]` expands the untrusted VERSION without double quotes. Fix: pass the values through env: entries (e.g. INPUT_VERSION: ${{ inputs.version }}, GH_TOKEN: ${{ github.token }}) and refer to them in double quotes ("$INPUT_VERSION", "$GH_TOKEN").

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

`echo "$YQ_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from untrusted data without sanitizing it. YQ_TOOL_DIR is made from VERSION, which comes from inputs.version or the GitHub API response, and from the inherited RUNNER_TOOL_CACHE environment variable. The required `printf '%s' ... | tr -d '\n\r'` step is missing, so newlines in the value could add extra entries to PATH. Fix: `safe=$(printf '%s' "$YQ_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH"`.

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

Moved inputs.version and github.token into env (INPUT_VERSION, GH_TOKEN) and referenced them quoted; quoted $VERSION in the [[ ]] test; YQ_TOOL_DIR is stripped of CR/LF before it is written to $GITHUB_PATH.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

action.yml: the version input now has newlines stripped as soon as it is read (printf | tr -d '\n\r'), so the install directory matches the path that gets written. YQ_TOOL_DIR is also cleaned into $safe before it is written to $GITHUB_PATH. Nothing else changed.

