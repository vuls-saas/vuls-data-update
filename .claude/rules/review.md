# Review Guidelines

Enriched from analysis of 120+ merged PRs. See `docs/review/checklist.md` for the full checklist.

## Review Focus Areas

### Deterministic Output
- Changes to `pkg/extract/types/` must maintain deterministic JSON output
- Verify `Sort()` and `Compare()` are updated for new/modified types
- Check that `util.Write` is used instead of ad-hoc marshaling

### Backward Compatibility
- Types under `pkg/extract/types/data` are imported by external repos (`filter-vuls-data-extracted-redhat`, `vuls2`)
- Breaking changes to these types require coordination
- Use `feat!()` commit prefix for breaking changes

### Correctness and Edge Cases
- Regex patterns: check for overly restrictive quantifiers (e.g., `\d{4}` vs `\d{4,}` for IDs that can have 5+ digits)
- Data source coverage: check for missing variants (e.g., SUSE has both `suse.linux.enterprise.micro` and `suse.linux.micro`)
- Enum completeness: when matching advisory prefixes (RLSA-, RBSA-, etc.), verify all valid prefixes are handled

### Error Handling
- Use `github.com/pkg/errors` consistently (not `fmt.Errorf %w`)
- Error messages: lowercase verb phrase in library code, `"failed to ..."` in `pkg/cmd/`
- Validation errors: use `"unexpected X. expected: %q, actual: %q"` pattern
- Non-fatal errors: `slog.Warn(...)` + skip, don't silently ignore

### Code Idioms
- Pre-allocate slices: `make([]T, 0, len(x))` — not `var s []T` when capacity is known
- Avoid `*[]T` (pointer to slice) — pass by value and return
- Use `strings.Contains` over regex for simple checks
- Use `switch` over `if-else` chains for type dispatching
- Fill default directory in options: `dir: filepath.Join(util.CacheDir(), "extract", "<name>")`
- Add `omitempty` on optional JSON fields- Sort `maps.Keys()` / `slices.Collect()` results when output is logged, cached, or compared
- Standardize naming: don't mix suffixes (e.g., `Cache` vs `Map` — pick one)
- Use `internal/` package for code that is shared within a module but should not be importable externally
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
- "Please add test data" is a common review request — don't skip tests

### CI Integration
- New data sources must be added as CI targets in `.github/workflows/extract.yml`
- Don't leave new targets commented out

## Severity Classification

- **CRITICAL**: Security vulnerability, data corruption, breaking API change without coordination
- **HIGH**: Logic error, missing error handling, test coverage gap for critical path, missing CI integration
- **MEDIUM**: Non-idiomatic code, missing edge case handling, overly restrictive regex, missing `omitempty`
- **LOW**: Style nit, naming suggestion, minor optimization
