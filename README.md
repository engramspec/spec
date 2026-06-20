<p align="center">
  <img src="assets/engramspec_logo_transparent.png" alt="EngramSpec open memory protocol logo" style="width: 100%; max-width: 760px; height: auto;">
</p>

# EngramSpec

**The open standard for portable, governed AI memory.**

[![Status](https://img.shields.io/badge/status-RFC%20Draft%20v0.1-blue)](spec/v0.1.md)
[![License](https://img.shields.io/badge/license-CC%20BY%204.0-green)](LICENSE)
[![Spec](https://img.shields.io/badge/spec-engramspec.org-black)](https://engramspec.org)

---

## Abstract

Every AI system today starts from zero knowledge of its user. Memory is siloed by product, by company, by session — a structural problem no single AI company has incentive to fix. EngramSpec defines Engram, an open protocol that lets any AI runtime retrieve structured, signed, user-governed memory from any conforming provider. One HTTP endpoint. Any runtime. User controls everything.

Engram is a format specification — like JSON, like OAuth — not a product. Anyone can implement it. No one owns it.

---

## The Problem

| Today | With Engram |
|-------|-------------|
| Every AI starts from zero | Any AI that speaks Engram knows you instantly |
| Memory locked per product | Memory follows you across runtimes |
| No correction audit trail | Every change recorded — old value, new value, when, why |
| Provider owns your data | You own your memory. Export is unconditional and free. |
| No standard format | One envelope. Any language. Any framework. |

---

## How It Works

A conforming provider exposes one required endpoint:

```bash
GET /v1/context
Authorization: Bearer <token>
```

Any AI runtime calls this at session start and receives a signed JSON envelope:

```json
{
  "engram_version": "0.1",
  "issued_at": "2026-06-03T09:00:00Z",
  "expires_at": "2026-06-03T21:00:00Z",
  "kid": "key-2026-06",
  "signature": "Ed25519-signature-over-payload",
  "system_prompt_injection": "User is Ashish (Asia/Singapore). Prefers short answers. No emoji. Building AI memory infrastructure.",
  "identity": {
    "display_name": "Ashish",
    "timezone": "Asia/Singapore",
    "role": "Founder"
  },
  "beliefs": [
    {
      "category": "communication",
      "key": "response_length",
      "value": "short and direct",
      "confidence": 0.95,
      "source": "user_stated",
      "status": "active"
    }
  ],
  "corrections": [],
  "evolution": []
}
```

Inject `system_prompt_injection` into your LLM system prompt. Done. Your runtime knows its user.

---

## Minimal Integration — Any Runtime

```python
from datetime import datetime, timezone
import requests

def get_engram_context(provider_url: str, token: str) -> str:
    r = requests.get(
        f"{provider_url}/v1/context",
        headers={"Authorization": f"Bearer {token}"}
    )
    r.raise_for_status()
    envelope = r.json()

    # Reject expired exports
    expires = datetime.fromisoformat(envelope["expires_at"].replace("Z", "+00:00"))
    if expires < datetime.now(timezone.utc):
        raise ValueError("Engram export expired — fetch a fresh one")

    # Use the pre-built injection string
    return envelope.get("system_prompt_injection", "")

# Before every LLM call:
context = get_engram_context("https://api.8mem.com", user_token)
system_prompt = f"{base_prompt}\n\n{context}"
```

```bash
# Or from the shell — 30 seconds to wire any runtime
curl https://api.8mem.com/v1/context \
  -H "Authorization: Bearer $ENGRAM_TOKEN" \
  | jq -r '.system_prompt_injection'
```

---

## The Four Core Objects

| Object | What it stores | Trust model |
|--------|---------------|-------------|
| `identity` | Stable facts — name, timezone, role | Highest — rarely changes |
| `beliefs` | Dynamic facts — preferences, habits, working style | High — user-correctable, confidence-scored |
| `corrections` | Explicit user overrides — full audit trail | Highest — user-initiated, immutable |
| `evolution` | History of every belief change — old, new, trigger, context | Append-only — permanent record |

---

## Full API Contract

### Required Endpoints

```
GET  /v1/context                       Full signed export
GET  /v1/context?scope={scope}         Scoped export
GET  /v1/context?categories={cat}      Category-filtered export
POST /v1/context/correct               Submit correction (pending user confirmation)
GET  /.well-known/engram               Provider discovery
```

### Optional (required from v0.2)

```
GET  /v1/context/diff?since={ISO8601}  Incremental sync with tombstones
GET  /.well-known/engram-keys          Ed25519 public key distribution
```

### Scoped Exports

| Scope | Categories included |
|-------|---------------------|
| `full` | Everything |
| `professional` | communication, work_style, projects, decision_making, values |
| `personal` | relationships, health, learning, custom |
| `financial` | financial only |
| `minimal` | identity + communication only |
| `custom` | User-defined |

---

## Conformance Tiers

A runtime or provider is listed in the implementation registry at one of three tiers:

| Tier | Requirements |
|------|-------------|
| **Reader** | Calls `GET /v1/context`, injects `system_prompt_injection`, checks `expires_at` |
| **Writer** | + submits corrections via `POST /v1/context/correct`, implements incremental diff |
| **Full** | + verifies Ed25519 signature, implements scoped exports, passes conformance test suite |

Reader is sufficient for public listing. See [`spec/CONFORMANCE.md`](spec/CONFORMANCE.md) for the full test suite.

---

## Security Model

Every export is signed with the provider's **Ed25519** private key. Runtimes verify using public keys from `/.well-known/engram-keys`. Every export has a mandatory `expires_at` — maximum 24 hours. Runtimes must reject expired envelopes.

Eight-step verification process: parse → check version → check expiry → fetch public keys → match `kid` → verify signature → check clock skew → apply filters. See [§3.8 of the spec](spec/v0.1.md) for the full process.

**v0.1 note:** signature verification is defined in the spec but not yet enforced by reference runtimes. Required for Tier 2+ conformance from v0.2. Documented honestly in [§12 of the spec](spec/v0.1.md).

---

## Reference Implementation

[8mem](https://github.com/tempomesh/8mem-clean) (`pip install 8mem`) is the reference implementation of Engram.

| Feature | Status |
|---------|--------|
| `GET /v1/context` — full export | Live |
| `GET /v1/context?scope=` — scoped export | Live |
| `POST /v1/context/correct` — runtime corrections | Live |
| `GET /v1/context/diff` — incremental sync | Live |
| `GET /.well-known/engram` — discovery | Live |
| `GET /.well-known/engram-keys` — key distribution | Live |
| Ed25519 signing (server-side) | Live |
| Ed25519 verification (client-side) | v0.2 |
| Delegation / consent flow | v0.2 |

---

## Live Implementations

| Runtime | Language | Tier | Maintained by |
|---------|----------|------|---------------|
| [OpenClaw](https://openclaw.ai) | Node.js | Writer | OpenClaw Foundation |
| [Hermes](https://hermes.ai) | Python | Writer | Nous Research |

*Building an Engram integration? Open an issue with label `implementation` to be listed here.*

---

## Relation to MCP

MCP (Model Context Protocol, Anthropic 2024) and Engram operate at the same layer and are complementary:

```
MCP:    defines what AI can DO  →  call tools, access files, run code
Engram: defines who AI is TALKING TO  →  identity, preferences, corrections, history
```

A runtime can implement both. 8mem exposes Engram as both an HTTP API and an MCP server.

---

## Comparison with Prompt Stuffing

Every AI product today injects user context as raw text in the system prompt. Engram is not that.

| Dimension | Prompt stuffing | Engram |
|-----------|----------------|--------|
| Portability | Zero — one product | Full — any runtime |
| Ownership | Provider owns data | User owns data |
| Correctability | Overwrite (lossy) | Correction chain (permanent audit) |
| Verification | None | Ed25519 signed, `expires_at` enforced |
| Discovery | Manual | `/.well-known/engram` |
| Staleness | None | `stale_after_days` per belief |
| Standard format | No | Yes — CC-BY 4.0 |

---

## Known Limitations (v0.1)

Documented honestly in [Part 12 of the spec](spec/v0.1.md). Summary:

| Gap | Planned |
|-----|---------|
| No delegation / consent flow (the OAuth equivalent) | v0.2 |
| No signature verification in reference runtimes | v0.2 |
| No multi-store discovery | v0.2 |
| No webhook / push model (pull-only) | v0.2 |
| No bulk correction endpoint | v0.2 |
| No machine-readable JSON Schema | v0.1.1 |

---

## Implementing Engram

**For runtime builders** (AI assistants, agents, copilots):
→ Call `GET /v1/context`. Inject `system_prompt_injection`. 30 minutes to Tier 1.

**For providers** (memory services):
→ Serve the envelope format defined in the spec. Implement `/.well-known/engram` for discovery.

**Full implementation guide:** [`spec/v0.1.md §5`](spec/v0.1.md)

**Official SDKs:** coming soon

- Python: `pip install engram`
- Node.js: `npm install @engram/client`

---

## Governance

Engram is governed by the Engram Steering Group. 8mem initiated the spec and chairs the group. Voting majority transfers to the community as independent implementations grow. See [`GOVERNANCE.md`](GOVERNANCE.md) for the full model, RFC process, and constitutional constraints.

Changes to the spec follow a public RFC process — open to anyone. See [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## Citation

If you use or reference Engram in academic work:

```bibtex
@misc{engram2026,
  title        = {Engram: An Open Protocol for Portable, Governed {AI} Memory --- The {OAuth} for {AI} Memory},
  author       = {Verma, Ashish},
  year         = {2026},
  doi          = {10.2139/ssrn.6878038},
  url          = {https://papers.ssrn.com/abstract=6878038},
  howpublished = {\url{https://engramspec.org}},
  note         = {SSRN Working Paper. RFC Draft v0.1. CC BY 4.0.}
}
```

Working paper: Verma, A. (2026). *Engram: An Open Protocol for Portable, Governed AI Memory — The OAuth for AI Memory.* SSRN. [https://doi.org/10.2139/ssrn.6878038](https://doi.org/10.2139/ssrn.6878038)

---

## Repository Structure

```
spec/
  v0.1.md          — Full RFC specification (normative)
  CONFORMANCE.md   — Conformance test suite and certification tiers
GOVERNANCE.md      — Steering group, RFC process, constitutional constraints
CONTRIBUTING.md    — How to propose changes
CHANGELOG.md       — Version history
LICENSE            — CC BY 4.0
```

---

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)

Free to implement, fork, and build upon. Attribution required.

---

*Engram Memory Protocol — v0.1*
*Maintained by the Engram Steering Group. Initiated by [8mem](https://8mem.com).*
*Specification: [engramspec.org](https://engramspec.org)*
