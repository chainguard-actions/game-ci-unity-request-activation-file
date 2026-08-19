<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-request-activation-file/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-request-activation-file/v2.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of full 40-character commit SHAs. If a tag is moved (e.g. by a compromised upstream), the workflow will silently execute different code.

- .github/workflows/main.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- .github/workflows/versioning.yml: `Actions-R-Us/actions-tagger@v2`

All should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:10`
- `.github/workflows/main.yml:11`
- `.github/workflows/versioning.yml:11`

### script-injection (severity: high)

Rule (a) violation: The `run:` block in the 'Check for failure' step of main.yml directly interpolates `${{ steps.getManualLicenseFile.outcome }}` inside a shell command string. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell command before the shell parses it, allowing metacharacters to be interpreted.

Offending lines:
  `if [ "${{ steps.getManualLicenseFile.outcome }}" != "failure"  ]; then`
  `echo "Expected failure, got ${{ steps.getManualLicenseFile.outcome }}"`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable:
```yaml
env:
  OUTCOME: ${{ steps.getManualLicenseFile.outcome }}
run: |
  if [ "$OUTCOME" != "failure" ]; then
    echo "Expected failure, got $OUTCOME"
    exit 1
  fi
```

Locations:

- `.github/workflows/main.yml:34`

### permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` key, and no individual job within either file defines its own `permissions:` block. Without explicit permissions, workflows inherit the repository's default token permissions, which may be broader than necessary (e.g. `write` access to contents). Each workflow should declare minimal required permissions at the top level or per job.

Affected files:
- .github/workflows/main.yml (jobs: `tests`, `requestManualActivationFile`)
- .github/workflows/versioning.yml (job: `updateMajorTag`)

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all action references to full SHAs:
   - `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4` (both occurrences in main.yml)
   - `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`
   - `Actions-R-Us/actions-tagger@v2` → `@330ddfac760021349fef7ff62b372f2f691c20fb # v2`

2. **script-injection**: In main.yml's 'Check for failure' step, moved `${{ steps.getManualLicenseFile.outcome }}` out of the `run:` block into an `env:` variable (`OUTCOME`) and updated the shell script to reference `$OUTCOME`.

3. **permissions**: Added `permissions: {}` at the top level of both workflow files. For versioning.yml, the `updateMajorTag` job gets a job-level `permissions: contents: write` since the actions-tagger action needs to push/update tags.

