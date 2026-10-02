<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Both go-install.sh and rust-install.sh write the value of $INPUT_REPOSITORY (which is set from inputs.repository — an untrusted caller-controlled input) directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$INPUT_REPOSITORY" | tr -d '\n\r'). A malicious repository name containing newline characters could inject arbitrary key=value pairs into the GitHub output context, potentially overwriting other step outputs. The offending line in both scripts is: echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"

Locations:

- `go-install.sh:101`
- `rust-install.sh:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in both go-install.sh (line 101) and rust-install.sh (line 82). In each file, replaced the direct `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"` with a sanitized form: first assign `safe_repository=$(printf '%s' "$INPUT_REPOSITORY" | tr -d '\n\r')`, then use `echo "binary_path=$TMPDIR/$safe_repository" >> "$GITHUB_OUTPUT"`. This strips any embedded newline or carriage-return characters from the caller-controlled `INPUT_REPOSITORY` value before it is written to GITHUB_OUTPUT, preventing output context injection.

