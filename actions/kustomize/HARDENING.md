<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kustomize/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kustomize/version/v0.21.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Download the binary to the runner's cache dir' puts GitHub expressions straight into the run: script. Line 18: `VERSION=${{ inputs.version }}`. This assignment is unquoted, so a crafted version input such as `x; malicious-cmd` runs as shell code. Line 20: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Both values should be passed in through env: variables (e.g. INPUT_VERSION, GITHUB_TOKEN) and expanded with double quotes. Rule (b): line 26 also expands the input-derived variable unquoted: `if [[ ! $VERSION = v* ]]`.

Locations:

- `action.yml:18`
- `action.yml:20`
- `action.yml:26`

### github-env-injection (severity: high)

Line 82 runs `echo "$KUSTOMIZE_TOOL_DIR" >> "$GITHUB_PATH"`. KUSTOMIZE_TOOL_DIR is built from VERSION, which comes from the untrusted inputs.version, and from the inherited env var RUNNER_TOOL_CACHE. Nothing runs `printf '%s' ... | tr -d '\n\r'` on it before the write, so a newline in the value can add extra entries to PATH.

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

In action.yml, I moved inputs.version and github.token out of the run: script and into the step's env: block as INPUT_VERSION and GH_TOKEN_VALUE, and the script now expands both inside double quotes. I also quoted $VERSION in the v* check. KUSTOMIZE_TOOL_DIR now has newlines stripped (printf | tr -d '\n\r') before it is written to GITHUB_PATH. The script's behavior is otherwise the same as upstream. I haven't run the action to test it.

