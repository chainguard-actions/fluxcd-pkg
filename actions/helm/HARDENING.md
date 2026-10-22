<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--helm/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--helm/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download the binary to the runner's cache dir' step puts GitHub expressions straight into the run: script. Line 17: `VERSION=${{ inputs.version }}` lets a caller-supplied version inject shell commands. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also uses template substitution inside the script. Rule (b): line 25 `if [[ ! $VERSION = v* ]]` expands the input-derived $VERSION without quotes. Fix: pass inputs.version and github.token in through env: and quote every expansion.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

HELM_TOOL_DIR is built from VERSION, which comes from the untrusted input inputs.version (`HELM_TOOL_DIR="${RUNNER_TOOL_CACHE}/helm/${VERSION}/${OS}/${ARCH}"`). Line 60 then writes it to $GITHUB_PATH (`echo "$HELM_TOOL_DIR" >> "$GITHUB_PATH"`) without first running `printf '%s' ... | tr -d '\n\r'`. A newline in the input could add arbitrary directories to PATH.

Locations:

- `action.yml:60`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

I fixed all three findings in `action.yml`. I did not run the action afterwards.

- **script-injection / static-inline-injection:** `inputs.version` and `github.token` now reach the download step through its `env:` block as `INPUT_VERSION` and `GH_TOKEN`. The script reads them as quoted variables: `VERSION="$INPUT_VERSION"`, and the curl header uses `${GH_TOKEN}`. The check `[[ ! $VERSION = v* ]]` now quotes `$VERSION`.
- **github-env-injection:** before the Helm directory is written to `$GITHUB_PATH`, newline and carriage-return characters are stripped from it with `printf '%s' | tr -d '\n\r'`.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

In action.yml, inputs.version and github.token now reach the 'Download the binary' step through env: as INPUT_VERSION and GITHUB_TOKEN. The script reads them as "$INPUT_VERSION" and "$GITHUB_TOKEN". I also quoted $VERSION in the v* prefix check.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Quoted $VERSION in the [[ ! "$VERSION" = v* ]] check in action.yml. The v* pattern stays unquoted so it still matches as a glob.

