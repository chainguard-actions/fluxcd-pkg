<!-- markdownlint-disable -->

# Hardening Report: fluxcd--pkg--actions--kubeconform/version/v0.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--pkg--actions--kubeconform/version/v0.20.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are pasted straight into the composite run: script. Line 17 has `VERSION=${{ inputs.version }}`, which is unquoted. A caller-supplied version string such as `1; malicious-cmd` would run as shell code. Line 19 has `curl -fsSL -H "Authorization: token ${{github.token}}" ...`, another expression inside run:. Rule (b): on line 25 the input-derived value is expanded without quotes, in `[[ ! $VERSION = v* ]]`. Fix: pass inputs.version and github.token in through env: (for example INPUT_VERSION and GH_TOKEN), and use them only as double-quoted "$INPUT_VERSION" / "$GH_TOKEN".

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:25`

### github-env-injection (severity: high)

The step writes `echo "$KUBECONFORM_TOOL_DIR" >> "$GITHUB_PATH"` with no sanitization. KUBECONFORM_TOOL_DIR is built from VERSION, which comes from the untrusted inputs.version, and from RUNNER_TOOL_CACHE, which the composite action inherits from the calling environment. A value containing a newline could add arbitrary directories to PATH. Fix: clean the value just before the write, e.g. safe=$(printf '%s' "$KUBECONFORM_TOOL_DIR" | tr -d '\n\r').

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

In action.yml, inputs.version and github.token now come in through env: as INPUT_VERSION and GH_TOKEN. They are used only as double-quoted variables. The check `[[ ! $VERSION = v* ]]` now quotes its variable. Newlines and carriage returns are stripped from the tool dir before it is written to GITHUB_PATH.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In action.yml, line 25 was a corrupted line with an unmatched double quote. I put back the intended `echo "Unable to determine kubeconform version"` and indented `exit 1` correctly. On line 28 I quoted `$VERSION` inside the `[[ ]]` test. I also quoted `$KUBECONFORM_EXEC_FILE` in the tar command. It holds a single fixed filename, so quoting it doesn't change how the command behaves.

