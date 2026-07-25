<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.46.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.46.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a mutable Docker image tag instead of a SHA digest. The reference 'docker://ghcr.io/laminas/laminas-continuous-integration:1' uses the tag ':1', which can be silently replaced with a different (potentially malicious) image. It should be pinned to a SHA256 digest, e.g. 'ghcr.io/laminas/laminas-continuous-integration@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:10`

### permissions (severity: medium)

missing-permissions: None of the workflow files define a top-level 'permissions:' key, and no job within them defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Each workflow should declare minimal required permissions.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

Rule (b) violation — unquoted shell variable expansions of workflow-controllable data in the 'update-tags' job's run block. The env vars TAGS (sourced from '${{ needs.tags.outputs.tags }}') and ORIGINAL_TAG (sourced from '${{ github.ref }}') are expanded unquoted: 'for TAG in ${TAGS};do', 'git tag -f ${TAG} ${ORIGINAL_TAG}', and 'git push -f origin ${TAG}'. An attacker who can influence the tag name or step outputs could inject shell metacharacters to execute arbitrary commands.

Locations:

- `.github/workflows/create-additional-action-tags.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

1. action.yml: Pinned Docker image 'docker://ghcr.io/laminas/laminas-continuous-integration:1' to immutable digest 'docker://ghcr.io/laminas/laminas-continuous-integration:1@sha256:d37db94425e259cd023a405fafc6573a2200f46fe8e30bdd26d1473dfd522213', preserving the docker:// scheme and :1 tag inline.
2. Added top-level 'permissions:' blocks to all four workflow files: shellcheck.yml (contents:read), build-and-push-containers.yml (contents:read, packages:write), create-additional-action-tags.yml (contents:write), release-on-milestone-closed.yml (contents:write, pull-requests:write).
3. Fixed script injection in create-additional-action-tags.yml update-tags job: replaced unquoted 'for TAG in ${TAGS}' with a safe 'while IFS= read -r TAG; do ... done <<< "${TAGS}"' loop, and added double-quotes around all variable expansions ("${TAG}", "${ORIGINAL_TAG}").

