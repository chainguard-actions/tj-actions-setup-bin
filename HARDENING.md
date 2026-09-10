<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In go-install.sh, the value of `$INPUT_REPOSITORY` (inherited from `inputs.repository`, an attacker-controlled input) is written directly to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`. While `basename` is applied earlier to strip path separators, it does NOT strip newline characters. A repository name containing a newline could inject additional key=value pairs into GITHUB_OUTPUT, allowing output poisoning.

Locations:

- `go-install.sh:96`

### github-env-injection (severity: high)

In rust-install.sh, the value of `$BINARY_PATH` (derived from `$INPUT_REPOSITORY`, which comes from the caller-controlled `inputs.repository` input) is written to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"`. A crafted repository name containing a newline character would survive the `basename` call and could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `rust-install.sh:265`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings:
1. go-install.sh (line 96): Replaced `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"` with a sanitized form using `safe_binary_path=$(printf '%s' "$TMPDIR/$INPUT_REPOSITORY" | tr -d '\n\r')` followed by `echo "binary_path=$safe_binary_path" >> "$GITHUB_OUTPUT"`.
2. rust-install.sh (line 265): Replaced `echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT"` with a sanitized form using `safe_binary_path=$(printf '%s' "$BINARY_PATH" | tr -d '\n\r')` followed by `echo "binary_path=$safe_binary_path" >> "$GITHUB_OUTPUT"`.
Both fixes strip newline and carriage-return characters from the attacker-controlled value before writing to GITHUB_OUTPUT, preventing output poisoning.

