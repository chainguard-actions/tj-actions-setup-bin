<!-- markdownlint-disable -->

# Hardening Report: tj-actions--setup-bin/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--setup-bin/v1.2.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved. Unpinned references found:
- auto-approve.yml: `hmarr/auto-approve-action@v3`
- greetings.yml: `actions/first-interaction@v1`
- rebase.yml: `cirrus-actions/rebase@1.8`
- sync-release-version.yml: `tj-actions/release-tagger@v4`, `tj-actions/sync-release-version@v13`, `tj-actions/git-cliff@v1`, `peter-evans/create-pull-request@v5`
- test.yml: `reviewdog/action-shellcheck@v1`
- update-readme.yml: `tj-actions/auto-doc@v3`, `tj-actions/remark@v3`, `tj-actions/verify-changed-files@v16`, `peter-evans/create-pull-request@v5`

Locations:

- `.github/workflows/auto-approve.yml:8`
- `.github/workflows/greetings.yml:8`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:10`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:17`
- `.github/workflows/sync-release-version.yml:19`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-readme.yml:11`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:30`

### missing-permissions (severity: medium)

None of the workflow files define a `permissions:` block at the top level or at the job level. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. All six workflow files are affected: auto-approve.yml, greetings.yml, rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

Locations:

- `.github/workflows/auto-approve.yml:1`
- `.github/workflows/greetings.yml:1`
- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in test.yml directly interpolate GitHub Actions expressions inside shell command strings. The expressions `${{ steps.setup-bin-go.outputs.binary_path }}` and `${{ steps.setup-bin-rust.outputs.binary_path }}` are substituted into the shell script before execution. Although `steps.*.outputs.*` values are set by the action itself in this case, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it. Offending lines:
- `echo "setup-bin-go: ${{ steps.setup-bin-go.outputs.binary_path }}"`
- `echo "setup-bin-rust: ${{ steps.setup-bin-rust.outputs.binary_path }}"`

Locations:

- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:38`

### github-env-injection (severity: high)

Both go-install.sh and rust-install.sh write the value of `$INPUT_REPOSITORY` (an inherited process env var sourced from `inputs.repository` via the composite action's `env:` block) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker-controlled repository name containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. The offending lines are:
- go-install.sh: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`
- rust-install.sh: `echo "binary_path=$TMPDIR/$INPUT_REPOSITORY" >> "$GITHUB_OUTPUT"`

Locations:

- `go-install.sh:97`
- `rust-install.sh:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings: (1) Pinned 11 unpinned action references to full commit SHAs across 6 workflow files; (2) Added minimal permissions blocks to all 6 workflow files; (3) Moved ${{ steps.*.outputs.binary_path }} expressions from run: blocks into env: blocks in test.yml; (4) Sanitized $INPUT_REPOSITORY with printf/tr before writing to $GITHUB_OUTPUT in both go-install.sh and rust-install.sh.

