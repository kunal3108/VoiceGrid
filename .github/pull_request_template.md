# Pull Request

## Summary

<!-- Describe what this PR does in 2-3 sentences. What problem does it solve? What component does it add or change? -->

## Type of Change

<!-- Check all that apply -->

- [ ] New feature (new functionality that does not break existing behavior)
- [ ] Bug fix (fixes an incorrect behavior)
- [ ] Performance optimization (improves speed, throughput, or GPU utilization)
- [ ] Refactoring (code change that does not change behavior)
- [ ] Test addition or improvement
- [ ] Documentation update
- [ ] Benchmark experiment
- [ ] Infrastructure or configuration change
- [ ] Stage completion (all acceptance criteria for a stage are met)

## Testing

### What Was Tested

<!-- Describe specifically what was tested. Do not write "tested thoroughly" — describe the actual tests. -->

- 
- 

### Commands to Run Tests

<!-- Exact commands someone else can run to verify your tests pass -->

```bash
# C++ tests (if applicable)


# Python tests (if applicable)


# Integration tests (if applicable)

```

### Test Results

<!-- Paste actual test output. Do not write "all tests pass" without evidence. -->

```
(paste actual test output here)
```

## Benchmark Impact

<!-- Did this change affect performance? If yes, provide data. If no, confirm why not. -->

- [ ] This change improves performance — benchmark results below
- [ ] This change does not affect performance — explanation below
- [ ] This change may affect performance — not yet measured; open issue created

**Benchmark data or explanation:**

## Documentation Updates

<!-- What documentation was updated as part of this PR? -->

- [ ] Hackathon README updated to reflect completed tasks
- [ ] Learning journal(s) updated (topics marked as practiced, confidence updated)
- [ ] DECISIONS.md updated (if new architectural decision was made)
- [ ] ROADMAP.md stage status updated (if stage criteria met)
- [ ] PROJECT_STATUS.md updated
- [ ] SESSION_HANDOFF.md updated
- [ ] docs/benchmarks/ updated (if benchmark was run)
- [ ] N/A — no documentation updates needed (explain why)

## Related Issues

<!-- Reference all related GitHub Issues -->

Closes #
Relates to #

## Checklist

Before requesting review or merging:

- [ ] Code compiles without warnings (`-Wall -Wextra` for C++; `mypy` for Python)
- [ ] All tests pass (run them — do not assume)
- [ ] No secrets, credentials, API keys, or passwords committed
- [ ] No benchmark result invented — all numbers come from actual runs
- [ ] No uncommitted changes in the working directory
- [ ] DECISIONS.md updated if a new technology or architectural choice was made
- [ ] If this closes a stage: all stage acceptance criteria are verified and documented
- [ ] File modification policy from CLAUDE.md / AGENTS.md was followed

## Notes for Reviewer

<!-- Anything specific the reviewer should focus on or be aware of -->
