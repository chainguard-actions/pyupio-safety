<!-- markdownlint-disable -->

# Hardening Report: pyupio--safety/3.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyupio--safety/3.7.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image with a mutable tag (`docker://pyupio/safety-v2-beta:latest`) instead of a SHA digest. This means the image can be silently replaced with a different version, enabling supply-chain attacks. The `image:` field should reference a specific SHA digest, e.g. `docker://pyupio/safety-v2-beta@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from `docker://pyupio/safety-v2-beta:latest` to `docker://pyupio/safety-v2-beta:latest@sha256:a911aa6afee02c3cc4ff6a6a51c7bada392df8b775d32d22c2bdb3ef70040ec7`. The `docker://` scheme and `:latest` tag are preserved inline, with the immutable digest appended to prevent supply-chain attacks via mutable tags.

