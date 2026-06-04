# Engram Governance

> Version: 1.0
> Effective: 2026-06-03

---

## Principles

1. **User ownership is non-negotiable.** No version of Engram removes the user's right to export, correct, or delete their memory.
2. **Backwards compatibility.** After v1.0, no breaking changes without a major version bump.
3. **Honest about gaps.** Every version documents what it does not define.
4. **Low implementation barrier.** Every change is evaluated against: does this make Engram harder to implement?
5. **Governed, not controlled.** 8mem initiated the spec. The community governs it.

---

## The Steering Group

The body responsible for approving spec changes.

**Current composition:**

| Role | Organisation |
|------|-------------|
| Chair | 8mem (Ashish Verma) |

**How seats are added:**
- After 3 independent production implementations: one community seat opens
- After 10 implementations: 8mem holds one seat of five — no longer a majority
- After 25 implementations: Engram Foundation is initiated; spec ownership transfers; 8mem retains one seat

This is the planned path to neutrality. Control follows adoption.

---

## RFC Process

All changes to the Engram spec follow this process.

```
Step 1 — IDEA
  Open a GitHub Discussion tagged "RFC Idea"
  Describe: the problem, the proposed change, who it helps

Step 2 — DRAFT RFC
  If ≥5 upvotes or steering group interest:
  Author writes RFC using the template below
  Opens a GitHub Issue titled: "RFC-NNN: [description]"

Step 3 — COMMENT PERIOD
  Minimum 14 days open
  Steering group must comment within 14 days
  Author responds to all substantive comments

Step 4 — STEERING GROUP VOTE
  Accept / Reject / Defer
  Result posted in the Issue

Step 5 — IMPLEMENTATION
  Spec PR merged → engramspec.org updated within 48 hours
  Changelog entry added
  Community notified via GitHub Releases
```

### RFC Template

```markdown
# RFC-NNN: [Title]

**Author:** [Name]
**Status:** Draft
**Created:** YYYY-MM-DD
**Target version:** 0.x / 1.x

## Problem
[What breaks today? Who feels this? Be specific.]

## Proposed Change
[What exactly changes? API endpoint, schema field, behaviour?]

## Example
[Before and after — concrete JSON or curl if applicable.]

## Backwards Compatibility
[Does this break existing implementations? Migration path if yes.]

## Alternatives Considered
[Why is this approach better than the alternatives?]

## Open Questions
[What needs community input?]
```

---

## Versioning

| Version | Meaning |
|---------|---------|
| `0.x` | RFC / draft phase — breaking changes allowed with discussion |
| `1.0` | Stable — triggered when ≥3 independent implementations exist |
| `1.x` | Backwards-compatible additions only |
| `2.0` | Breaking change — migration guide required ≥30 days before |

**What counts as a breaking change:** removing a required field, changing a field type, removing an endpoint, changing error code values, changing the signing algorithm, making an optional field required.

When in doubt, treat as breaking.

---

## Constitutional Constraints

These properties cannot be changed by any RFC or version bump:

1. `GET /v1/context` must always be available — export is unconditional
2. `expires_at` is mandatory — stale memory must not persist silently
3. The `corrections` array is append-only — audit trail is permanent
4. Deleted beliefs appear as tombstones in exports — caches can purge correctly
5. Providers may not paywall `GET /v1/context` — the user's right to read their own memory is free

---

## Conformance Tiers

| Tier | Label | Requirements |
|------|-------|-------------|
| 1 | Reader | `GET /v1/context`, inject `system_prompt_injection`, check `expires_at` |
| 2 | Writer | + `POST /v1/context/correct`, `GET /v1/context/diff`, user confirmation for corrections |
| 3 | Full | + Ed25519 verification, scoped exports, conformance test suite pass |

See [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md) for the full test suite.

---

## The Foundation Path

When ≥10 independent implementations exist:
- Engram Foundation initiated (nonprofit or fiscally sponsored)
- Spec ownership transfers from 8mem to Foundation
- 8mem retains one steering group seat, not the chair
- Chair becomes elected by the community

This is how the spec outlives any single organisation.

---

## Joining the Steering Group

Requirements:
1. Production implementation publicly listed in the registry
2. Active participation in ≥2 RFC discussions
3. Self-nomination via GitHub Issue tagged `steering-group-nomination`
4. Existing steering group approves within 14 days

---

## Contact

- RFC discussions: [GitHub Discussions](https://github.com/engramspec/spec/discussions)
- Steering group: governance@engramspec.org
- Spec: [engramspec.org](https://engramspec.org)

---

*Engram Governance v1.0 — CC BY 4.0*
