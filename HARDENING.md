<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.43.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.43.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based references instead of pinned SHA digests, making them vulnerable to supply-chain attacks.

action.yml: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'` uses a mutable tag `:1` instead of a SHA digest.

build-and-push-containers.yml: `actions/checkout@v5`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v6`.

create-additional-action-tags.yml: `actions/checkout@v5`.

release-on-milestone-closed.yml: `actions/checkout@v5`, `laminas/automatic-releases@v1` (×5).

shellcheck.yml: `actions/checkout@v5`, `reviewdog/action-shellcheck@v1`.

Locations:

- `action.yml:11`
- `.github/workflows/build-and-push-containers.yml:12`
- `.github/workflows/build-and-push-containers.yml:15`
- `.github/workflows/build-and-push-containers.yml:19`
- `.github/workflows/build-and-push-containers.yml:26`
- `.github/workflows/build-and-push-containers.yml:40`
- `.github/workflows/create-additional-action-tags.yml:22`
- `.github/workflows/release-on-milestone-closed.yml:18`
- `.github/workflows/release-on-milestone-closed.yml:21`
- `.github/workflows/release-on-milestone-closed.yml:28`
- `.github/workflows/release-on-milestone-closed.yml:35`
- `.github/workflows/release-on-milestone-closed.yml:42`
- `.github/workflows/release-on-milestone-closed.yml:49`
- `.github/workflows/shellcheck.yml:13`
- `.github/workflows/shellcheck.yml:16`

### permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and no job within any of them defines a job-level `permissions:` key. This means every workflow runs with the default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build-and-push-containers.yml:1`
- `.github/workflows/create-additional-action-tags.yml:1`
- `.github/workflows/release-on-milestone-closed.yml:1`
- `.github/workflows/shellcheck.yml:1`

### script-injection (severity: high)

create-additional-action-tags.yml contains two `run:` blocks with unquoted shell variable expansions of workflow-controllable data (sub-rule b).

In the `tags` job (lines 14–18): `${TAG}` and `${GITHUB_REF}` are expanded unquoted inside command substitutions: `echo ${TAG}` and `echo ${GITHUB_REF}`. Although `GITHUB_REF` is a GitHub-provided env var, it flows through YAML template substitution and is unquoted.

In the `update-tags` job (lines 29–35): `TAGS` is set from `${{ needs.tags.outputs.tags }}` (a step output, which is workflow-controllable). It is then expanded unquoted in a `for` loop: `for TAG in ${TAGS};do`. The loop variable `${TAG}` and `${ORIGINAL_TAG}` (set from `${{ github.ref }}`) are also used unquoted in `git tag -f ${TAG} ${ORIGINAL_TAG}` and `git push -f origin ${TAG}`, allowing shell metacharacter injection.

Locations:

- `.github/workflows/create-additional-action-tags.yml:16`
- `.github/workflows/create-additional-action-tags.yml:17`
- `.github/workflows/create-additional-action-tags.yml:31`
- `.github/workflows/create-additional-action-tags.yml:33`
- `.github/workflows/create-additional-action-tags.yml:34`

### github-env-injection (severity: high)

In create-additional-action-tags.yml, the `tags` job's `run:` block (line 18) writes values derived from `GITHUB_REF` — specifically `${MAJOR}` and `${MINOR}` — to the step output via the legacy `::set-output` workflow command without first sanitizing them with `printf '%s' ... | tr -d '\n\r'`. A crafted tag ref containing newline characters could inject additional output key-value pairs, which are then consumed by the `update-tags` job via `${{ needs.tags.outputs.tags }}`.

Locations:

- `.github/workflows/create-additional-action-tags.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yml and four workflow files:

1. unpinned-uses: Pinned all action references to full commit SHAs (with tag comments for readability) and pinned the docker container image in action.yml to its sha256 digest while preserving the docker:// scheme and :1 tag.

2. permissions: Added minimal top-level permissions blocks to all four workflow files: build-and-push-containers.yml (contents:read, packages:write), create-additional-action-tags.yml (contents:write), release-on-milestone-closed.yml (contents:write, pull-requests:write), shellcheck.yml (contents:read).

3. script-injection: In create-additional-action-tags.yml, moved github.ref into an env block (GITHUB_REF_VAL), double-quoted all variable expansions, and replaced the unsafe unquoted for loop with a safe while IFS= read -r loop.

4. github-env-injection: Replaced the deprecated ::set-output command with $GITHUB_OUTPUT, and sanitized MAJOR/MINOR values using printf '%s' ... | tr -d '\n\r' before writing to the output file.

