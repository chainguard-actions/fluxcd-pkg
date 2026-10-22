<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--envtest/version/v0.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--envtest/version/v0.19.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): an input expression is placed directly inside the run: script in step 'Use specified version to install': `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Rule (a): a GitHub expression is placed directly inside the run: script in step 'Determine version to install': `VERSION=$(curl -fsSL -H "Authorization: token ${{github.token}}" ...)`. Pass the token in through env: instead.

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a): an input expression is placed directly, without quotes, inside the run: script in step 'Install the specified version of envtest': `K8S_VERSION=${{ inputs.k8s_version }}`.

Locations:

- `action.yml:69`

### github-env-injection (severity: high)

Step 'Use specified version to install' writes inputs.envtest_version straight to $GITHUB_ENV without the printf '%s' | tr -d '\n\r' cleanup: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`. A newline in the input can inject any environment variable.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes SETUPENVTEST_TOOL_DIR to $GITHUB_PATH without the cleanup step: `echo "$SETUPENVTEST_TOOL_DIR" >> "$GITHUB_PATH"`. That value is built from the inherited env vars $VERSION (which comes from inputs.envtest_version through GITHUB_ENV) and $RUNNER_TOOL_CACHE, neither of which this script sets.

Locations:

- `action.yml:64`

### github-env-injection (severity: high)

Step 'Install the specified version of envtest' writes KUBEBUILDER_ASSETS to $GITHUB_ENV without the cleanup step: `echo "KUBEBUILDER_ASSETS='$ENVTEST_ASSETS_PATH'" >> "$GITHUB_ENV"`. The value is derived from inputs.k8s_version (through K8S_VERSION) and the inherited RUNNER_TOOL_CACHE.

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

I fixed all eight findings in action.yml. I haven't run the action since the change.

- **Expressions moved out of the scripts:** `inputs.envtest_version`, `github.token` and `inputs.k8s_version` are now passed in through each step's `env:` block. The scripts read them as `$INPUT_ENVTEST_VERSION`, `$GH_TOKEN` and `"$INPUT_K8S_VERSION"` (the last one is now quoted). There are no `${{ }}` expressions left inside any `run:` script.
- **Newlines stripped before writing:** three values now go through `printf '%s' | tr -d '\n\r'` before they are written: VERSION to $GITHUB_ENV, SETUPENVTEST_TOOL_DIR to $GITHUB_PATH, and KUBEBUILDER_ASSETS to $GITHUB_ENV.
- **Unchanged:** the KUBEBUILDER_ASSETS line still wraps the path in single quotes, as upstream did, so behaviour is the same. The output of `setup-envtest` is still saved with a plain assignment first and cleaned in a separate step, so a failure still stops the step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.envtest_version into env INPUT_ENVTEST_VERSION and stripped newlines before writing it to GITHUB_ENV; moved github.token into env GITHUB_TOKEN for the curl call.

