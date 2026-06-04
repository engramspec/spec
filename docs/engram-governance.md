# Engram Governance Model
## How the Engram Memory Protocol Is Governed, Changed, and Owned

> Version: 1.0
> Status: Active
> Effective: 2026-06-03
> Maintained by: Engram Steering Group

---

## Why Governance Matters

A spec without governance is a spec one company controls forever.
A spec with governance is a standard anyone can trust.

The moment Engram has a governance model, three things change:
1. Runtime developers can implement Engram without worrying that 8mem will break it to suit their commercial interests
2. Enterprises can rely on it — procurement and legal require open governance for any protocol dependency
3. The standard can outlive 8mem — if 8mem pivots, disappears, or is acquired, Engram continues

OAuth 2.0 has the IETF. OpenID has the OpenID Foundation. Engram needs its own neutral governance structure. This document defines it.

---

## Guiding Principles

**1. User ownership is non-negotiable.**
No version of Engram will remove the user's right to export, correct, or delete their memory. This is a constitutional constraint — not subject to majority vote.

**2. Backwards compatibility across minor versions.**
0.x → 1.0 may have one migration. After 1.0, major version bumps are breaking changes. Implementers are never surprised.

**3. Honest about limitations.**
Every version documents what it does not define. A spec that pretends to be complete is more dangerous than one that names its gaps.

**4. Low barrier to implementation.**
Every change is evaluated against: "does this make it harder to implement?" If yes, the burden of proof is on the proposer.

**5. Governed, not controlled.**
8mem maintains the reference implementation. It does not control the spec. The steering group governs. Any adopter can join.

---

## Governance Structure

### The Steering Group

The body responsible for approving changes to the Engram spec.

**Initial composition (at launch):**

| Role | Holder | Organisation |
|------|--------|-------------|
| Chair | Ashish | 8mem (Founder) |
| Technical Lead | TBD — invited post-launch | First independent adopter |
| Community Seat | TBD — elected after 3 implementations | Community representative |

**How it grows:**
- After 3 independent implementations: one elected community seat added
- After 10 implementations: independent technical lead replaces 8mem technical lead
- After 25 implementations: 8mem holds one seat out of five — no longer a majority

This is the planned path to neutrality. 8mem starts in control because it wrote the spec. 8mem loses control as the community grows. This is by design.

**Quorum:** Majority of seats must vote for a change to pass.

**Chair veto:** The Chair may veto any change that violates the five guiding principles. The veto may be overridden by unanimous other seats.

---

### The RFC Process

All changes to the Engram spec follow this process:

```
Step 1 — IDEA
  Anyone opens a GitHub Discussion (not an Issue yet)
  Tag: "RFC Idea"
  Describe: problem, proposed change, who it helps

Step 2 — DRAFT RFC
  If idea gets ≥5 upvotes or steering group signals interest:
  Author writes a formal RFC document (template below)
  Opens a GitHub Issue with label "RFC"
  Issue title: "RFC-XXX: [Short description]"

Step 3 — COMMENT PERIOD
  Minimum 14 days open for community comment
  Steering group members must comment within 14 days
  Author responds to all substantive comments

Step 4 — STEERING GROUP VOTE
  After comment period closes:
  Steering group votes: Accept / Reject / Defer
  Result posted in the Issue

Step 5 — IMPLEMENTATION (if accepted)
  For spec changes: PR to spec document
  For API changes: reference implementation updates in 8mem
  For SDK changes: SDK update in engram-sdk-python / engram-sdk-node

Step 6 — PUBLISH
  Merged to main → engramspec.org updated within 48 hours
  Changelog entry added
  Community notified via GitHub Releases
```

---

### RFC Template

```markdown
# RFC-XXX: [Title]

**Author:** [Name, GitHub handle]
**Status:** Draft
**Created:** YYYY-MM-DD
**Target version:** 0.x / 1.x / 2.0

## Problem

[What problem does this solve? Who feels this pain? Be specific.]

## Proposed Change

[What exactly changes? API endpoint? Schema field? Behaviour? Be precise.]

## Example

[Show before and after. Concrete JSON or code if applicable.]

## Backwards Compatibility

[Does this break existing implementations? If yes: migration path?]

## Alternatives Considered

[What else could solve this? Why is this approach better?]

## Open Questions

[What is not yet decided? What do you need community input on?]
```

---

## Versioning Rules

### Semantic Versioning for Specs

```
0.x   —  RFC / draft phase. Breaking changes allowed between 0.x releases.
          Implementers must check engram_version on every response.

1.0   —  Stable. Triggered when ≥3 independent implementations exist.
          After 1.0: no breaking changes without major version bump.

1.x   —  Backwards-compatible additions only.
          New optional fields. New optional endpoints.
          Old 0.1 exports remain parseable.

2.0   —  Breaking change. Requires migration guide published ≥30 days before.
          8mem publishes v1 → v2 migration path in reference implementation.
```

### What Counts as a Breaking Change

| Change | Breaking? |
|--------|-----------|
| Adding an optional field | No |
| Adding a new endpoint | No |
| Removing a required field | Yes |
| Changing a field type | Yes |
| Removing an endpoint | Yes |
| Changing error code values | Yes |
| Changing signing algorithm | Yes |
| Making an optional field required | Yes (treat as breaking) |

When in doubt: treat as breaking. The cost of a forced migration exceeds the cost of a deprecated optional field.

---

## Conformance Tiers

A runtime or provider is "Engram conformant" at one of three tiers. These are the tiers used in the public implementation registry.

### Reader (Tier 1)
Minimum viable conformance. Sufficient for public listing.

- Calls `GET /v1/context` at session start or on a defined schedule
- Injects `system_prompt_injection` into LLM system prompt
- Checks `expires_at` — rejects expired exports
- Links to engramspec.org in integration documentation

### Writer (Tier 2)
Full read-write lifecycle.

- All Tier 1 requirements
- Submits corrections via `POST /v1/context/correct`
- Implements incremental sync via `GET /v1/context/diff`
- Surfaces pending corrections to user for confirmation (does not auto-apply)

### Full (Tier 3)
Production-grade conformance.

- All Tier 2 requirements
- Verifies Ed25519 signature on every response (required from v0.2)
- Implements discovery via `GET /.well-known/engram`
- Implements scoped exports via `GET /v1/context?scope=`
- Has passed the Engram conformance test suite (when published)

---

## What Cannot Be Changed — Constitutional Constraints

The following properties of Engram are locked. No RFC can remove them. No version bump can override them. They are the social contract between Engram and its users.

**1. `GET /v1/context` must always be available.**
A user's right to export their own memory is absolute. No authenticated provider may withhold this endpoint.

**2. `expires_at` is mandatory.**
No export may be served without an expiry. Stale memory must not persist silently.

**3. The `corrections` array is append-only.**
Corrections are never deleted from the audit trail. Users may tombstone them, but the record of correction stays.

**4. `status: "deleted"` beliefs must appear in exports.**
Deletion is a user right. But tombstones must propagate so caches can be purged.

**5. No provider may monetise export.**
`GET /v1/context` may not be paywalled. A provider may charge for storage, write operations, advanced features — not for the user's right to read their own memory.

---

## Dispute Resolution

**Spec ambiguity:** Open a GitHub Issue tagged "ambiguity". Steering group clarifies within 7 days. Clarification is normative — same weight as the spec text.

**Implementation dispute:** If two implementations disagree on correct behaviour, the reference implementation (8mem) is authoritative for v0.1. After v1.0, the written spec is authoritative.

**Governance dispute:** Escalate to Chair. If Chair is the dispute party, escalate to full steering group vote. Majority rules.

**Bad-faith actors:** Any implementation that uses "Engram compatible" branding without meeting Tier 1 requirements may be removed from the registry by steering group vote.

---

## The Foundation Path

When Engram reaches 10+ independent implementations, the governance model will migrate:

```
Current:  Engram Steering Group (informal, 8mem-led)
  ↓
Future:   Engram Foundation (nonprofit or fiscally-sponsored project)
          Similar to: OpenID Foundation, Apache Software Foundation (project level)
          8mem contributes the spec. Foundation owns it.
          8mem retains reference implementation and commercial products.
```

This is the move that makes Engram permanently neutral. The spec belongs to the community. 8mem captures value through being the best implementation, not through owning the standard.

Timeline: initiate Foundation formation when ≥3 independent runtimes implement Engram without 8mem writing the adapter.

---

## Joining the Steering Group

To be considered for a steering group seat:
1. Your runtime or provider must be publicly listed in the implementation registry
2. You must have actively participated in ≥2 RFC discussions
3. Nomination: self-nominate via GitHub Issue tagged "steering-group-nomination"
4. Approval: existing steering group votes within 14 days

New seats are not created on demand — they open on the schedule defined in the "how it grows" section above.

---

## Contact and Location

| Resource | URL |
|----------|-----|
| Spec repository | github.com/engramspec/spec |
| RFC discussions | github.com/engramspec/spec/discussions |
| Implementation registry | engramspec.org/implementations |
| Steering group | governance@engramspec.org |
| Spec document | engramspec.org |

---

*Engram Governance Model v1.0*
*Effective 2026-06-03*
*Maintained by the Engram Steering Group*
*The spec is CC-BY 4.0. This governance document is CC-BY 4.0.*
