<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Both go-install.sh and rust-install.sh write values derived from the untrusted input $INPUT_REPOSITORY (set from inputs.repository via the env: block in action.yml) to $GITHUB_OUTPUT without the required newline-stripping sanitization (printf '%s' "$VAR" | tr -d '\n\r'). In go-install.sh the write is: echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT". In rust-install.sh the write is: echo "binary_path=$BINARY_PATH" >> "$GITHUB_OUTPUT" where $BINARY_PATH is derived from $INPUT_REPOSITORY. While basename is applied to $INPUT_REPOSITORY at the top of each script, basename only strips path separators — it does NOT strip newline characters. A caller-controlled repository name containing a newline (e.g. "repo\nmalicious_key=injected_value") would inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps. The fix is to apply safe=$(printf '%s' "$INPUT_REPOSITORY" | tr -d '\n\r') before constructing the path written to $GITHUB_OUTPUT.

Locations:

- `go-install.sh:113`
- `rust-install.sh:338`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed newline injection vulnerability in both go-install.sh (line 113) and rust-install.sh (line 338). In each script, added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing the value derived from `$INPUT_REPOSITORY` to `$GITHUB_OUTPUT`. While `basename` strips path separators, it does not strip newline characters, so a malicious repository name containing a newline could inject additional key=value pairs into GITHUB_OUTPUT. The fix captures the sanitized value in a new variable (`safe_repository` / `safe_binary_path`) and uses that in the echo statement.

