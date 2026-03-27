# lite-series

A collection of lightweight CLI tools for working with local and cloud LLMs.
Each tool is small, focused, and designed to compose cleanly in scripts and pipelines.

## Projects

| Project | Description |
|---------|-------------|
| [lite-llm](https://github.com/nlink-jp/lite-llm) | Lightweight CLI for OpenAI-compatible LLM APIs. Supports batch mode, structured output, streaming, and prompt-injection protection. |
| [lite-rag](https://github.com/nlink-jp/lite-rag) | CLI-based RAG tool for Markdown documents. Indexes files into a local DuckDB vector store and answers natural-language questions via a local LLM. |
| [lite-switch](https://github.com/nlink-jp/lite-switch) | Natural language classifier for shell pipelines. Reads free-form text from stdin and outputs the best-matching tag via an OpenAI-compatible LLM. |
| [lite-eml](https://github.com/nlink-jp/lite-eml) | EML parser for shell pipelines. Extracts headers and body from .eml files as structured JSONL, with full charset decoding (ISO-2022-JP, Shift_JIS, etc.). |

## Design Philosophy

- **Small and focused** — each tool does one thing well and composes with others via stdin/stdout.
- **Local-first** — works with local LLMs (LM Studio, Ollama) out of the box; cloud APIs are opt-in.
- **Scriptable** — no interactive UI required; all configuration is file- or environment-based.

## Conventions

All projects in this series follow shared conventions documented in [CONVENTIONS.md](CONVENTIONS.md):
config file format, CLI flag naming, Makefile targets, release process, package structure, and more.
