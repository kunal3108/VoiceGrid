---
name: Bug Report
about: Report a bug in VoiceGrid
title: '[BUG] '
labels: bug
assignees: ''
---

## Description

<!-- A clear, concise description of the bug. What happened that should not have happened? -->

## Reproduction Steps

<!-- Provide exact steps to reproduce the bug. Be specific enough that another engineer can reproduce it. -->

1. 
2. 
3. 
4. 

## Expected Behavior

<!-- What should have happened? -->

## Actual Behavior

<!-- What actually happened? Include error messages, incorrect output, or unexpected state. -->

## Environment

| Field | Value |
|-------|-------|
| Operating System | (e.g., Ubuntu 22.04, macOS 14.0) |
| GPU Model | (e.g., NVIDIA RTX 4090, NVIDIA A100) |
| CUDA Version | (e.g., 12.3 — run `nvcc --version`) |
| Python Version | (e.g., 3.11.5 — run `python --version`) |
| C++ Compiler | (e.g., GCC 12.2, Clang 16.0) |
| Redis Version | (e.g., 7.2.1 — run `redis-server --version`) |
| VoiceGrid Git Commit | (run `git rev-parse HEAD`) |
| VoiceGrid Stage | (e.g., Stage 2 — H02) |

## Logs

<!-- Paste relevant logs here. Use code blocks. Remove any sensitive data. -->

```
(paste logs here)
```

## Additional Context

<!-- Screenshots, profiler output, or any other context that helps diagnose the issue -->

## Severity

<!-- Choose one and explain why -->

- [ ] **Critical** — Causes data loss, security vulnerability, or complete system failure
- [ ] **High** — Causes incorrect results or prevents a component from functioning
- [ ] **Medium** — Component works but behaves incorrectly in a specific case
- [ ] **Low** — Minor issue; workaround exists

**Severity justification:** 

## Attempted Fixes

<!-- If you have tried anything to fix the bug, describe it here -->

## Definition of Fixed

This issue is closed only when:
- [ ] Root cause identified and documented in this issue
- [ ] Fix implemented and committed
- [ ] Tests added to prevent regression
- [ ] Tests pass
- [ ] Fix confirmed on the original reproduction steps
