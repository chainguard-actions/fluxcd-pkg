<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kustomize/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kustomize/version/v0.17.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download the binary to the runner's cache dir' step puts expressions straight into the run script. Line 18: `VERSION=${{ inputs.version }}`. Line 20: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Because inputs.version is substituted unquoted, a crafted version string can run arbitrary commands. Fix: pass both values through env: (for example INPUT_VERSION and GITHUB_TOKEN) and refer to them as "$INPUT_VERSION" and "$GITHUB_TOKEN". Rule (b) also applies at line 26: `[[ ! $VERSION = v* ]]` expands the input-derived variable without quotes. It should be quoted as "$VERSION".

Locations:

- `action.yml:18`
- `action.yml:20`
- `action.yml:26`

### github-env-injection (severity: high)

Line 82: `echo "$KUSTOMIZE_TOOL_DIR" >> "$GITHUB_PATH"`. KUSTOMIZE_TOOL_DIR is built at line 45 from VERSION, which comes from inputs.version, and from the inherited env var RUNNER_TOOL_CACHE. The value is written to $GITHUB_PATH without first running `printf '%s' ... | tr -d '\n\r'`, so embedded newlines could add extra PATH entries. Fix: `safe=$(printf '%s' "$KUSTOMIZE_TOOL_DIR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH"`.

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

I moved inputs.version and github.token out of the run script and into the step's env: block as INPUT_VERSION and GITHUB_TOKEN. The script now reads them as quoted shell variables: VERSION="$INPUT_VERSION", and the curl call uses "$GITHUB_TOKEN". I added quotes to $VERSION in the v* check. Before the tool directory is written to GITHUB_PATH, newlines are now stripped with printf | tr -d '\n\r'. Nothing was run or tested after these edits.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the broken lines at the end of the install step in action.yml. The `chmod +x "$KUSTOMIZE_TOOL_DIR/$KUSTOMIZE_EXEC_FILE"` command and the missing closing `fi` are back. I also removed the second write to GITHUB_PATH that used the raw value. The step now writes only one value to GITHUB_PATH: the tool directory with newlines and carriage returns stripped. Nothing was run to test this; the fix was checked by reading the edited file.

