# AGENTS.md — lite-series

This repository is the root of the **lite-series**: a collection of lightweight Go CLI tools
for LLM workflows, maintained under the [nlink-jp](https://github.com/nlink-jp) organization.

## Repository structure

```
lite-series/
├── CONVENTIONS.md   ← series-wide conventions (read this first)
├── lite-llm/        ← submodule: github.com/nlink-jp/lite-llm
└── lite-rag/        ← submodule: github.com/nlink-jp/lite-rag
```

## Rules

- Series-wide conventions (config format, CLI, Makefile targets, release process, etc.):
  → [CONVENTIONS.md](CONVENTIONS.md)
- Project-level rules (security, testing, documentation policy, etc.):
  → each project's `RULES.md`

## Working with submodules

```sh
# Clone with submodules
git clone --recurse-submodules git@github.com:nlink-jp/lite-series.git

# Update submodule references after a subproject release
git submodule update --remote
git add lite-llm lite-rag
git commit -m "chore: update submodules"
```

## Communication language

Japanese (日本語) — see Rule 12 in each project's `RULES.md`.
