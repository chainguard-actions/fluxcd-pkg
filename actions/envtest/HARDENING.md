<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--envtest/version/v0.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--envtest/version/v0.17.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Use specified version to install' puts an expression straight into the shell script: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Rule (a): the step 'Determine version to install' puts an expression straight into the shell script: `curl -fsSL -H "Authorization: token ${{github.token}}" ...`. Pass the token through env: instead.

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a): the step 'Install the specified version of envtest' puts an expression straight into the shell script without quotes: `K8S_VERSION=${{ inputs.k8s_version }}`.

Locations:

- `action.yml:69`

### github-env-injection (severity: high)

The step 'Use specified version to install' writes inputs.envtest_version straight to $GITHUB_ENV (`echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`) without the printf '%s' | tr -d '\n\r' cleanup. A newline in the input could set any environment variable.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The step 'Download the binary to the runner's cache dir' writes SETUPENVTEST_TOOL_DIR to $GITHUB_PATH without the printf '%s' | tr -d '\n\r' cleanup. That path is built from the inherited env var VERSION, which comes from a user input, so it is untrusted: `echo "$SETUPENVTEST_TOOL_DIR" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:64`

### github-env-injection (severity: high)

The step 'Install the specified version of envtest' writes `KUBEBUILDER_ASSETS='$ENVTEST_ASSETS_PATH'` to $GITHUB_ENV without the printf '%s' | tr -d '\n\r' cleanup. The value depends on inputs.k8s_version, because it is the output of setup-envtest run with that version.

Locations:

- `action.yml:79`

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

action.yml: moved inputs.envtest_version, github.token and inputs.k8s_version from the run: scripts into env: (INPUT_ENVTEST_VERSION, GH_TOKEN, INPUT_K8S_VERSION) and quoted them where they are used. Newlines are now stripped with printf '%s' | tr -d '\n\r' before writing VERSION to GITHUB_ENV, SETUPENVTEST_TOOL_DIR to GITHUB_PATH and KUBEBUILDER_ASSETS to GITHUB_ENV. The setup-envtest command substitution is still a bare assignment, so errexit still stops the step if it fails. No tests were run.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed the garbled end of action.yml (an env: block stuck inside the run script, the 'hell: bash' line, and two run: keys in one step) by making it a separate 'Install envtest binaries' step. That step uses shell: bash and sets env INPUT_K8S_VERSION: ${{ inputs.k8s_version }}, and the script now reads K8S_VERSION="$INPUT_K8S_VERSION". Making it a separate step also means the earlier GITHUB_PATH addition is in effect, so setup-envtest is on PATH. The KUBEBUILDER_ASSETS value now has newlines stripped (printf '%s' | tr -d '\n\r') before it is written to GITHUB_ENV.

