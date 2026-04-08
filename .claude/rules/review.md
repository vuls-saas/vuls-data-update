# Review Guidelines

> **Note**: These guidelines will be enriched after analyzing past PRs (Phase 1). Current content is an initial baseline.

## Review Focus Areas

### Deterministic Output
- Changes to `pkg/extract/types/` must maintain deterministic JSON output
- Verify `Sort()` and `Compare()` are updated for new/modified types
- Check that `util.Write` is used instead of ad-hoc marshaling

### Backward Compatibility
- Types under `pkg/extract/types/data` are imported by external repos (`filter-vuls-data-extracted-redhat`, `vuls2`)
- Breaking changes to these types require coordination

### Golden Test Stability
- If golden diffs appear, verify they are intentional
- Widespread unexpected diffs usually indicate a sorting/determinism issue

### Security
- No hardcoded secrets or credentials
- Validate inputs at system boundaries (CLI args, HTTP responses, file paths)
- Avoid `exec.Command("sh", "-c", userInput)` patterns
- Check for path traversal in file operations

### Test Coverage
- New data sources must include golden tests
- Fetcher tests should use recorded HTTP responses (httptest or fixtures)

## Severity Classification

- **CRITICAL**: Security vulnerability, data corruption, breaking API change without coordination
- **HIGH**: Logic error, missing error handling, test coverage gap for critical path
- **MEDIUM**: Non-idiomatic code, missing edge case handling, incomplete documentation
- **LOW**: Style nit, naming suggestion, minor optimization
