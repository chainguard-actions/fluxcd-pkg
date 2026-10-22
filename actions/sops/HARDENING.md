<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--sops/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--sops/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are pasted straight into the run: script of the 'Download the binary to the runner's cache dir' step. Line 17: `VERSION=${{ inputs.version }}` puts the caller-supplied input into the shell source, so a value like `1.0; malicious-cmd` runs as a command. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression directly into the script. Both values should be passed through env: (e.g. INPUT_VERSION, GITHUB_TOKEN) and referenced as quoted shell variables. Rule (b): line 25 `[[ ! $VERSION = v* ]]` expands the input-derived variable without quotes.

Locations:

- `action.yaml:17`
- `action.yaml:19`
- `action.yaml:25`

### github-env-injection (severity: high)

Line 78: `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` writes SOPS_TOOL_DIR to GITHUB_PATH. That value is built from VERSION (taken from inputs.version, line 17) and the inherited RUNNER_TOOL_CACHE env var (line 44). No `printf '%s' ... | tr -d '\n\r'` sanitization happens before the write, so a newline in the input could add extra PATH entries.

Locations:

- `action.yaml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

action.yaml: the step now reads inputs.version and github.token from env (INPUT_VERSION, GH_TOKEN) and uses them as quoted shell variables. The check `[[ ! $VERSION = v* ]]` now quotes "$VERSION". SOPS_TOOL_DIR goes through `printf '%s' | tr -d '\n\r'` before it is written to GITHUB_PATH. The finding refers to action.yml, but the only file is action.yaml, so that is the one I changed.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the broken install block: put back `chmod +x "$SOPS_TOOL_DIR/$SOPS_EXEC_FILE"` and the closing `fi`, and removed the stray 'i'. Removed the unsanitized `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` so the only write is SAFE_SOPS_TOOL_DIR, which has newlines and carriage returns stripped. Nothing was run to test this.

