---
description: "Go code conventions for vuls-data-update: deterministic JSON, util.Write, Sort/Compare patterns, error handling"
---
# Go Code Conventions

## Go Version and Modern Idioms

- Check `go.mod` for the required Go version. Write code using modern idioms available at that version — do not use patterns superseded by it.
- Use `go fix ./...` to apply automated modernizations. Key fixers include:
  - `any` over `interface{}`
  - `min`/`max` builtins over if/else
  - `new(expr)` over `&Type{}`
  - `slices.Contains` over manual loops
  - `slices.Sort` over `sort.Slice` for basic types
  - `strings.CutPrefix`/`strings.Cut` over `HasPrefix`+`TrimPrefix` / `Index`+slice
  - `strings.SplitSeq`/`strings.FieldsSeq` iterators over `Split`/`Fields` when ranging
  - `fmt.Appendf` over `[]byte(fmt.Sprintf(...))`
  - `t.Context()` over `context.WithCancel` in tests
  - `wg.Go` over `wg.Add(1)` / `go` / `wg.Done()`
- When bumping `go` directive in `go.mod`, run `go fix ./...` and commit the result together.

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
  - Use specific sentinel errors (e.g., `ErrNotFoundX`) rather than generic ones when callers need to distinguish error types
- Non-fatal errors: `slog.Warn(...)` + skip (e.g., invalid CPE, unparseable score)
- Bare `return err` when no useful context to add
- Short-circuit early: return `nil, nil` when there is nothing to do, don't let callers trip over validation of empty inputs

## Slice and Map Idioms

- Pre-allocate slices when capacity is known: `make([]T, 0, len(x))`
- For complex capacity: `make([]T, 0, func() int { cap := 0; for ...; return cap }())`
- Avoid `*[]T` (pointer to slice) for mutation — pass slice directly and return
- Use `strings.Contains` over regex for simple substring checks
- Use `switch` over `if-else` chains for type dispatching
- Sort `maps.Keys()` / `slices.Collect()` results when output is logged, cached, or compared — map iteration order is random

## Options Pattern

- New data sources use functional options: `WithDir(dir string) Option`
- Always fill the default directory in the options struct: `dir: filepath.Join(util.CacheDir(), "extract", "<domain>/<name>")`
- Don't leave default paths empty

## JSON Field Tags

- Use `omitempty` on optional string/slice/map/pointer fields; use `omitzero` on struct and `time.Time` fields
- `encoding/json/v2` handles struct tags differently from v1 — no special handling needed for v2-style tags

## CI Integration

- New data sources must be added as CI targets in `.github/workflows/extract.yml`
- Don't leave CI targets commented out

## Cleanup Helpers

- Use `util.RemoveAll(dir)` when cleaning output trees — it preserves `README.md` and `.git` directories
- `util.CacheDir()` defaults to `~/.cache/vuls-data-update` (or temp dir fallback)
