<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

go-install.sh writes the user-controlled env var $INPUT_REPOSITORY (sourced from inputs.repository via the action.yml env: block) directly to $GITHUB_OUTPUT without sanitization. An attacker supplying a repository name containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning downstream step outputs. The required sanitization step (`printf '%s' "$INPUT_REPOSITORY" | tr -d '\n\r'`) is absent. Offending line: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`

Locations:

- `go-install.sh:97`

### github-env-injection (severity: high)

rust-install.sh writes $BINARY_PATH to $GITHUB_OUTPUT without sanitization. $BINARY_PATH is derived from $TMPDIR and $INPUT_REPOSITORY (sourced from inputs.repository via the action.yml env: block), making it transitively user-controlled. An attacker supplying a repository name containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$BINARY_PATH" | tr -d '\n\r'`) is absent. Offending line: `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"`

Locations:

- `rust-install.sh:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. go-install.sh (line 97): Replaced `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"` with a sanitized form using `safe_binary_path=$(printf '%s' "$TMPDIR/$INPUT_REPOSITORY" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
2. rust-install.sh (line 271): Replaced `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"` with a sanitized form using `safe_binary_path=$(printf '%s' "$BINARY_PATH" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
Both fixes strip newline and carriage-return characters from the user-controlled repository name before it reaches GITHUB_OUTPUT, preventing newline injection attacks.

