# lite-series Conventions

This document defines the conventions shared across all projects in the lite-series.
New projects must follow these conventions to remain consistent with the series.

Each project also maintains its own `RULES.md` for project-specific rules.
The `RULES.md` files are nearly identical; differences are noted at the end of this document.

---

## Development Principles

These principles apply to all work across the lite-series. They take precedence over
convenience or speed.

### Security first

Treat security as a first-class concern at every stage of design, implementation,
and review.

- Never embed secrets, credentials, or sensitive data in source code.
- Config files that may contain API keys must warn on insecure permissions (see
  [Configuration File — Permission warning](#permission-warning)).
- Keep dependencies minimal; document the rationale for each third-party library.
- Run `govulncheck` as part of the quality gate (`make check`).

### Build small, fix small

- Build the smallest unit that satisfies the requirement, then iterate.
- A fix must be scoped to the reported problem — do not refactor unrelated code in
  the same change.
- Do not add features, helpers, or abstractions for hypothetical future requirements.
- Prefer composition over monolithic structures.

### Design and implement for testability

- Separate concerns so that each package can be tested in isolation.
- Inject dependencies (I/O, clocks, external clients) rather than hard-coding them,
  so tests can substitute fakes or in-process servers without external infrastructure.
- Avoid global mutable state; pass configuration and dependencies explicitly.

### Tests alongside implementation

- Write tests in the same commit or PR as the implementation — never defer them.
- Do not merge untested production code.
- Tests must pass before marking a task complete and before committing.

### Documentation alongside implementation

- Update `README`, `docs/`, and `CHANGELOG` in the same commit or PR as the
  implementation.
- Japanese translations (`README.ja.md`, `docs/ja/`) must be kept in sync with
  every change to English documentation.
- Stale documentation is a bug.

---

## Configuration File

### Format

All projects use **TOML with named sections**. Flat top-level keys are not used.

```toml
[api]
base_url = "https://api.openai.com"
api_key  = "sk-..."

[model]
name = "gpt-4o-mini"
```

Section names follow the concern they represent (`[api]`, `[model]`, `[database]`,
`[server]`, `[retrieval]`, etc.).

### Default path

```
~/.config/<project-name>/config.toml
```

Respect `XDG_CONFIG_HOME` when set.

### Permission warning

If the config file is readable by group or others (`perm & 0077 != 0`), warn on stderr:

```
Warning: config file <path> has permissions <octal>; expected 0600.
  The file may contain an API key. Run: chmod 600 <path>
```

### Environment variable overrides

Every config value must be overridable by an environment variable.

Naming convention: `LITE_<PROJECT>_<KEY>` in `SCREAMING_SNAKE_CASE`.

Examples:
- `LITE_LLM_API_KEY`, `LITE_LLM_BASE_URL`, `LITE_LLM_MODEL`
- `LITE_RAG_API_KEY`, `LITE_RAG_API_BASE_URL`, `LITE_RAG_CHAT_MODEL`

### Priority order (highest first)

```
CLI flags  >  environment variables  >  config file  >  compiled-in defaults
```

### Example file

Provide a `config.example.toml` at the project root. Header format:

```toml
# <project-name> example configuration
# Copy this file to ~/.config/<project-name>/config.toml and fill in your values.
# All settings can also be overridden via environment variables (prefix: LITE_<PROJECT>_).
```

Commented-out keys show optional settings; uncommented keys show required or
strongly-recommended defaults.

---

## CLI Conventions

- Use [Cobra](https://github.com/spf13/cobra) for all CLI projects.
- Short flags (`-p`, `-f`, `-m`, `-q`, `-c`) paired with long flags (`--prompt`, `--file`, etc.).
- Multi-word flags use hyphens: `--system-prompt`, `--json-schema`, `--no-safe-input`.
- Negation prefix: `--no-<feature>` to disable a default-on feature.
- Always provide `--config` / `-c` to override the default config path.
- Declare mutually exclusive flag groups explicitly with `MarkFlagsMutuallyExclusive`.

---

## Makefile Targets

All projects must define the following targets with these exact names and semantics:

| Target | Description |
|--------|-------------|
| `build` | Compile the binary for the current platform |
| `test` | Run all tests (`go test ./...`) |
| `vet` | Run `go vet ./...` |
| `lint` | Run `golangci-lint run ./...` |
| `check` | Full quality gate: vet → lint → test → build → govulncheck |
| `setup` | Install Git hooks from `scripts/hooks/` |
| `clean` | Remove build artifacts |

Additional cross-compilation targets may differ per project (see below).

### Version injection

```makefile
VERSION ?= $(shell git describe --tags --always --dirty 2>/dev/null || echo "dev")
LDFLAGS := -ldflags "-X main.version=$(VERSION)"
```

### Cross-compilation targets

Minimum set of platforms for release binaries:

| GOOS | GOARCH |
|------|--------|
| linux | amd64 |
| linux | arm64 |
| darwin | amd64 |
| darwin | arm64 |
| windows | amd64 |

Note: projects that depend on CGO (e.g. DuckDB) require platform-specific toolchains
and may not support all platforms.

---

## Release Process

1. All tests pass (`make check`).
2. Update `CHANGELOG.md` with the new version and date.
3. Commit and push to `main`.
4. Create a Git tag (`git tag vX.Y.Z`) and push it.
5. Cross-compile release binaries.
6. Package binaries: `.zip` per platform.
7. Create a GitHub Release with English release notes and upload the zips as assets.

Breaking changes require a **minor version bump** while the project is in the `0.x` series.

---

## Package Structure

```
<project>/
├── cmd/              # CLI entry point (Cobra commands)
├── internal/         # Application-private code
│   ├── config/       # Config struct, Load(), defaults, env overrides
│   └── ...           # One package per concern
├── pkg/              # Public library code (only if intended for external use)
├── scripts/
│   └── hooks/        # pre-commit, pre-push
├── docs/
│   ├── design/       # Architecture and design decisions
│   ├── guide/        # User-facing guides
│   ├── ja/           # Japanese translations (mirror of docs/)
│   ├── dependencies.md
│   ├── setup.md
│   └── structure.md
├── CHANGELOG.md
├── RULES.md
├── Makefile
├── config.example.toml
├── README.md
└── README.ja.md
```

---

## Documentation Conventions

- All source code comments and primary documentation in **English**.
- A Japanese translation (`docs/ja/`, `README.ja.md`) must be maintained in parallel
  and kept in sync with every change.
- README sections (in order): description → features → installation → configuration →
  usage → building → documentation links.
- `docs/dependencies.md` — document every third-party dependency: purpose, why not
  implemented in-house, license.

---

## CHANGELOG and Versioning

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) +
[Semantic Versioning](https://semver.org/).

Section categories: `Added`, `Changed`, `Fixed`, `Removed`, `Docs`, `Internal`.
Prefix breaking changes with **Breaking:**.

Header template:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).
```

---

## RULES.md Divergence

Each project carries its own `RULES.md`. As of 2026-03-27, the files are identical
except for two rules:

| Rule | lite-llm | lite-rag |
|------|----------|----------|
| **22** (Release) | Detailed checklist including binary packaging and GitHub Release steps | Minimal: maintain CHANGELOG and tag releases |
| **23** (Code Review) | Single-contributor; direct commits to `main` are the normal workflow | Requires at least one reviewer before merging to `main` |

When updating shared rules, apply the change to both files.
