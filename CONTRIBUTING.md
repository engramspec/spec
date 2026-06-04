# Contributing to Engram

Engram is an open standard. The spec, governance model, and conformance tests are all open for contribution.

---

## What You Can Contribute

| Contribution type | Where |
|-------------------|-------|
| Spec change proposal (RFC) | GitHub Issue → RFC process |
| Typo / clarification | Pull Request directly |
| New implementation | GitHub Issue with label `implementation` |
| Conformance test addition | PR to `spec/CONFORMANCE.md` |
| Governance feedback | GitHub Discussion |

---

## The RFC Process

All spec changes (schema fields, API endpoints, security model, versioning rules) go through the RFC process defined in [`GOVERNANCE.md`](GOVERNANCE.md).

**Short version:**

1. Open a GitHub Discussion describing the problem and proposed change
2. If it gets traction, open a GitHub Issue titled `RFC-NNN: [description]`
3. Minimum 14-day comment period — open to everyone
4. Steering group votes: Accept / Reject / Defer
5. Accepted RFCs become spec PRs

**RFC template is in `GOVERNANCE.md`.**

---

## What Makes a Good RFC

- Starts with a concrete problem, not a proposed solution
- Shows a specific use case where the current spec fails
- Addresses backwards compatibility explicitly
- Proposes the minimum change that solves the problem
- Lists alternatives considered and why they were rejected

---

## Small Changes

For typos, broken examples, clarifications, and documentation improvements:

- Open a Pull Request directly
- No RFC needed
- Include: what you changed and why in the PR description

---

## Registering an Implementation

If you have built an Engram-compatible runtime or provider, open a GitHub Issue with label `implementation` and include:

- Project name and URL
- Which conformance tier you implement (Reader / Writer / Full)
- A link to your integration code or documentation
- Whether you are open to being listed in the public registry

---

## Acceptance Criteria for Spec Changes

All accepted changes must:

1. **Solve a documented real-world problem** — not hypothetical
2. **Be backwards compatible** for minor versions (existing exports remain valid)
3. **Be implementable** by any runtime or provider without deep platform coupling
4. **Respect the constitutional constraints** in `GOVERNANCE.md` — these cannot change

---

## Versioning

- `0.x` — RFC phase. Breaking changes allowed with steering group approval and discussion.
- `1.0` — Stable. No breaking changes without a major version bump.

All changes logged in [`CHANGELOG.md`](CHANGELOG.md).

---

## Code of Conduct

Be specific. Be direct. Disagreement on technical merit is welcome. Personal attacks are not.

---

*Engram is initiated by [8mem](https://8mem.com) and governed by the Engram Steering Group.*
*Open to all — see [`GOVERNANCE.md`](GOVERNANCE.md).*
