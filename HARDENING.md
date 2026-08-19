<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.40.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.40.1** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple unpinned action/image references found. In action.yml, the Docker image uses a mutable tag ':1' instead of a SHA digest: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'`. In workflow files, all `uses:` references use mutable version tags instead of pinned 40-character commit SHAs: `actions/checkout@v4`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v6`, `laminas/automatic-releases@v1`, `reviewdog/action-shellcheck@v1`.

Locations:

- `action.yml:11`
- `.github/workflows/build-and-push-containers.yml:13`
- `.github/workflows/build-and-push-containers.yml:16`
- `.github/workflows/build-and-push-containers.yml:21`
- `.github/workflows/build-and-push-containers.yml:30`
- `.github/workflows/build-and-push-containers.yml:38`
- `.github/workflows/create-additional-action-tags.yml:22`
- `.github/workflows/release-on-milestone-closed.yml:19`
- `.github/workflows/release-on-milestone-closed.yml:23`
- `.github/workflows/release-on-milestone-closed.yml:33`
- `.github/workflows/release-on-milestone-closed.yml:42`
- `.github/workflows/release-on-milestone-closed.yml:51`
- `.github/workflows/release-on-milestone-closed.yml:60`
- `.github/workflows/shellcheck.yml:14`
- `.github/workflows/shellcheck.yml:17`

### permissions (severity: medium)

None of the four workflow files define a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows run with default (potentially broad) token permissions. All jobs in all four workflow files lack any permissions restriction.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

Rule (b) violation — unquoted shell variable expansions of workflow-controllable data in run blocks. In `create-additional-action-tags.yml`: (1) The `Compile tag list` step uses `echo ${TAG}` and `echo ${GITHUB_REF}` unquoted inside command substitutions where TAG derives from GITHUB_REF. (2) The `Update tags` step sets `TAGS` from `${{ needs.tags.outputs.tags }}` (a step output, which is workflow-controllable) and then iterates `for TAG in ${TAGS}` (unquoted), and passes `${TAG}` and `${ORIGINAL_TAG}` unquoted to `git tag -f ${TAG} ${ORIGINAL_TAG}` and `git push -f origin ${TAG}`. Shell metacharacters in these values would be interpreted by the shell.

Locations:

- `.github/workflows/create-additional-action-tags.yml:15`
- `.github/workflows/create-additional-action-tags.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned all action references to full 40-character commit SHAs (with tag comments for readability) and pinned the Docker image in action.yml to its sha256 digest while preserving the docker:// scheme and :1 tag.

2. **permissions**: Added minimal `permissions:` blocks to all four workflow files: build-and-push-containers.yml (contents:read, packages:write), create-additional-action-tags.yml (contents:write), release-on-milestone-closed.yml (contents:write, pull-requests:write), shellcheck.yml (contents:read).

3. **script-injection**: Fixed create-additional-action-tags.yml by: (a) quoting all variable expansions in the Compile tag list step (${TAG} → "${TAG}", ${GITHUB_REF} → "${TAG}" via parameter expansion), (b) replacing deprecated ::set-output with >> "$GITHUB_OUTPUT", (c) quoting ${TAG} and ${ORIGINAL_TAG} in git tag and git push commands in the Update tags step, (d) using bash parameter expansion for stripping refs/tags/ prefix instead of shell substitution patterns.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted `${TAGS}` expansion in the 'Update tags' step of `.github/workflows/create-additional-action-tags.yml`. Replaced `for TAG in ${TAGS}` (which allowed shell word-splitting and glob expansion) with `while IFS= read -r TAG; do ... done <<< "${TAGS}"`. This safely splits the newline-separated tag list without glob expansion, and skips empty lines with a guard clause.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Compile tag list' step in .github/workflows/create-additional-action-tags.yml. All values derived from GITHUB_REF (TAG, MAJOR, MINOR, and the final concatenated tags string) are now sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline injection attacks that could poison downstream step outputs.

