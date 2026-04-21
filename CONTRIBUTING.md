# Contributing to Engram

Engram is an open standard. Contributions, feedback, and proposals are welcome.

---

## How to Propose a Change

**For small changes** (typos, clarifications, examples):
- Open a GitHub Issue describing what should change and why
- Or open a Pull Request directly

**For schema changes** (new fields, modified types, new objects):
- Open a GitHub Issue first — discuss before implementing
- Label it `proposal`
- Include: what you want to change, why it's needed, backwards compatibility impact

**For new endpoints or API changes:**
- Open a GitHub Issue labeled `api-proposal`
- Include: use case, example request/response, which runtimes need this

---

## What Gets Accepted

Changes to the spec must meet these criteria:

1. **Solves a real problem** — not hypothetical. Show a concrete use case.
2. **Backwards compatible** (for 0.x → 1.x) — existing exports must still be valid
3. **Minimal** — add the least possible to solve the problem
4. **Implementable** — any Engram-compatible system should be able to implement it

---

## Versioning

- `0.x` — RFC phase. Breaking changes allowed with discussion.
- `1.0` — Stable. No breaking changes without a major version bump.

All accepted changes are logged in [CHANGELOG.md](CHANGELOG.md).

---

## What This Is Not

- Not a product feature request tracker — Engram is a format spec, not software
- Not a place to propose implementations — build your own and link to it

---

## Code of Conduct

Be direct. Be specific. No fluff.

---

*Engram is initiated by [8mem](https://8mem.com) and open to all.*
