# Changelog

All notable changes to the Engram standard are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [0.1] — 2026-04-20

### Added
- Core schema: `IDENTITY`, `BELIEFS`, `EVOLUTION`, `CORRECTIONS`
- Top-level envelope with `engram_version`, `issued_at`, `issuer`, `subject`, `scope`, `signature`
- Standard belief categories: communication, work_style, decision_making, relationships, projects, values, learning, health, financial, custom
- Scoped exports: `full`, `professional`, `personal`, `financial`, `minimal`, `custom`
- Required API endpoints: `GET /v1/context`, `POST /v1/context/correct`, `GET /v1/context/diff`
- HMAC-SHA256 signature and verification process
- Versioning rules
- Full example export (Appendix A)

---

*Engram is initiated by [8mem](https://8mem.com) and open to all.*
