<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubectl/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubectl/version/v0.21.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are put straight into the run: script of the 'Download the binary to the runner's cache dir' step. Line 17: `VERSION=${{ inputs.version }}` is unquoted, so a crafted version input such as `1.0; malicious-cmd` runs as shell code. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression in the script. Fix: pass both through env: (e.g. INPUT_VERSION, GITHUB_TOKEN) and quote them when used ("$INPUT_VERSION").

Locations:

- `action.yml:17`
- `action.yml:19`

### script-injection (severity: high)

Rule (b): variables derived from inputs.version are expanded without quotes. Line 60: `sum=$(openssl sha1 -sha256 ${KUBECTL_TARGET_FILE} | awk '{print $2}')`. Line 61: `expected_sum=$(cat ${KUBECTL_SHA256_FILE})`. Both paths are built from DL_DIR and KUBECTL_EXEC_FILE, not VERSION, but the rule asks for consistent quoting. Line 25: `if [[ ! $VERSION = v* ]]` leaves the input-derived VERSION unquoted. Quote every expansion ("$KUBECTL_TARGET_FILE", "$KUBECTL_SHA256_FILE", "$VERSION").

Locations:

- `action.yml:25`
- `action.yml:60`
- `action.yml:61`

### github-env-injection (severity: high)

Line 74: `echo "$KUBECTL_TOOL_DIR" >> "$GITHUB_PATH"` writes KUBECTL_TOOL_DIR to $GITHUB_PATH. That value is built from the untrusted inputs.version (VERSION) and the inherited RUNNER_TOOL_CACHE. No `printf '%s' ... | tr -d '\n\r'` step runs first, so a version input containing a newline could add arbitrary entries to PATH.

Locations:

- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN_VALUE, and the script now reads them as quoted shell variables. Added quotes around $VERSION, KUBECTL_TARGET_FILE, KUBECTL_SHA256_FILE and the DL_DIR inside the trap. Newlines and carriage returns are now stripped from KUBECTL_TOOL_DIR before it is written to $GITHUB_PATH. Nothing was run to test these changes.

