<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubeconform/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubeconform/version/v0.19.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), direct expression interpolation in the composite run: block. Line 17: `VERSION=${{ inputs.version }}`. The caller-supplied input is pasted into the bash script before the shell runs, so a value like `1; malicious_cmd` runs as code. Line 19: `curl -fsSL -H "Authorization: token ${{github.token}}" ...` also interpolates an expression inside run:. Rule (b), unquoted expansion of untrusted data. Line 25: `if [[ ! $VERSION = v* ]]; then`, where VERSION comes from inputs.version. Fix: map inputs.version and github.token into env: variables (e.g. INPUT_VERSION, GH_TOKEN) and always reference them double-quoted.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

Line 83: `echo "$KUBECONFORM_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from untrusted sources without sanitizing it first. KUBECONFORM_TOOL_DIR (line 44) is built from VERSION, which comes from inputs.version, and from the inherited RUNNER_TOOL_CACHE env var. A newline in the input could inject extra PATH entries. Fix: apply `printf '%s' "$KUBECONFORM_TOOL_DIR" | tr -d '\n\r'` right before the write.

Locations:

- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download the binary to the runner's cache dir"; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

All changes are in action.yml. inputs.version and github.token now come in through env: as INPUT_VERSION and GH_TOKEN. The script assigns VERSION="$INPUT_VERSION" and uses ${GH_TOKEN} in the curl Authorization header. I quoted $VERSION in the v* prefix check. Newlines and carriage returns are now stripped from KUBECONFORM_TOOL_DIR before it is written to GITHUB_PATH. The action works the same as before when given normal values.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In action.yml, changed `[[ ! $VERSION = v* ]]` to `[[ ! "$VERSION" = v* ]]`. I also quoted two other variables that weren't flagged: $KUBECONFORM_EXEC_FILE in the tar command and $DL_DIR in the trap. Neither changes how the action behaves.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Quoted $VERSION in the version-prefix check in action.yml: [[ ! "$VERSION" = v* ]]. The v* glob is still unquoted, so the check works the same way.

