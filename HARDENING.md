<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.44.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.44.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based references instead of pinned full SHA commit hashes or SHA digests. This exposes the action to supply-chain attacks if the referenced tag is moved or the image is replaced.

action.yml: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'` — mutable tag `:1` instead of a SHA digest.

build-and-push-containers.yml: `actions/checkout@v6`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v6`.

release-on-milestone-closed.yml: `actions/checkout@v6`, `laminas/automatic-releases@v1` (used five times).

create-additional-action-tags.yml: `actions/checkout@v6`.

shellcheck.yml: `actions/checkout@v6`, `reviewdog/action-shellcheck@v1`.

Locations:

- `action.yml:10`
- `.github/workflows/build-and-push-containers.yml:13`
- `.github/workflows/build-and-push-containers.yml:16`
- `.github/workflows/build-and-push-containers.yml:21`
- `.github/workflows/build-and-push-containers.yml:28`
- `.github/workflows/build-and-push-containers.yml:35`
- `.github/workflows/release-on-milestone-closed.yml:18`
- `.github/workflows/release-on-milestone-closed.yml:22`
- `.github/workflows/release-on-milestone-closed.yml:30`
- `.github/workflows/release-on-milestone-closed.yml:38`
- `.github/workflows/release-on-milestone-closed.yml:46`
- `.github/workflows/release-on-milestone-closed.yml:54`
- `.github/workflows/create-additional-action-tags.yml:23`
- `.github/workflows/shellcheck.yml:13`
- `.github/workflows/shellcheck.yml:16`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and no individual job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

Rule (b) violation in create-additional-action-tags.yml: the `update-tags` job sets env vars `ORIGINAL_TAG` from `${{ github.ref }}` and `TAGS` from `${{ needs.tags.outputs.tags }}`, then uses both unquoted inside the `run:` shell script. Specifically:
- `for TAG in ${TAGS};do` — `${TAGS}` is unquoted, allowing shell word-splitting and glob expansion on attacker-influenced data.
- `git tag -f ${TAG} ${ORIGINAL_TAG}` — both `${TAG}` and `${ORIGINAL_TAG}` are unquoted, allowing shell metacharacter injection.
These should be double-quoted: `for TAG in "${TAGS}"; do` and `git tag -f "${TAG}" "${ORIGINAL_TAG}"`.

Locations:

- `.github/workflows/create-additional-action-tags.yml:30`
- `.github/workflows/create-additional-action-tags.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned all action references to full commit SHAs with tag comments. Pinned the container image in action.yml to its SHA digest while preserving the docker:// scheme and :1 tag inline.
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - docker/setup-buildx-action@v3 → @8d2750c68a42422c14e847fe6c8ac0403b4cbd6f
   - docker/login-action@v3 → @c94ce9fb468520275223c153574b00df6fe4bcc9
   - docker/metadata-action@v5 → @c299e40c65443455700f0fdfc63efafe5b349051
   - docker/build-push-action@v6 → @10e90e3645eae34f1e60eeb005ba3a3d33f178e8
   - laminas/automatic-releases@v1 → @935a44be31998a35d0cc57c32f0cc50ccc2b7b54 (all 5 occurrences)
   - reviewdog/action-shellcheck@v1 → @4c07458293ac342d477251099501a718ae5ef86e
   - ghcr.io/laminas/laminas-continuous-integration:1 → :1@sha256:e621d82621f143838af89820573e8521e34c84291594f1ead5634f95acb2893d

2. missing-permissions: Added top-level permissions blocks to all four workflow files with minimal required permissions.

3. script-injection: Fixed unquoted variable expansions in create-additional-action-tags.yml — `${TAGS}` and `${TAG}` and `${ORIGINAL_TAG}` are now double-quoted in the for loop and git tag command.

