# lite-series Conventions

This document defines the conventions specific to the lite-series.

Development policy, security, authentication, versioning, documentation, and
submodule workflow are defined in the
[nlink-jp organization conventions](https://github.com/nlink-jp/.github/blob/main/CONVENTIONS.md).
The rules there apply to all projects in this series.

Each project also maintains its own `RULES.md` for project-specific rules.
The `RULES.md` files are nearly identical; differences are noted at the end of this document.

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
3. Update any stale documentation (`docs/`, `README.md`, `README.ja.md`).
   Japanese translations must be kept in sync with every English change.
4. Commit and push to `main`.
5. Create a Git tag (`git tag vX.Y.Z`) and push it.
6. Cross-compile release binaries (`make build-all` or `make dist` depending on project).
7. Package binaries: one `.zip` (or `.tar.gz` for CGO projects) per platform.
8. Create a GitHub Release with English release notes and upload the archives as assets.
9. Update the `lite-series` umbrella repository — see
   [Working with Submodules](https://github.com/nlink-jp/.github/blob/main/CONVENTIONS.md#working-with-submodules).

Breaking changes require a **minor version bump** while the project is in the `0.x` series.

Security fixes follow an extended checklist — see
[docs/process/security-patch.md](docs/process/security-patch.md).

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

## RULES.md Divergence

Each project carries its own `RULES.md`. As of 2026-03-27, the files are identical
except for two rules:

| Rule | lite-llm | lite-rag |
|------|----------|----------|
| **22** (Release) | Detailed checklist including binary packaging and GitHub Release steps | Minimal: maintain CHANGELOG and tag releases |
| **23** (Code Review) | Single-contributor; direct commits to `main` are the normal workflow | Requires at least one reviewer before merging to `main` |

When updating shared rules, apply the change to both files.
