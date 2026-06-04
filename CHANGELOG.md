# Changelog

All changes to the Engram Memory Protocol are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned for v0.1.1
- JSON Schema file at `engramspec.org/schema/v0.1.json`

### Planned for v0.2
- Delegation and consent flow — user grants runtime X access to scope Y for duration Z
- Ed25519 signature verification required for Tier 2+ conformance
- Webhook / push model — `POST /v1/context/subscribe`
- Multi-store discovery — WebFinger-style user-level provider discovery
- Bulk correction endpoint — `POST /v1/context/correct/batch`

---

## [0.1.2] — 2026-06-04

### Added
- `CONSTRAINTS` as fifth core object — hard rules the agent MUST NEVER violate (§3.7)
- Enforcement contract (normative): load before other objects, check before generation, refuse and surface if violated
- `POST /v1/context/constrain` — user sets a new hard constraint (Tier 2+)
- `DELETE /v1/context/constrain/{id}` — tombstones a constraint (Tier 2+)
- `constraint_not_found` (404) added to standard error codes
- Constraint tombstone pattern — deleted constraints remain in exports with `status: "deleted"`
- CONSTRAINTS included in every scope export by default
- `constraints` changes object added to `GET /v1/context/diff` response
- `constrain` endpoint added to `/.well-known/engram` discovery response
- Standard constraint categories: `communication`, `content`, `data`, `behaviour`, `safety`, `custom`

### Changed
- Schema overview: four core objects → five core objects
- Adoption tiers: Full tier now includes constraint management and Ed25519 verification
- Minimal runtime compatibility definition now requires loading and respecting CONSTRAINTS

---

## [0.1.1] — 2026-04-29

### Added
- `system_prompt_injection` field to top-level envelope — pre-formatted natural language string for direct injection into LLM system prompts (§3.2)
- Tiered injection policy guidance — which beliefs to include/exclude based on confidence and source (§3.2)
- Injection policy table: Always (identity + high-confidence beliefs), Conditional (professional beliefs ≥0.7), Never (confidence <0.7 or stale)

### Changed
- Reference implementation table updated — `GET /v1/context` status: Live

---

## [0.1] — 2026-04-20

### Added
- Core schema: four objects — `IDENTITY`, `BELIEFS`, `EVOLUTION`, `CORRECTIONS`
- Top-level envelope fields: `engram_version`, `issued_at`, `expires_at`, `kid`, `issuer`, `subject`, `scope`, `signature`
- `IDENTITY` object: `display_name`, `timezone`, `locale`, `role`, `domains`, `bio`
- `BELIEFS` array: `id` (UUIDv4), `category`, `key`, `value`, `value_type`, `confidence`, `source`, `status`, `created_at`, `last_confirmed`, `stale_after_days`, `tags`
- `EVOLUTION` array: append-only history of belief changes with `trigger` and `context`
- `CORRECTIONS` array: explicit user overrides with `corrected_by`, `method`, and full audit trail
- Standard belief categories: `communication`, `work_style`, `decision_making`, `relationships`, `projects`, `values`, `learning`, `health`, `financial`, `custom`
- Scoped exports: `full`, `professional`, `personal`, `financial`, `minimal`, `custom`
- `scope_definition` object for custom and named scopes — `issued_to` must be a URI
- Tombstone pattern: deleted beliefs appear in exports with `status: "deleted"` so runtimes can purge caches
- Staleness model: `stale_after_days` stored on server; staleness adjustment is a client-side computation (confidence unchanged in signed payload)
- Required API endpoints: `GET /v1/context`, `GET /v1/context?scope=`, `GET /v1/context?categories=`, `POST /v1/context/correct`, `GET /.well-known/engram`
- Optional endpoints: `GET /v1/context/diff?since=`, `GET /.well-known/engram-keys`
- Ed25519 asymmetric signing — private key signs, public key verifies, no shared secret
- `kid` field in envelope — identifies signing key for correct rotation without trying all keys
- Eight-step verification process (§3.8)
- `/.well-known/engram-keys` — key distribution endpoint with rotation support
- Mandatory `expires_at` — maximum 24 hours recommended; `null` treated as expired
- Error response format — consistent shape with `error.code`, `error.message`, `error.status`
- 11 standard error codes: `unauthorized`, `token_expired`, `forbidden`, `user_not_found`, `belief_not_found`, `invalid_scope`, `invalid_request`, `rate_limited`, `unsupported_version`, `server_error`
- Versioning rules: `0.x` draft phase, `1.0` stable, `1.x` backwards-compatible, `2.0` breaking
- `runtime_id` required on `POST /v1/context/correct` — displayed to user during confirmation
- `diff` endpoint security note: not independently signed; use full `GET /v1/context` for tamper-proof deltas
- Appendix A — full example export with valid UUIDv4 identifiers throughout
- Part 12 — known limitations and v0.2 roadmap (documented honestly)

---

*Engram Memory Protocol*
*Maintained by the Engram Steering Group — [engramspec.org](https://engramspec.org)*
