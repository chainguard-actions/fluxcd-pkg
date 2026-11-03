<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--envtest/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--envtest/version/v0.20.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Use specified version to install' puts an expression directly in the run: script: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Rule (a): the step 'Determine version to install' puts an expression directly in the run: script: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Pass the token in through an env: variable instead.

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a): the step 'Install the specified version of envtest' puts an expression directly in the run: script, and it is unquoted: `K8S_VERSION=${{ inputs.k8s_version }}`.

Locations:

- `action.yml:69`

### github-env-injection (severity: high)

The step 'Use specified version to install' writes inputs.envtest_version straight to $GITHUB_ENV (`echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`) with no printf '%s' | tr -d '\n\r' step first. A newline in the input could inject extra environment variables.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The step 'Download the binary to the runner's cache dir' builds SETUPENVTEST_TOOL_DIR from the inherited env vars VERSION (set from inputs.envtest_version) and RUNNER_TOOL_CACHE. Neither is computed in this script. It then writes the path to $GITHUB_PATH (`echo "$SETUPENVTEST_TOOL_DIR" >> "$GITHUB_PATH"`) without printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:64`

### github-env-injection (severity: high)

The step 'Install the specified version of envtest' writes ENVTEST_ASSETS_PATH to $GITHUB_ENV (`echo "KUBEBUILDER_ASSETS='$ENVTEST_ASSETS_PATH'" >> "$GITHUB_ENV"`) without printf '%s' | tr -d '\n\r'. That value is derived from inputs.k8s_version and the inherited RUNNER_TOOL_CACHE.

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

Changes to action.yml:
- inputs.envtest_version now comes in as the env var INPUT_ENVTEST_VERSION. Newlines are stripped before it is written to GITHUB_ENV.
- github.token now comes in as the env var GH_TOKEN for the curl call.
- inputs.k8s_version now comes in as the env var INPUT_K8S_VERSION and is assigned with quotes.
- SETUPENVTEST_TOOL_DIR has newlines stripped before it is written to GITHUB_PATH.
- ENVTEST_ASSETS_PATH has newlines stripped before it is written to GITHUB_ENV.
The action still behaves the same way, including the single quotes upstream puts around the KUBEBUILDER_ASSETS value.

