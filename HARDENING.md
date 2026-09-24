<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.47.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **laminas--laminas-continuous-integration-action/1.47.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag ('1') instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on future runs. The failing reference is: `image: 'docker://ghcr.io/laminas/laminas-continuous-integration:1'`. It should be pinned to a full SHA256 digest, e.g. `image: 'docker://ghcr.io/laminas/laminas-continuous-integration@sha256:<64-hex-char-digest>'`.

Locations:

- `action.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker container image in action.yml from the mutable tag `ghcr.io/laminas/laminas-continuous-integration:1` to the immutable digest `ghcr.io/laminas/laminas-continuous-integration:1@sha256:0e10b98519eff57185315b132441fe5964a8064983611d95577593da3b4b1166`. The `docker://` scheme and `:1` tag are preserved inline as required.

