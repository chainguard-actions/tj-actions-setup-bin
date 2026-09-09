<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In go-install.sh, the value `$INPUT_REPOSITORY` — which is set by the calling workflow from `${{ inputs.repository }}` (an untrusted, workflow-controlled input) — is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious repository name containing newlines could inject arbitrary key=value pairs into the output environment. The offending line is: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`

Locations:

- `go-install.sh:97`

### github-env-injection (severity: high)

In rust-install.sh, the value `$BINARY_PATH` — which is derived from `$INPUT_REPOSITORY` (set by the calling workflow from `${{ inputs.repository }}`, an untrusted input) and `$TMPDIR` — is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious repository name containing newlines could inject arbitrary key=value pairs into the output environment. The offending line is: `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"`

Locations:

- `rust-install.sh:298`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. go-install.sh (line 97): Replaced `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"` with a two-step sanitization: `safe_binary_path=$(printf '%s' "$TMPDIR/$INPUT_REPOSITORY" | tr -d '\n\r')` followed by `echo "binary_path=$safe_binary_path" >> "$GITHUB_OUTPUT"`. This prevents a malicious repository name containing newlines from injecting arbitrary key=value pairs into the output environment.
2. rust-install.sh (line 298): Replaced `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"` with the same sanitization pattern using `printf '%s' "$BINARY_PATH" | tr -d '\n\r'` to strip any embedded newlines or carriage returns from the user-controlled value before writing to $GITHUB_OUTPUT.

