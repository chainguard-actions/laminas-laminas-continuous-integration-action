<!-- markdownlint-disable -->

# Hardening Report: laminas--laminas-continuous-integration-action/1.43.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **laminas--laminas-continuous-integration-action/1.43.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag instead of a SHA digest. The image `docker://ghcr.io/laminas/laminas-continuous-integration:1` uses the tag `:1`, which can be silently updated to point to a different (potentially malicious) image. It should be pinned to a specific SHA256 digest, e.g. `docker://ghcr.io/laminas/laminas-continuous-integration@sha256:<64-hex-char-digest>`

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image `ghcr.io/laminas/laminas-continuous-integration:1` to its SHA256 digest `sha256:3e1f5b97ccabf424137a8ccee249d6a5cb1fc275da2e2b876be47964b0e9975a` in action.yml line 10. The original tag `:1` is preserved as a comment for readability.

