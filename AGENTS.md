# AGENTS.md — lite-series

## Project summary

Umbrella repository for nlink-jp's local-first LLM interaction and pipeline
tools. Each tool lives in its own repository, included here as a submodule.
The catalog — one row per tool — is [README.md](README.md); this file covers
only how to work with the umbrella (ADR-005). Series-specific rules live in
[CONVENTIONS.md](CONVENTIONS.md), series-level documents under `docs/`.

## Key commands

| Command | Purpose |
|---------|---------|
| `git clone --recurse-submodules https://github.com/nlink-jp/lite-series.git` | Clone with all tools |
| `git submodule update --init` | Populate submodules in an existing clone |
| `git submodule update --remote <tool>` | Pull a tool's latest main |
| `git add <tool>` → commit `chore: bump <tool> to vX.Y.Z` | Update the pointer after a tool release |

## Gotchas

- Tool development happens in the tool repositories; new projects start in
  the workspace root `_wip/`, never directly inside this umbrella
  (CONVENTIONS.md — Starting a New Project).
- Submodule checkouts default to detached HEAD — `git checkout main` inside
  a submodule before committing.
- Submodule URLs are HTTPS only (SSH fails on machines without key auth).
- `lite-llm` is archived (kept as a submodule for reference); `lite-eml`
  and `lite-msg` were renamed and moved to util-series as `eml-to-jsonl`
  and `msg-to-jsonl` — don't resurrect their rows here.
- Every submodule needs a catalog row in README.md — `check-org.sh` fails
  otherwise.

## Module path

Repository: `github.com/nlink-jp/lite-series`
