<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubeconform/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubeconform/version/v0.21.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub expressions are placed straight into the run: script of the 'Download the binary to the runner's cache dir' step. Line 17 has `VERSION=${{ inputs.version }}`, which is unquoted and attacker-controllable, so a crafted version input such as `1; malicious-cmd` runs as shell commands. Line 19 has `curl ... -H "Authorization: token ${{github.token}}" ...`. Fix: move both values into env: variables (e.g. INPUT_VERSION, GITHUB_TOKEN) and use them double-quoted. Rule (b): line 25 also has an unquoted expansion of the input-derived variable in `[[ ! $VERSION = v* ]]`. Quote it as "$VERSION".

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 83 runs `echo "$KUBECONFORM_TOOL_DIR" >> "$GITHUB_PATH"`. KUBECONFORM_TOOL_DIR is built from VERSION, which comes from inputs.version, and from the inherited RUNNER_TOOL_CACHE env var. The value is never passed through `printf '%s' ... | tr -d '\n\r'` before the write. A version value containing newlines could therefore add arbitrary directories to PATH.

Locations:

- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

I moved inputs.version and github.token out of the run: script into the step's env: block as INPUT_VERSION and GH_TOKEN_VALUE. The script now reads them as double-quoted shell variables. I also quoted $VERSION in the v* check. Before the tool directory is added to $GITHUB_PATH, newlines are now stripped from it with printf | tr -d '\n\r'. Everything else works as upstream did. I haven't run the action to test these changes.

