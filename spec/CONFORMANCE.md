# Engram Conformance

## How to verify your Engram implementation is correct

> Version: 0.1
> Test runner: [github.com/engramspec/conformance](https://github.com/engramspec/conformance) (coming Month 2)

---

## Conformance Tiers

A runtime or provider is conformant at one of three tiers. Tier determines your listing in the [implementation registry](https://engramspec.org/implementations).

### Tier 1 — Reader

Minimum viable conformance. Sufficient for public listing.

**Provider requirements:**
- [ ] `GET /v1/context` returns HTTP 200 with valid Engram envelope
- [ ] `expires_at` is present and not null
- [ ] `expires_at` is no more than 24 hours after `issued_at`
- [ ] `engram_version` is present
- [ ] `identity.display_name` and `identity.timezone` are present
- [ ] `GET /.well-known/engram` returns valid discovery document
- [ ] Unauthenticated requests return HTTP 401 with `error.code: "unauthorized"`

**Runtime requirements:**
- [ ] Calls `GET /v1/context` at session start or on a defined schedule
- [ ] Injects `system_prompt_injection` into LLM system prompt (or builds equivalent from beliefs array)
- [ ] Rejects exports where `expires_at` is in the past
- [ ] Links to engramspec.org in integration documentation

---

### Tier 2 — Writer

Full read-write memory lifecycle.

**All Tier 1 requirements, plus:**

**Provider requirements:**
- [ ] `POST /v1/context/correct` accepts corrections and returns `status: "pending_user_confirmation"`
- [ ] Corrections are NOT auto-applied — user confirmation required before becoming authoritative
- [ ] `GET /v1/context/diff?since=` returns incremental changes with tombstones for deleted beliefs
- [ ] `runtime_id` is required in correction requests

**Runtime requirements:**
- [ ] Calls `POST /v1/context/correct` when user corrects a belief
- [ ] Does NOT treat a correction as authoritative until user confirms
- [ ] Implements incremental sync via `GET /v1/context/diff`

---

### Tier 3 — Full

Production-grade conformance. Required for enterprise use cases.

**All Tier 2 requirements, plus:**

**Provider requirements:**
- [ ] `GET /.well-known/engram-keys` returns Ed25519 public key list
- [ ] Each key has `kid`, `alg: "Ed25519"`, `public_key` (base64url), `created_at`, `expires_at`
- [ ] Every export envelope has a `kid` field matching a key in `/.well-known/engram-keys`
- [ ] Signature verifies: Ed25519 over canonical JSON (alphabetically sorted keys, no whitespace, UTF-8)
- [ ] Scoped exports work: `GET /v1/context?scope=professional` excludes health and financial beliefs
- [ ] `GET /v1/context?scope=invalid` returns HTTP 400 with `error.code: "invalid_scope"`

**Runtime requirements:**
- [ ] Fetches public keys from `/.well-known/engram-keys`
- [ ] Selects key by matching `kid` in envelope
- [ ] Verifies Ed25519 signature on every received export
- [ ] Rejects exports where signature verification fails
- [ ] Requests scoped exports when full context is not required

---

## Self-Certification Checklist

Run through these checks against your implementation before submitting for listing.

### Provider Checklist

```bash
# 1. Basic response
curl -s https://your-provider.com/v1/context \
  -H "Authorization: Bearer $TOKEN" | jq '.engram_version'
# → should return "0.1"

# 2. expires_at is present and in the future
curl -s https://your-provider.com/v1/context \
  -H "Authorization: Bearer $TOKEN" | jq '.expires_at'
# → ISO8601 timestamp, must be in the future

# 3. Unauthenticated request returns 401
curl -s -o /dev/null -w "%{http_code}" https://your-provider.com/v1/context
# → 401

# 4. Discovery endpoint
curl -s https://your-provider.com/.well-known/engram | jq '.engram_version'
# → "0.1"

# 5. Correction endpoint
curl -s -X POST https://your-provider.com/v1/context/correct \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"belief_id":"test","new_value":"test","runtime_id":"test","timestamp":"2026-01-01T00:00:00Z"}' \
  | jq '.status'
# → "pending_user_confirmation"
```

### Runtime Checklist

- [ ] Your runtime calls `GET /v1/context` — confirm via network log
- [ ] `system_prompt_injection` from the response appears in your LLM system prompt
- [ ] If you receive an export with `expires_at` in the past, your runtime rejects it (does not inject stale context)
- [ ] Your documentation links to engramspec.org

---

## Claiming a Tier Badge

Add the appropriate badge to your README:

```markdown
[![Engram Reader v0.1](https://engramspec.org/badges/reader-v0.1.svg)](https://engramspec.org/implementations)
[![Engram Writer v0.1](https://engramspec.org/badges/writer-v0.1.svg)](https://engramspec.org/implementations)
[![Engram Full v0.1](https://engramspec.org/badges/full-v0.1.svg)](https://engramspec.org/implementations)
```

Self-certification: run the checklist, add the badge, open a GitHub Issue with label `implementation` to be listed in the registry. The steering group does not actively audit — but bad-faith claims can be reported and result in removal.

---

## Automated Test Suite

A bash-based automated test runner is in development at [github.com/engramspec/conformance](https://github.com/engramspec/conformance).

When available, run it against your provider:

```bash
git clone https://github.com/engramspec/conformance
cd conformance
./run-provider.sh https://your-provider.com YOUR_TEST_TOKEN
```

Expected output includes a pass/fail report per tier and a recommended tier badge.

---

## Test Fixtures

### Minimal valid export

```json
{
  "engram_version": "0.1",
  "issued_at": "2026-01-01T10:00:00Z",
  "expires_at": "2026-01-01T22:00:00Z",
  "kid": "test-key-001",
  "issuer": { "name": "Test Provider", "url": "https://example.com" },
  "subject": { "id": "550e8400-e29b-41d4-a716-446655440000", "display_name": "Test User" },
  "scope": "full",
  "signature": "base64url-placeholder",
  "identity": {
    "display_name": "Test User",
    "timezone": "UTC",
    "created_at": "2026-01-01T00:00:00Z",
    "last_updated": "2026-01-01T00:00:00Z"
  },
  "beliefs": [],
  "corrections": [],
  "evolution": [],
  "system_prompt_injection": "User: Test User (UTC)."
}
```

### Expired export (runtimes must reject)

Same as above but with `"expires_at": "2020-01-01T00:00:00Z"`.

A conformant runtime must not inject context from an expired export.

---

## Error Code Reference

All non-2xx responses must use this shape:

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Token missing or invalid",
    "status": 401
  }
}
```

| Code | HTTP | Meaning |
|------|------|---------|
| `unauthorized` | 401 | Token missing or invalid |
| `token_expired` | 401 | Token has expired |
| `forbidden` | 403 | Token valid but insufficient scope |
| `user_not_found` | 404 | No Engram record for this subject |
| `belief_not_found` | 404 | Belief ID does not exist |
| `invalid_scope` | 400 | Unknown or malformed scope |
| `invalid_request` | 400 | Malformed request body |
| `rate_limited` | 429 | Too many requests |
| `unsupported_version` | 400 | `engram_version` not supported |
| `server_error` | 500 | Internal error |

---

*Engram Conformance — v0.1*
*Spec: [engramspec.org](https://engramspec.org) — Full RFC: [`v0.1.md`](v0.1.md)*
