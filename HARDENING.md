<!-- markdownlint-disable -->

# Hardening Report: haodehaode378--text-encoding-guard/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **haodehaode378--text-encoding-guard/v1.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated into the run: shell script in the 'Scan for encoding corruption' step. Attacker-controlled inputs are injected directly into shell commands without going through env: variables, allowing arbitrary command injection:
- Line 27: `if [ -n "${{ inputs.ext }}" ]` — inputs.ext interpolated directly
- Line 28: `IFS=',' read -ra EXTS <<< "${{ inputs.ext }}"` — inputs.ext interpolated directly
- Line 33: `if [ "${{ inputs.fix-gbk }}" = "true" ]` — inputs.fix-gbk interpolated directly
- Line 36: `python "${{ github.action_path }}/scripts/check_mojibake.py"` — github.action_path interpolated directly
- Line 37: `--root "${{ inputs.root }}"` — inputs.root interpolated directly
All ${{ }} expressions should be moved to env: variables and then referenced as quoted shell variables (e.g., "$INPUT_ROOT").

Locations:

- `action.yml:27`
- `action.yml:28`
- `action.yml:33`
- `action.yml:36`
- `action.yml:37`

### unpinned-uses (severity: high)

The step `uses: actions/setup-python@v5` references a mutable tag (`v5`) instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a malicious commit. It should be pinned to a specific SHA, e.g., `actions/setup-python@<40-char-sha> # v5`.

Locations:

- `action.yml:21`

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

Fixed two categories of issues in action.yml: (1) Pinned actions/setup-python from mutable tag v5 to full commit SHA a26af69be951a213d495a4c3e4e4022e16d87065. (2) Moved all ${{ }} expressions (inputs.ext, inputs.fix-gbk, inputs.root, github.action_path) out of the run: shell script into an env: block, and updated the shell script to reference the corresponding environment variables ($INPUT_EXT, $INPUT_FIX_GBK, $INPUT_ROOT, $ACTION_PATH) instead.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced the unquoted string-based argument accumulation (`$EXT_ARGS` and `$FIX_FLAG`) with a bash array `EXTRA_ARGS=()`. Extensions from user input are appended as separate array elements via `EXTRA_ARGS+=(--ext "$ext")`, and the fix flag is appended as `EXTRA_ARGS+=(--fix-gbk)`. The array is expanded safely as `"${EXTRA_ARGS[@]}"` in the python command, preventing word splitting and glob expansion on user-controlled values.

