# Security Patch Process

This document defines the end-to-end workflow for handling security findings
in the lite-series. Follow these steps in order for every security fix,
regardless of severity.

---

## 1. Triage

For each finding, decide one of three dispositions before writing any code:

| Disposition | Meaning |
|-------------|---------|
| **Fix** | Implement a code change in this cycle. |
| **Won't fix** | The risk model does not apply to this codebase (document the rationale). |
| **Deferred** | Valid risk, but the fix requires a larger feature; document the mitigation and a trigger condition for when it becomes mandatory. |

Record every disposition — including won't-fix — in
`docs/security/code-review-findings.md` before starting implementation.
See [Recording Findings](#4-record-findings) below.

---

## 2. Assess Scope

Check whether the finding applies to more than one project in the series.
Security issues often share a root cause across projects (e.g. unbounded
`io.ReadAll`, missing permission check). Fix all affected projects in the
same cycle rather than patching them one at a time.

---

## 3. Implement

- One commit per project, scoped to the security fix only.
  Do not bundle unrelated refactors or feature changes.
- Commit message format:

  ```
  security: <short description in imperative mood>

  <optional body — explain the risk and the fix>
  ```

- Verify that all existing tests pass after each fix (`go test ./...`).
- Add tests for the new behaviour where practical (e.g. depth-limit rejection,
  size-limit truncation).

---

## 4. Record Findings

Update `docs/security/code-review-findings.md` (and `docs/ja/security/`) with:

- **Implemented fixes** — what the risk was, what was changed, which constant
  or guard was introduced.
- **Won't-fix items** — include an explicit rationale explaining *why* the risk
  model does not apply. For CLI tools, the standard rationale is:

  > The flag is a CLI argument supplied by the user who owns the process.
  > The user already has OS-level access to the referenced resource.
  > Path/input restrictions are meaningful on server-side code resolving
  > untrusted input — not here.

- **Deferred items** — state the current mitigation and the trigger condition
  that makes the fix mandatory (e.g. "must implement before changing the
  default bind address from loopback to 0.0.0.0").

The findings document is the audit trail. Every item in the security review
must appear in it.

---

## 5. Update CHANGELOG

Add a `### Security` section to the new version entry in each affected
project's `CHANGELOG.md`. Use plain language — describe the risk and the
protection added, not the internal implementation.

Example:

```markdown
## [0.1.1] - 2026-03-27

### Security

- Added MIME recursion depth limit (`maxMIMEDepth = 10`) to prevent stack
  exhaustion from maliciously crafted deeply-nested multipart messages.
- Added per-part memory cap (`maxPartSize = 25 MiB`) to prevent memory
  exhaustion from oversized body parts.
```

---

## 6. Determine Version Numbers

Security-only patches increment the **patch** version (`0.x.Y → 0.x.Y+1`).

If the security fix also includes a breaking API change, increment the
**minor** version instead (`0.x.Y → 0.(x+1).0`).

---

## 7. Release Each Affected Project

Follow the standard [Release Process](../../CONVENTIONS.md#release-process)
for each project. The sequence within a single project is:

1. `make check` — all tests pass.
2. CHANGELOG updated with `### Security` section.
3. Findings document updated (EN + JA).
4. Commit and push to `main`.
5. `git tag vX.Y.Z` and push the tag.
6. `make build-all` (or `make dist` for CGO projects).
7. Create GitHub Release; upload all platform archives as assets.

---

## 8. Update Submodule Pointers

After all affected projects are released, update the `lite-series` umbrella
repository:

```bash
git submodule update --remote <project1> <project2> ...
git add <project1> <project2> ...
git commit -m "chore: update submodule pointers to security release commits"
git push
```

This step is easy to forget — do not skip it.

---

## Checklist (copy for each security cycle)

```
[ ] 1. Triage all findings; record dispositions in code-review-findings.md
[ ] 2. Identify all affected projects
[ ] 3. Implement fixes (one commit per project)
[ ] 4. Update code-review-findings.md (EN + JA) with full rationale
[ ] 5. Update CHANGELOG for each affected project (### Security section)
[ ] 6. Determine patch/minor version bump per project
[ ] 7. Release each project (tag → build → GitHub Release + assets)
[ ] 8. Update lite-series submodule pointers and push
```
