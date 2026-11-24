# CLAUDE.md — lite-series

**Organization rules (mandatory): https://github.com/nlink-jp/.github/blob/main/CONVENTIONS.md**

## Non-negotiable rules

- **Tests are mandatory** — write them with the implementation. A feature is not complete without tests.
- **Design for testability** — pure functions, injected dependencies, no untestable globals.
- **Never `go build` directly** — always use `make build` (outputs to `dist/`). `go build` without `-o dist/...` drops the binary in the project root, polluting the working tree.
- **Docs in sync** — update `README.md` and `README.ja.md` in the same commit as behaviour changes.
- **Small, typed commits** — `feat:`, `fix:`, `test:`, `chore:`, `docs:`, `refactor:`, `security:`

## This series

Small, local-first CLI tools for LLM interaction, retrieval, classification, and document parsing.

```
lite-series/
├── lite-rag/    github.com/nlink-jp/lite-rag    (Go — RAG CLI with DuckDB)
└── lite-switch/ github.com/nlink-jp/lite-switch (Go — LLM-based stdin classifier)
```

## Config file convention

Sectioned TOML at `~/.config/<tool>/config.toml`:

```toml
[default]
base_url = "http://localhost:1234/v1"
api_key  = "dummy"
name     = "default"
```

## Release checklist

The steps are [CONVENTIONS.md §Release Checklist](https://github.com/nlink-jp/.github/blob/main/CONVENTIONS.md#release-checklist) —
open it for each release. This is its outline, not a substitute.

1. Update `CHANGELOG.md` → commit `chore: release vX.Y.Z` (do not push yet)
2. `make vuln` (Go: `govulncheck`; Python: `uv audit --frozen`). A finding
   stops the release before the version is spent: `git reset --soft HEAD~1`,
   fix it in its own commit, commit the release again
3. Tag → push (the build reads its version from the tag)
4. `make clean && make package` — builds every platform, notarizes darwin/arm64
5. `make verify-release`
6. `gh release create` (no assets) — only after step 5 passes
7. Upload each archive one by one (`gh release upload`)
8. `make brew` for tap-eligible tools
9. Update umbrella submodule pointer in this repo
10. Update `nlink-jp/.github/profile/README.md` if new tool
11. Feed new knowledge back to `nlink-jp/knowledge` if any arose
12. Run `check-org.sh` to verify all green
