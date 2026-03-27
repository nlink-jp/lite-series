# Security Code Review Findings

This document records the findings from the lite-series security code review,
the disposition of each finding, and the rationale for any item that was
intentionally not addressed.

---

## Implemented Fixes

### [High] lite-eml — Recursive MIME parsing with no depth limit

**Risk:** A crafted email with deeply nested `multipart/*` parts can exhaust the
call stack and crash the process.

**Fix:** Added `maxMIMEDepth = 10` constant in `internal/parser/body.go`.
`parseBody`, `parseMultipart`, and `processRawPart` now thread a `depth int`
parameter through the recursion. Any part at depth ≥ 10 is silently dropped and
returns an empty result instead of recursing further.

### [Medium] lite-eml — No memory limit on body parts

**Risk:** A crafted email with a very large MIME body part can exhaust process
memory.

**Fix:** Added `maxPartSize = 25 MiB` constant in `internal/parser/body.go`.
`decodeTransfer` now wraps its reader with `io.LimitReader(r, maxPartSize)`
before calling `io.ReadAll`, so no single part can consume more than 25 MiB
of heap regardless of transfer encoding.

### [Medium] lite-switch — Missing config file permission check

**Risk:** A world-readable config file exposes the API key to other users on a
shared system.

**Fix:** `loadSystem` in `internal/config/config.go` now calls `os.Stat` before
`toml.DecodeFile` and passes the result to `checkPermissions`. If the file
mode's group/other bits are non-zero (`perm & 0077 != 0`), a warning is
printed to stderr advising the user to run `chmod 600`.

### [Medium] lite-rag — Missing config file permission check

**Risk:** Same as lite-switch above.

**Fix:** Same pattern applied to `Load` in `internal/config/config.go`.

### [Medium] lite-llm — API key sent over HTTP without warning

**Risk:** When `base_url` begins with `http://` and `api_key` is non-empty, the
key is transmitted in cleartext on the network.

**Fix:** `client.New` in `internal/client/client.go` now checks
`strings.HasPrefix(cfg.Endpoint, "http://") && cfg.APIKey != ""` and prints a
warning to stderr before returning the client. The request is not blocked
because the user may be deliberately using an unencrypted local endpoint (e.g.
LM Studio on localhost).

---

## Intentionally Not Addressed

### [High] lite-llm — Path traversal via `--file`

**Finding:** The `--file` flag accepts an arbitrary filesystem path, which an
attacker could set to `/etc/passwd` or similar.

**Rationale — not fixed:**
The `--file` flag is a CLI argument supplied by the user who is running the
process. In a CLI tool, the user already has full access to the filesystem
under their own account; restricting which paths they may read would serve no
security purpose and would actively harm usability (e.g. reading files from
`/tmp` or outside the working directory is a common scripting pattern).
Path-traversal restrictions are meaningful on *server-side* code that
resolves untrusted user input to a file. They are not appropriate here.

If a future version adds a mode where paths come from untrusted input (e.g. a
batch file fetched from a remote source), path validation should be revisited
at that point.

### [High] lite-rag — Path traversal via `--db` and indexer paths

**Finding:** The `--db` flag and indexer directory arguments accept arbitrary
paths.

**Rationale — not fixed:** Same reasoning as lite-llm `--file` above. The
user controls these arguments at the command line and already has OS-level
access to the referenced paths.

### [High] lite-rag — Web server `/api/ask` has no authentication

**Finding:** The HTTP server started by `lite-rag serve` exposes `/api/ask`
without any authentication, allowing any process on the network to query the
knowledge base.

**Rationale — not fixed at this time:**
The server's default bind address is `127.0.0.1:8080` (loopback only).
Requests from other hosts are rejected at the network layer without any code
change. Adding authentication would require a key-management scheme (generating
a token, distributing it to clients, rotating it), which is a non-trivial
feature rather than a simple security patch.

The current mitigation is documented: users who need to expose the server on a
non-loopback interface should place it behind a reverse proxy with authentication
(e.g. nginx + HTTP basic auth, or Caddy). A future release may add an optional
`--token` flag.

If the default bind address is ever changed to `0.0.0.0`, authentication
becomes mandatory and must be implemented before that change ships.

### [High] lite-switch — Path validation insufficient for `--config` / `--switches`

**Finding:** These flags accept arbitrary paths without canonicalisation or
allow-list checks.

**Rationale — not fixed:** Same reasoning as lite-llm `--file`. These are
user-supplied CLI arguments; restricting them is unhelpful and inappropriate
for a local CLI tool.

---

## Summary Table

| ID | Severity | Project | Finding | Status |
|----|----------|---------|---------|--------|
| 1  | High     | lite-eml | Recursive MIME depth — DoS | Fixed |
| 2  | Medium   | lite-eml | Unbounded part body size | Fixed |
| 3  | Medium   | lite-llm | API key over HTTP | Fixed |
| 4  | Medium   | lite-switch | Config file permission check | Fixed |
| 5  | Medium   | lite-rag | Config file permission check | Fixed |
| 6  | High     | lite-llm | Path traversal via `--file` | Won't fix — CLI tool, user controls paths |
| 7  | High     | lite-rag | Path traversal via `--db` | Won't fix — CLI tool, user controls paths |
| 8  | High     | lite-rag | Unauthenticated web server | Deferred — localhost-only default; future `--token` flag |
| 9  | High     | lite-switch | Insufficient path validation | Won't fix — CLI tool, user controls paths |
