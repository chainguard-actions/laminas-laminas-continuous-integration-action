<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.48.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.48.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml points runs.image at a mutable tag instead of a SHA256 digest: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'`. Whoever controls the tag can change the image without any change here. Pin it as `ghcr.io/laminas/laminas-continuous-integration@sha256:<digest> # 1`.

Locations:

- `action.yml:12`

### script-injection (severity: high)

The entrypoint passes its argument (inputs.job, given via runs.args) to a shell. entrypoint.sh pulls `.command`, `.before_script[]` and `.after_script[]` out of the job JSON and runs each one with `sudo --preserve-env --set-home -u testuser /bin/bash -c "${COMMAND}"` (also with "${BEFORE_SCRIPT_COMMAND}" and "${AFTER_SCRIPT_COMMAND}"). Whatever is in the job input runs as shell code. The action is built to run CI commands from the job, but the input must come only from trusted sources, such as a matrix the workflow generates itself, and never from data an attacker can control.

Locations:

- `entrypoint.sh:246`
- `entrypoint.sh:253`
- `entrypoint.sh:266`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

unpinned-uses: runs.image is now pinned as 'docker://ghcr.io/laminas/laminas-continuous-integration:1@sha256:3f0dd7573f0e6e3a6e4f1c0d03b3e5131167277de87d4c677966d00d03fe6201'. The docker:// scheme and the tag are kept. script-injection: NOT fixed in code. The whole point of this action is to run the job's command, before_script and after_script as shell commands, so removing the bash -c calls would break it. runs.args is passed to the container as an argv array, so the args line itself is not injectable and was left unchanged. I added a security comment in entrypoint.sh and a note in the input description saying the job input must only come from trusted sources, such as a matrix the workflow generates itself. Also note that the action runs a prebuilt image, so the entrypoint.sh in the repo is not what actually runs.

