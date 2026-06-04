# Security Policy

## Scope

This document covers security issues in the **Engram Memory Protocol specification** itself — the schema, API contract, signing model, and governance rules.

Security issues in specific implementations (such as 8mem) should be reported to the respective maintainer, not here.

---

## Supported Versions

| Version | Supported |
|---------|-----------|
| 0.1.x | Yes — current |
| < 0.1 | No |

---

## Reporting a Vulnerability

**Do not open a public GitHub Issue for security vulnerabilities.**

Report privately to: **security@engramspec.org**

Include:
- Description of the vulnerability
- Which part of the spec is affected (schema, signing, API contract, scoping model)
- Proof of concept or concrete scenario where the vulnerability applies
- Proposed fix if you have one

**Response time:** We will acknowledge within 48 hours and provide an initial assessment within 7 days.

---

## Disclosure Policy

1. Reporter submits to security@engramspec.org
2. Steering group acknowledges within 48 hours
3. Steering group assesses severity and scope within 7 days
4. If confirmed: spec errata or RFC drafted, fix timeline communicated to reporter
5. Fix published in spec update
6. Reporter credited in CHANGELOG unless they prefer anonymity
7. Public disclosure after fix is published (coordinated with reporter)

---

## Known Security Properties and Limitations (v0.1)

These are documented limitations, not vulnerabilities — they are stated honestly in the spec (§12):

**Ed25519 signing is defined but not enforced in reference runtimes.**
The 8mem server signs every export. Reference runtimes (OpenClaw, Hermes) do not yet verify the signature. This means an attacker who can modify the context file on a local machine could inject false memory without detection. Mitigation: v0.2 will require verification for Tier 2+ conformance. Risk is low for personal-use deployments where the user controls both the provider and the runtime.

**`GET /v1/context/diff` is not independently signed.**
The diff endpoint is an optimisation for trusted channels. Runtimes requiring tamper-proof deltas should call `GET /v1/context` for a full signed export. This is documented in §3.9 of the spec.

**Token issuance is not standardised in v0.1.**
The spec does not define how a user grants a runtime access to their Engram store. Until v0.2 defines the delegation flow, runtimes use tokens issued directly by the provider's own auth system. This is documented in §12.1.

---

## What Would Constitute a Spec-Level Vulnerability

- A flaw in the Ed25519 signing specification (§3.8) that allows signature forgery
- A schema design that enables PII leakage across scoped exports
- An API contract design that allows a runtime to write to user memory without user confirmation
- A versioning rule that would allow a downgrade attack
- A `/.well-known/engram-keys` design flaw that enables key confusion attacks

---

*Engram Security Policy*
*Contact: security@engramspec.org*
*Spec: [engramspec.org](https://engramspec.org)*
