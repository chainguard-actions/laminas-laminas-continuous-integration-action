<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.42.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.42.0** was hardened automatically. 1 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag (`:1`) instead of a SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The reference `docker://ghcr.io/laminas/laminas-continuous-integration:1` should be pinned to a specific SHA digest, e.g. `docker://ghcr.io/laminas/laminas-continuous-integration@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `docker://ghcr.io/laminas/laminas-continuous-integration:1` to the immutable digest `docker://ghcr.io/laminas/laminas-continuous-integration:1@sha256:e621d82621f143838af89820573e8521e34c84291594f1ead5634f95acb2893d`. The `docker://` scheme and `:1` tag are preserved inline alongside the digest.

### Iteration 2

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all four workflow files:

1. **unpinned-uses**: Pinned all 7 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v4→34e114876b0b11c390a56381ad16ebd13914f8d5, docker/setup-buildx-action@v3→8d2750c68a42422c14e847fe6c8ac0403b4cbd6f, docker/login-action@v3→c94ce9fb468520275223c153574b00df6fe4bcc9, docker/metadata-action@v5→c299e40c65443455700f0fdfc63efafe5b349051, docker/build-push-action@v6→10e90e3645eae34f1e60eeb005ba3a3d33f178e8, laminas/automatic-releases@v1→935a44be31998a35d0cc57c32f0cc50ccc2b7b54, reviewdog/action-shellcheck@v1→4c07458293ac342d477251099501a718ae5ef86e.

2. **permissions**: Added top-level `permissions: {}` to all four workflow files. Added minimal job-level permissions: build-and-push-containers gets `contents: read, packages: write`; update-tags job gets `contents: write`; release job gets `contents: write, pull-requests: write`; shellcheck job gets `contents: read`.

3. **script-injection**: In create-additional-action-tags.yml, quoted all variable expansions in run blocks (`${TAG}` → `"${TAG}"`, etc.), replaced deprecated `::set-output` with `>> "$GITHUB_OUTPUT"`, and properly quoted `${TAG}` and `${ORIGINAL_TAG}` in git commands.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted `${TAGS}` expansion in the `for` loop in `.github/workflows/create-additional-action-tags.yml`. Replaced `for TAG in ${TAGS};do` with `while IFS= read -r TAG; do ... done <<< "${TAGS}"`. The here-string `<<< "${TAGS}"` quotes the expansion, preventing shell metacharacter injection (word-splitting, glob expansion, command separators). The `TAGS` env var was already correctly set via the `env:` block, so no changes were needed there.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Compile tag list' step in .github/workflows/create-additional-action-tags.yml. Added sanitization of MAJOR and MINOR variables (derived from GITHUB_REF) using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. The sanitized values (SAFE_MAJOR and SAFE_MINOR) are now used in the echo statement instead of the raw unsanitized variables, preventing newline injection attacks.

