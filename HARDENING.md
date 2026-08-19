<!-- markdownlint-disable -->

# Hardening Report: haodehaode378--text-encoding-guard/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haodehaode378--text-encoding-guard/v1.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Scan for encoding corruption' run: block in action.yml directly interpolates multiple ${{ }} expressions inside shell commands (sub-rule a). This allows an attacker who controls the calling workflow's inputs to inject arbitrary shell commands before the shell ever sees the value:
- Line 27: `if [ -n "${{ inputs.ext }}" ]; then` — inputs.ext interpolated directly
- Line 28: `IFS=',' read -ra EXTS <<< "${{ inputs.ext }}"` — inputs.ext interpolated directly
- Line 34: `if [ "${{ inputs.fix-gbk }}" = "true" ]; then` — inputs.fix-gbk interpolated directly
- Line 37: `python "${{ github.action_path }}/scripts/check_mojibake.py" \` — github.action_path interpolated directly
- Line 38: `--root "${{ inputs.root }}" \` — inputs.root interpolated directly

Additionally (sub-rule b), the shell variables $FIX_FLAG and $EXT_ARGS — which accumulate values derived from inputs.* — are expanded unquoted on lines 39–40 (`$FIX_FLAG` and `$EXT_ARGS`), allowing shell metacharacter injection.

Fix: move all inputs into env: vars, double-quote every expansion, and never use ${{ }} directly inside a run: block.

Locations:

- `action.yml:27`
- `action.yml:28`
- `action.yml:34`
- `action.yml:37`
- `action.yml:38`
- `action.yml:39`
- `action.yml:40`

### unpinned-uses (severity: high)

The composite action step `uses: actions/setup-python@v5` (line 22) references a mutable version tag (`@v5`) rather than an immutable 40-character commit SHA. If the `actions/setup-python` repository is compromised or the tag is moved, the action will silently execute attacker-controlled code. Pin to a full SHA, e.g. `uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5`.

Locations:

- `action.yml:22`

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

Fixed all findings in hardened/action/action.yml:
1. Pinned `actions/setup-python@v5` to full commit SHA `a26af69be951a213d495a4c3e4e4022e16d87065`.
2. Moved all ${{ }} expressions (inputs.ext, inputs.fix-gbk, inputs.root, github.action_path) out of the run: block and into a step-level env: map (INPUT_EXT, INPUT_FIX_GBK, INPUT_ROOT, ACTION_PATH).
3. Replaced unquoted string-concatenated variables ($EXT_ARGS, $FIX_FLAG) with a bash array `args=()` that accumulates --ext and --fix-gbk arguments safely. The array is expanded as "${args[@]}" ensuring every value is properly quoted and argument boundaries are preserved.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

1. Pinned all four action references to full commit SHAs: actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 (used in both test and lint jobs), actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 (used in both test and lint jobs). Original tags preserved as inline comments. 2. Added top-level `permissions: contents: read` block and job-level `permissions: contents: read` blocks to both the test and lint jobs, granting only the minimum permissions needed (read-only repository access for checkout).

