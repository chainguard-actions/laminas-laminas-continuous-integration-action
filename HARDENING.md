<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/v1.46

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/v1.46** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag (':1') instead of a SHA digest. The image reference 'docker://ghcr.io/laminas/laminas-continuous-integration:1' can silently change to a different (potentially malicious) image at any time. It should be pinned to a specific SHA digest, e.g. 'docker://ghcr.io/laminas/laminas-continuous-integration@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:11`

### missing-permissions (severity: medium)

None of the workflow files define a top-level 'permissions:' key, and no job within them defines job-level permissions. This means all jobs run with the default (potentially broad) GITHUB_TOKEN permissions. Each workflow should declare minimal required permissions explicitly.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

Rule (b) violation in create-additional-action-tags.yml: The 'Update tags' run block uses unquoted shell variable expansions of workflow-controllable values. 'TAGS' is set from '${{ needs.tags.outputs.tags }}' (a step output) and 'ORIGINAL_TAG' from '${{ github.ref }}'. The unquoted expansions 'for TAG in ${TAGS}', 'git tag -f ${TAG} ${ORIGINAL_TAG}', and 'git push -f origin ${TAG}' allow shell metacharacter injection if the tag name contains special characters. Additionally, in the 'Compile tag list' run block, 'echo ${TAG}' and 'echo ${GITHUB_REF}' are unquoted inside command substitutions, also violating rule (b).

Locations:

- `.github/workflows/create-additional-action-tags.yml:14`
- `.github/workflows/create-additional-action-tags.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. action.yml: Pinned Docker image 'docker://ghcr.io/laminas/laminas-continuous-integration:1' to immutable digest sha256:d37db94425e259cd023a405fafc6573a2200f46fe8e30bdd26d1473dfd522213, preserving the tag and docker:// scheme.

2. Added top-level permissions blocks to all four workflow files: build-and-push-containers.yml (contents: read, packages: write), create-additional-action-tags.yml (contents: write), release-on-milestone-closed.yml (contents: write, pull-requests: write), shellcheck.yml (contents: read).

3. Fixed script injection in create-additional-action-tags.yml: Added double quotes around all variable expansions in both the 'Compile tag list' and 'Update tags' steps. Replaced the unsafe 'for TAG in ${TAGS}' (vulnerable to word-splitting and glob expansion) with a safe 'while IFS= read -r TAG; do ... done <<< "${TAGS}"' loop that processes one tag per line.

