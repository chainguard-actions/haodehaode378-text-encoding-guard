<!-- markdownlint-disable -->

# Hardening Report: haodehaode378--text-encoding-guard/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haodehaode378--text-encoding-guard/v1.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses `actions/setup-python@v5`, which is pinned to a mutable tag rather than an immutable 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:21`

### script-injection (severity: high)

Sub-rule (a): Multiple GitHub Actions expressions are directly interpolated inside the `run:` shell block, allowing script injection. An attacker-controlled value in `inputs.ext`, `inputs.fix-gbk`, or `inputs.root` is expanded by the YAML template engine before the shell ever sees it, enabling arbitrary command injection.

Offending lines:
- Line 26: `if [ -n "${{ inputs.ext }}" ]; then`
- Line 27: `IFS=',' read -ra EXTS <<< "${{ inputs.ext }}"`
- Line 32: `if [ "${{ inputs.fix-gbk }}" = "true" ]; then`
- Line 35: `python "${{ github.action_path }}/scripts/check_mojibake.py" \`
- Line 36: `--root "${{ inputs.root }}" \`

Sub-rule (b): Shell variables `$FIX_FLAG` (line 37) and `$EXT_ARGS` (line 38) — both derived from attacker-controlled inputs — are expanded unquoted in the final `python` invocation, allowing shell metacharacter injection (word splitting, glob expansion, etc.).

Fix: Move all `inputs.*` values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$INPUT_EXT"`). Never interpolate `${{ ... }}` directly inside a `run:` block.

Locations:

- `action.yml:26`
- `action.yml:27`
- `action.yml:32`
- `action.yml:35`
- `action.yml:36`
- `action.yml:37`
- `action.yml:38`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ext }}" appears directly in run: block of step "Scan for encoding corruption"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ext }}" appears directly in run: block of step "Scan for encoding corruption"; move to env: map

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fix-gbk }}" appears directly in run: block of step "Scan for encoding corruption"; move to env: map

Locations:

- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.root }}" appears directly in run: block of step "Scan for encoding corruption"; move to env: map

Locations:

- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned actions/setup-python@v5 to full commit SHA a26af69be951a213d495a4c3e4e4022e16d87065 with # v5 comment. 2. Moved all ${{ inputs.ext }}, ${{ inputs.fix-gbk }}, ${{ inputs.root }}, and ${{ github.action_path }} expressions out of the run: block into an env: block as INPUT_EXT, INPUT_FIX_GBK, INPUT_ROOT, and ACTION_PATH. 3. Replaced unquoted $FIX_FLAG and $EXT_ARGS string variables with proper bash arrays (FIX_ARGS and EXT_ARGS) expanded with "${FIX_ARGS[@]}" and "${EXT_ARGS[@]}" to prevent word splitting and glob expansion. The comma-separated ext input is split using IFS=',' with read -ra, which is appropriate here since the delimiter is a comma (not whitespace) and the values are individual file extensions without embedded spaces.

