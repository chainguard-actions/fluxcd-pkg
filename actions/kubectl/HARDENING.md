<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubectl/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubectl/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download the binary to the runner's cache dir' breaks rule (a): expressions are placed directly in the run script. Line 17 `VERSION=${{ inputs.version }}` lets a caller's input inject shell commands, and line 19 `curl -fsSL -H "Authorization: token ${{github.token}}" ...` puts the token in through template substitution. It also breaks rule (b): variables built from the input are expanded without quotes, e.g. line 25 `if [[ ! $VERSION = v* ]]`, line 60 `openssl sha1 -sha256 ${KUBECTL_TARGET_FILE}`, line 61 `cat ${KUBECTL_SHA256_FILE}`, and line 49 `trap 'rm -rf $DL_DIR' EXIT`. Fix: pass inputs.version and github.token in through env: variables and put double quotes around every expansion.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`
- `action.yml:60`
- `action.yml:61`

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes `echo "$KUBECTL_TOOL_DIR" >> "$GITHUB_PATH"` (line 74). KUBECTL_TOOL_DIR is built at line 46 from VERSION (taken from inputs.version, which the caller controls) and from the inherited RUNNER_TOOL_CACHE. The value is not passed through `printf '%s' ... | tr -d '\n\r'` before the write, so a newline in it could add extra entries to PATH.

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

Moved inputs.version and github.token into the step's env: block as INPUT_VERSION and GH_TOKEN. Added double quotes to the expansions the finding named: $VERSION, the openssl/cat file paths and the trap's $DL_DIR. Before writing KUBECTL_TOOL_DIR to GITHUB_PATH, the script now removes newlines and carriage returns with printf | tr -d '\n\r'.

