<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--envtest/version/v0.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--envtest/version/v0.21.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the 'Use specified version to install' step, an expression is placed straight into the run script: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`. The input should be passed through an env: variable and expanded as a quoted shell variable instead.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Rule (a): in the 'Determine version to install' step, an expression is placed straight into the run script: `VERSION=$(curl -fsSL -H "Authorization: token ${{github.token}}" ...)`. The token should be passed through an env: variable (for example GH_TOKEN) and referenced as "$GH_TOKEN".

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a): in the 'Install the specified version of envtest' step, an expression is placed straight into the run script, and the result is also unquoted: `K8S_VERSION=${{ inputs.k8s_version }}`. Because it is an unquoted assignment, an input like `1.30; malicious_cmd` runs arbitrary commands. Pass it through env: and quote the expansion.

Locations:

- `action.yml:69`

### github-env-injection (severity: high)

In the 'Use specified version to install' step, inputs.envtest_version is written straight to $GITHUB_ENV with no sanitization: `echo "VERSION=${{ inputs.envtest_version }}" >> "$GITHUB_ENV"`. A value containing newlines could inject extra environment variables. Run it through `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

In the 'Download the binary' step, `echo "$SETUPENVTEST_TOOL_DIR" >> "$GITHUB_PATH"` writes a value built from the inherited env var ${VERSION} with no `printf '%s' ... | tr -d '\n\r'` sanitization. VERSION comes from inputs.envtest_version through GITHUB_ENV, and the step also uses ${RUNNER_TOOL_CACHE}. Newlines in VERSION could inject extra PATH entries.

Locations:

- `action.yml:64`

### github-env-injection (severity: high)

In the 'Install the specified version of envtest' step, `echo "KUBEBUILDER_ASSETS='$ENVTEST_ASSETS_PATH'" >> "$GITHUB_ENV"` writes a value with no `printf '%s' ... | tr -d '\n\r'` sanitization. The value comes from setup-envtest output, which is driven by the untrusted inputs.k8s_version.

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

All the fixes are in action.yml. I haven't run the action, so none of this is tested.

What changed:
- inputs.envtest_version, inputs.k8s_version and github.token now go through each step's env: block. The scripts use them as quoted variables: INPUT_ENVTEST_VERSION, INPUT_K8S_VERSION and GH_TOKEN.
- K8S_VERSION is now a quoted assignment, so a value like `1.30; cmd` can no longer run a command.
- Newlines and carriage returns are stripped with `printf '%s' | tr -d '\n\r'` before three writes: VERSION to GITHUB_ENV, the setup-envtest tool dir to GITHUB_PATH, and KUBEBUILDER_ASSETS to GITHUB_ENV.

Unchanged: the 'latest' defaults, the empty-value fallback, and the actions/setup-go step, which was already pinned to a commit SHA.

