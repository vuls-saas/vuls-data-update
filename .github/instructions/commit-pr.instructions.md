---
description: "Commit message format and PR workflow conventions"
---
# Commit & PR Conventions

> **Note**: These conventions will be refined after analyzing past commits/PRs (Phase 1). Current content is an initial baseline.

## Commit Messages

### Format

```
<type>(<scope>): <subject>

<body>
```

- **type**: feat, fix, refactor, test, docs, chore, ci
- **scope**: package or area affected (e.g., `extract/alma`, `fetch/nvd`, `types`)
- **subject**: imperative mood, lowercase, no period at end
- **body**: optional, explains *why* not *what*

### Examples

```
feat(extract/alma): add ALSA errata extraction

fix(types/data): stabilize Sort() for nested advisory slices

test(extract/ubuntu): add golden tests for USN fixtures

chore(deps): bump golang.org/x/net to v0.25.0
```

## Pull Requests

### Title
Same format as commit message subject line.

### Description
- **What**: Brief summary of the change
- **Why**: Motivation or issue reference
- **How**: Implementation approach (if not obvious)
- **Testing**: How the change was tested

### Review Workflow
1. Open PR as **draft** against `nightly`
2. Ensure CI passes
3. Address review feedback
4. Update PR description if scope changes during review
