<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--envtest/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--envtest/version/v0.20.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Use specified version to install' step puts an expression straight into the run script: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`. Fix: pass the input in through an env: variable and quote it where it is used.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Rule (a): the 'Determine version to install' step puts an expression straight into the run script: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Fix: pass it in through an env: variable (e.g. GH_TOKEN) and reference it as "$GH_TOKEN".

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a): the 'Install the specified version of envtest' step puts an expression straight into the run script, unquoted: `K8S_VERSION=${{ inputs.k8s_version }}`. Shell metacharacters in the input would run as commands. Fix: pass it in through an env: variable and quote every use.

Locations:

- `action.yml:69`

### github-env-injection (severity: high)

The 'Use specified version to install' step writes inputs.envtest_version straight to $GITHUB_ENV without stripping newlines: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`. A newline in the input lets a caller set any environment variable. Fix: run the value through printf '%s' ... | tr -d '\n\r' before writing it.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The 'Download the binary' step writes SETUPENVTEST_TOOL_DIR to $GITHUB_PATH: `echo "$SETUPENVTEST_TOOL_DIR" >> "$GITHUB_PATH"`. That path is built from the inherited env vars RUNNER_TOOL_CACHE and VERSION, and VERSION comes from the input. Newlines are not stripped with printf '%s' ... | tr -d '\n\r' first.

Locations:

- `action.yml:64`

### github-env-injection (severity: high)

The 'Install the specified version of envtest' step writes KUBEBUILDER_ASSETS to $GITHUB_ENV: `echo "KUBEBUILDER_ASSETS='$ENVTEST_ASSETS_PATH'" >> "$GITHUB_ENV"`. The value is built from inputs.k8s_version and the inherited RUNNER_TOOL_CACHE, and newlines are not stripped with printf '%s' ... | tr -d '\n\r' first.

Locations:

- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.envtest_version }}" appears directly in run: block of step "Use specified version to install"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.k8s_version }}" appears directly in run: block of step "Install the specified version of envtest"; move to env: map

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

All findings in action.yml are fixed. Nothing was run to test the changes.

- **Expressions moved to env:** inputs.envtest_version is now read from INPUT_ENVTEST_VERSION, github.token from GH_TOKEN, and inputs.k8s_version from INPUT_K8S_VERSION. Each is double-quoted where the script uses it.
- **Newlines stripped before writes:** the VERSION and KUBEBUILDER_ASSETS values written to $GITHUB_ENV, and the setup-envtest tool directory written to $GITHUB_PATH, all go through printf '%s' | tr -d '\n\r' first.
- **Extra sanitizing:** the version looked up with curl is cleaned the same way before it is written, although no finding flagged it.
- **Error handling kept:** the setup-envtest path lookup is still assigned on its own before cleaning, so the step still stops if that command fails.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Put back the upstream comment and the 'go clean -cache' block that an earlier edit had broken. Removed the bad write to "$GITHUB_PATH"s. The GITHUB_PATH write now uses SETUPENVTEST_TOOL_DIR after newlines are stripped with printf | tr -d '\n\r'.

