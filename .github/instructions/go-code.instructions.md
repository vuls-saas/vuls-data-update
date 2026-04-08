---
description: "Go code conventions for vuls-data-update: deterministic JSON, util.Write, Sort/Compare patterns, error handling"
---
# Go Code Conventions

## Deterministic JSON Output

- Use `encoding/json/v2` with `json.Deterministic(true)` and tab indent
- Always use `util.Write(path, content, doSort)` — never ad-hoc `json.Marshal`
- `util.Write` calls `Sort()` for known types, which recursively normalizes all nested slices
- **No need to pre-sort map keys** when building data structures — `Sort()` handles it

## Types and Schema

- Extracted JSON schema lives under `pkg/extract/types/**`
- When modifying types:
  - Keep output stable/deterministic
  - Update `Sort()` and `Compare()` where applicable
  - Maintain backward compatibility (used by `filter-vuls-data-extracted-redhat` and `vuls2`)

## Adding a New Data Source

1. Implement the fetcher in `pkg/fetch/<domain>/<name>/`
2. Implement the extractor in `pkg/extract/<domain>/<name>/`
3. Wire `newCmd…()` in both `pkg/cmd/fetch/fetch.go` and `pkg/cmd/extract/extract.go`
4. Follow the existing pattern shown by `newCmdAlmaErrata()` (default output under `util.CacheDir()`, `--dir/-d` flag)

## Error Handling

- Use `github.com/pkg/errors` for wrapping — **not** `fmt.Errorf("%w")` or `xerrors`
- `errors.Wrap(err, "msg")` / `errors.Wrapf(err, "fmt", args)` for wrapping with context
- `errors.Errorf("fmt", args)` for new errors (validation failures etc.)
- Message conventions:
  - Library/inner code: lowercase verb phrase — `"decode json"`, `"open %s"`, `"read %s"`
  - `pkg/cmd/` (Cobra layer): `"failed to ..."` prefix — `"failed to extract almalinux errata"`
  - Validation: `"unexpected X. expected: %q, actual: %q"` pattern
- Sentinel errors (`errors.New`) are rare; check with `errors.Is()`
- Non-fatal errors: `slog.Warn(...)` + skip (e.g., invalid CPE, unparseable score)
- Bare `return err` when no useful context to add

## Cleanup Helpers

- Use `util.RemoveAll(dir)` when cleaning output trees — it preserves `README.md` and `.git` directories
- `util.CacheDir()` defaults to `~/.cache/vuls-data-update` (or temp dir fallback)
