<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubectl/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubectl/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download the binary to the runner's cache dir' breaks both sub-rules. (a) A ${{ }} expression is placed directly in the run: script. Line 17 has `VERSION=${{ inputs.version }}`, which an attacker who controls the input can use to run commands. Line 19 has `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. (b) Variables built from the input are expanded without quotes: `[[ ! $VERSION = v* ]]` (line 25), `openssl sha1 -sha256 ${KUBECTL_TARGET_FILE}` (line 60) and `cat ${KUBECTL_SHA256_FILE}` (line 61). The last two paths include VERSION. Fix: pass inputs.version and github.token through env: variables and double-quote every expansion.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`
- `action.yml:60`
- `action.yml:61`

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' builds KUBECTL_TOOL_DIR from the untrusted inputs.version ("${RUNNER_TOOL_CACHE}/kubectl/${VERSION}/${OS}/${ARCH}") and then runs `echo "$KUBECTL_TOOL_DIR" >> "$GITHUB_PATH"`. There is no `printf '%s' ... | tr -d '\n\r'` sanitization before the write, so newlines in the input can add arbitrary PATH entries.

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

Moved inputs.version and github.token into the step's env: as INPUT_VERSION and GH_TOKEN. Quoted the $VERSION test, the openssl/cat file paths and the trap's rm path. Removed newlines from KUBECTL_TOOL_DIR with printf | tr -d '\n\r' before it is written to GITHUB_PATH.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed corrupted lines in action.yml: restored 'echo "Adding kubectl to path"', placed SAFE_TOOL_DIR=$(printf '%s' "$KUBECTL_TOOL_DIR" | tr -d '\n\r') on its own line, and fixed the GITHUB_PATH write to 'echo "$SAFE_TOOL_DIR" >> "$GITHUB_PATH"' with balanced quotes.

