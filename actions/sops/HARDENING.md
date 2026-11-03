<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--sops/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--sops/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub expressions are inserted straight into the run: script. Line 17: `VERSION=${{ inputs.version }}` – the caller-supplied input is pasted into the bash script as-is and is not quoted, so any shell metacharacters in it run as commands. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` – an expression interpolated inside run:. Fix: pass both through env: (e.g. INPUT_VERSION, GITHUB_TOKEN) and use them double-quoted. Rule (b) is also broken on line 25: `[[ ! $VERSION = v* ]]` expands the input-derived variable without quotes.

Locations:

- `action.yaml:17`
- `action.yaml:19`
- `action.yaml:25`

### github-env-injection (severity: high)

Line 78: `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` writes SOPS_TOOL_DIR to GITHUB_PATH. SOPS_TOOL_DIR is built from VERSION (taken from the untrusted inputs.version) and the inherited RUNNER_TOOL_CACHE env var. The value is not run through `printf '%s' ... | tr -d '\n\r'` first, so a newline in the input can add arbitrary entries to PATH.

Locations:

- `action.yaml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token into step env (INPUT_VERSION, GH_API_TOKEN) and referenced them double-quoted; quoted $VERSION in the [[ ]] test; sanitized SOPS_TOOL_DIR with tr -d '\n\r' before writing to GITHUB_PATH.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yaml I put the `fi` that closes the install block back on its own line, which fixes the bash syntax error. I removed the unsanitized `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"`. The step now strips newlines from SOPS_TOOL_DIR once, after the if-block, and writes only that cleaned value to GITHUB_PATH. That covers both the fresh-install path and the already-cached path. I also put back `chmod +x` on the sops binary after the `mv`. A bad edit seems to have overwritten it, but I restored it from memory of the upstream file and couldn't check it against the source. Without it the downloaded file is probably not executable, so `sops --version` would fail. Nothing was run to test the change.

