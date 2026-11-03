<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubectl/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubectl/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), direct expression interpolation in a run: block of the 'Download the binary to the runner's cache dir' step. Line 17: `VERSION=${{ inputs.version }}` puts the input straight into the shell script, unquoted. A crafted version string can run arbitrary commands. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also puts an expression straight into the script. Fix: pass both values through env: (for example INPUT_VERSION and GITHUB_TOKEN) and reference them as "$INPUT_VERSION" and "$GITHUB_TOKEN".

Locations:

- `action.yml:17`
- `action.yml:19`

### github-env-injection (severity: high)

Line 74 writes `echo "$KUBECTL_TOOL_DIR" >> "$GITHUB_PATH"`. KUBECTL_TOOL_DIR is built on line 46 from ${RUNNER_TOOL_CACHE} (inherited env) and VERSION, which comes from inputs.version. Nothing strips newlines first with printf '%s' ... | tr -d '\n\r', so a value containing a newline could add extra PATH entries. Fix: safe=$(printf '%s' "$KUBECTL_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH".

Locations:

- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.version and github.token out of the run script and into the step's env as INPUT_VERSION and GITHUB_TOKEN. The script now reads them as quoted variables (VERSION="$INPUT_VERSION"). Before KUBECTL_TOOL_DIR is written to $GITHUB_PATH, newlines and carriage returns are now stripped from it with printf | tr -d '\n\r'. Nothing else in the script changed. I haven't run the action or the analysis again, so these fixes aren't tested.

