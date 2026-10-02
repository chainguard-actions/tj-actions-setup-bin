<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In go-install.sh, the value of $INPUT_REPOSITORY (an inherited process env var set from the caller-controlled input `inputs.repository`) is written directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A caller supplying a repository name containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. The offending line is: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`

Locations:

- `go-install.sh:100`

### github-env-injection (severity: high)

In rust-install.sh, the value of $BINARY_PATH (derived from $INPUT_REPOSITORY, an inherited process env var set from the caller-controlled input `inputs.repository`) is written directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A caller supplying a repository name containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT. The offending line is: `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"`

Locations:

- `rust-install.sh:291`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. go-install.sh (line 100): Replaced `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"` with a sanitized version that strips newlines/carriage-returns using `safe_binary_path=$(printf '%s' "$TMPDIR/$INPUT_REPOSITORY" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
2. rust-install.sh (line 291): Replaced `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"` with a sanitized version that strips newlines/carriage-returns using `safe_binary_path=$(printf '%s' "$BINARY_PATH" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
Both fixes prevent a caller from injecting arbitrary key=value pairs into GITHUB_OUTPUT by supplying a repository name containing newline characters.

