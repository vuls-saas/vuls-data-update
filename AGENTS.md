# Project Guidelines — vuls-data-update

## What This Repo Does

CLI tool to **fetch** raw vulnerability data sources, **extract** them to canonical JSON datasets, and manage **dotgit** repositories for distribution.

## Build & Test

```sh
go build ./cmd/vuls-data-update
go test ./...
```

## Architecture

- **Cobra command tree**: Root in `pkg/cmd/root/root.go` → `fetch`, `extract`, `dotgit`
- **Data-source subcommands**: Registered in switchboards at `pkg/cmd/fetch/fetch.go` and `pkg/cmd/extract/extract.go`
- **Adding a new data source**:
  1. Implement fetcher in `pkg/fetch/<domain>/<name>/`
  2. Implement extractor in `pkg/extract/<domain>/<name>/`
  3. Wire `newCmd…()` in both `pkg/cmd/fetch/fetch.go` and `pkg/cmd/extract/extract.go`
  4. Follow the pattern of `newCmdAlmaErrata()`

## Key Conventions

- **Deterministic JSON**: Use `encoding/json/v2` with `json.Deterministic(true)` and tab indent via `util.Write()`
- **Sort/Compare**: `Sort()` recursively normalizes nested slices — no need to pre-sort map keys
- **Golden tests**: Fixtures in `testdata/fixtures/`, golden output in `testdata/golden/`. Use `util/test` helpers
- **Cleanup**: Use `util.RemoveAll(dir)` — preserves `README.md` and `.git`
- **Cache**: `util.CacheDir()` defaults to `~/.cache/vuls-data-update`

→ Detailed guidelines: see `.github/instructions/`
