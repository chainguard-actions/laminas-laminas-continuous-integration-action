<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.49.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.49.0** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action's runs.image points to a mutable tag, not a SHA digest: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'`. If someone re-publishes the `:1` tag, every consumer pulls a different image (supply-chain risk). Pin it to a digest instead, e.g. `docker://ghcr.io/laminas/laminas-continuous-integration@sha256:<digest> # 1`.

Locations:

- `action.yml:12`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://ghcr.io/laminas/laminas-continuous-integration:1@sha256:3f0dd7573f0e6e3a6e4f1c0d03b3e5131167277de87d4c677966d00d03fe6201. The docker:// scheme and :1 tag are still there. args were left as they were.

### Iteration 2

**Notes:**

I left the script-injection finding open on purpose. The action exists to run CI commands taken from the `job` JSON: `command`, `before_script` and `after_script` are run with `bash -c` by design. Removing or escaping that would stop the action from doing anything. The job arg in runs.args is passed as a Docker argv array, so no shell parses it there. Also, action.yml runs the prebuilt image ghcr.io/laminas/laminas-continuous-integration (already pinned by digest), not the entrypoint.sh in this repo, so changing that script would not change what runs. I only added documentation: a security warning in the `job` input description in action.yml (callers must pass only trusted data, such as the matrix action's output, never PR titles or branch names), and a matching comment above the command-execution loop in entrypoint.sh. A re-analysis may report the same finding again, because the behavior it describes is the action's purpose.

### Iteration 3

**Notes:**

No code changes, on purpose. The reported 'script injection' is the action's documented purpose: entrypoint.sh runs the job's before_script/command/after_script through bash -c, and each is a full shell command, so splitting them into argv would break real CI jobs. runs.args (${{ inputs.job }}) must stay as upstream wrote it: Docker args go to the container as an argv array with no shell, so there is no injection at the action.yml level. Also, action.yml uses the prebuilt digest-pinned image ghcr.io/laminas/laminas-continuous-integration:1, so the repo's entrypoint.sh isn't what runs. The previous iteration already added warnings to the job input description and a comment in entrypoint.sh saying job must only come from trusted sources (e.g. laminas-ci-matrix-action), never from PR titles, branch names or issue bodies. The remaining risk is in the calling workflow and should be marked as accepted rather than fixed.

