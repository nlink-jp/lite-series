# CLAUDE.md — lite-series

**Organization rules (mandatory): https://github.com/nlink-jp/.github/blob/main/CONVENTIONS.md**

## Non-negotiable rules

- **Tests are mandatory** — write them with the implementation. A feature is not complete without tests.
- **Design for testability** — pure functions, injected dependencies, no untestable globals.
- **Docs in sync** — update `README.md` and `README.ja.md` in the same commit as behaviour changes.
- **Small, typed commits** — `feat:`, `fix:`, `test:`, `chore:`, `docs:`, `refactor:`, `security:`

## This series

Small, local-first CLI tools for LLM interaction, retrieval, classification, and document parsing.

```
lite-series/
├── lite-eml/    github.com/nlink-jp/lite-eml    (Go — .eml parser → JSONL)
├── lite-llm/    github.com/nlink-jp/lite-llm    (Go — OpenAI-compatible LLM CLI)
├── lite-msg/    github.com/nlink-jp/lite-msg    (Go — .msg parser → JSONL)
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

1. Update `CHANGELOG.md` → commit `chore: release vX.Y.Z` → tag → push
2. `gh release create` (no assets)
3. Build 5 platforms: `linux/amd64`, `linux/arm64`, `darwin/amd64`, `darwin/arm64`, `windows/amd64`
4. Zip each binary + `README.md` → upload one by one
5. Update umbrella submodule pointer in this repo
6. Update org profile: `nlink-jp/.github/profile/README.md`
