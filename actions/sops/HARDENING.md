<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--sops/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--sops/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), expressions placed straight into a run: script: line 17 `VERSION=${{ inputs.version }}` drops the caller's input into the bash script unquoted, so a crafted version value (for example `1; malicious_cmd`) runs as shell code. Line 19 `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression in the script. Rule (b), unquoted expansion: line 25 `if [[ ! $VERSION = v* ]]` expands the input-derived variable without quotes. Fix: pass inputs.version and github.token through the step's env: block (for example VERSION_INPUT and GH_TOKEN), use "$VERSION_INPUT" and "$GH_TOKEN", and quote "$VERSION".

Locations:

- `action.yaml:17`
- `action.yaml:19`
- `action.yaml:25`

### github-env-injection (severity: high)

Line 78 `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` writes SOPS_TOOL_DIR to $GITHUB_PATH without cleaning it first. SOPS_TOOL_DIR is built at line 44 from VERSION (taken from inputs.version, or from the GitHub API response) and from the inherited RUNNER_TOOL_CACHE env var. The step never runs `printf '%s' ... | tr -d '\n\r'` before the write, so a value containing a newline could add extra lines to the special file. Fix: `safe=$(printf '%s' "$SOPS_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH"`.

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

I fixed all three findings in `action.yaml`. The finding refers to `action.yml`, but the only file in the action is `action.yaml`. I haven't run the action since these edits.

- **Script injection (both findings):** `inputs.version` and `github.token` now go through the step's `env:` block as `VERSION_INPUT` and `GH_TOKEN`. The script uses `VERSION="$VERSION_INPUT"` and `Authorization: token ${GH_TOKEN}`, and the test is now `[[ ! "$VERSION" = v* ]]`.
- **PATH write:** newlines and carriage returns are stripped from `SOPS_TOOL_DIR` with `printf '%s' ... | tr -d '\n\r'` before it is appended to `$GITHUB_PATH`.

Everything else, including the checksum check, is as upstream wrote it.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Restored the truncated `chmod +x "$SOPS_TOOL_DIR/$SOPS_EXEC_FILE"` line and closing `fi`, removed stray `i`, and replaced the unsanitized `echo "$SOPS_TOOL_DIR" >> "$GITHUB_PATH"` with a newline-stripped value via printf | tr -d '\n\r'.

