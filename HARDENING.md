<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.45.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **laminas--laminas-continuous-integration-action/1.45.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions and a Docker image by mutable tags instead of pinned SHA digests, making the action vulnerable to supply-chain attacks.

- action.yml: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'` (tag `:1`, not a SHA digest)
- build-and-push-containers.yml: `actions/checkout@v7`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`, `docker/metadata-action@v6`, `docker/build-push-action@v7`
- create-additional-action-tags.yml: `actions/checkout@v7`
- release-on-milestone-closed.yml: `actions/checkout@v7`, `laminas/automatic-releases@v1` (×5 steps)
- shellcheck.yml: `actions/checkout@v7`, `reviewdog/action-shellcheck@v1`

Locations:

- `action.yml:11`
- `.github/workflows/build-and-push-containers.yml:14`
- `.github/workflows/build-and-push-containers.yml:17`
- `.github/workflows/build-and-push-containers.yml:23`
- `.github/workflows/build-and-push-containers.yml:31`
- `.github/workflows/build-and-push-containers.yml:43`
- `.github/workflows/create-additional-action-tags.yml:25`
- `.github/workflows/release-on-milestone-closed.yml:17`
- `.github/workflows/release-on-milestone-closed.yml:20`
- `.github/workflows/release-on-milestone-closed.yml:30`
- `.github/workflows/release-on-milestone-closed.yml:40`
- `.github/workflows/release-on-milestone-closed.yml:50`
- `.github/workflows/release-on-milestone-closed.yml:60`
- `.github/workflows/shellcheck.yml:13`
- `.github/workflows/shellcheck.yml:16`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

In create-additional-action-tags.yml, the `run:` block in the 'Update tags' step uses unquoted shell variable expansions of workflow-controllable data (rule b):
- Line 33: `for TAG in ${TAGS};do` — `TAGS` is set from `${{ needs.tags.outputs.tags }}` (a step output) and is expanded unquoted, allowing word-splitting and glob expansion of attacker-influenced content.
- Line 35: `git tag -f ${TAG} ${ORIGINAL_TAG}` — both `${TAG}` (derived from `TAGS`) and `${ORIGINAL_TAG}` (set from `${{ github.ref }}`) are unquoted, enabling shell metacharacter injection.

Locations:

- `.github/workflows/create-additional-action-tags.yml:33`
- `.github/workflows/create-additional-action-tags.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. unpinned-uses: Pinned all action references to full commit SHAs with tag comments: actions/checkout@v7→9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0, docker/setup-buildx-action@v4→bb05f3f5519dd87d3ba754cc423b652a5edd6d2c, docker/login-action@v4→af1e73f918a031802d376d3c8bbc3fe56130a9b0, docker/metadata-action@v6→dc802804100637a589fabce1cb79ff13a1411302, docker/build-push-action@v7→53b7df96c91f9c12dcc8a07bcb9ccacbed38856a, laminas/automatic-releases@v1→935a44be31998a35d0cc57c32f0cc50ccc2b7b54, reviewdog/action-shellcheck@v1→4c07458293ac342d477251099501a718ae5ef86e. Also pinned Docker image in action.yml to sha256:e621d82621f143838af89820573e8521e34c84291594f1ead5634f95acb2893d. 2. missing-permissions: Added top-level permissions blocks to all four workflow files (build-and-push-containers: contents:read+packages:write; create-additional-action-tags: contents:write; release-on-milestone-closed: contents:write+pull-requests:write+issues:write; shellcheck: contents:read). 3. script-injection: In create-additional-action-tags.yml, replaced unquoted `for TAG in ${TAGS}` loop with a properly quoted `while IFS= read -r TAG` loop reading from a here-string, and added double-quotes around all variable expansions ($TAG, $ORIGINAL_TAG) in git commands.

