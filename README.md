# Engram

**The open standard for portable, governed AI memory.**

> Status: RFC Draft v0.1
> License: [CC BY 4.0](LICENSE)

---

## The Problem

You use ChatGPT for a year. It learns how you think.
You open Claude. Starts from zero.
You try a new AI tool. Starts from zero.

Every AI locks your memory inside itself. You own nothing. You explain yourself forever.

---

## What Engram Is

Engram is a standard format for AI memory — like a passport every AI agrees to read.

> Engram is your AI memory card — visible to you, controlled by you, readable by any AI, owned by no one but you.

It is a **format specification** — like JSON, like OAuth.
Anyone can implement it. No one owns it. Everyone benefits.

---

## Four Core Objects

| Object | What it stores |
|--------|---------------|
| `IDENTITY` | Stable facts — name, timezone, role, background |
| `BELIEFS` | Dynamic facts — preferences, habits, working style, opinions |
| `EVOLUTION` | How beliefs changed over time — the before and after |
| `CORRECTIONS` | Explicit user overrides — the audit trail of trust |

---

## Minimal Integration

A runtime only needs to do two things to be Engram compatible:

1. Call `GET /v1/context` at session start
2. Inject the response into the LLM system prompt

```python
import requests

def get_user_context(user_token: str) -> str:
    response = requests.get(
        "https://your-8mem-instance/v1/context",
        headers={"Authorization": f"Bearer {user_token}"}
    )
    engram = response.json()
    identity = engram["identity"]
    beliefs = engram["beliefs"]

    context = f"User: {identity['display_name']}\n"
    context += f"Timezone: {identity['timezone']}\n\n"
    context += "What I know about this user:\n"
    for belief in beliefs:
        context += f"- {belief['key']}: {belief['value']}\n"
    return context

# Before LLM call:
system_prompt = base_system_prompt + "\n\n" + get_user_context(user_token)
```

One endpoint. Memory works everywhere.

---

## Required API Endpoints

```
GET  /v1/context                           — full export
GET  /v1/context?scope={scope}             — scoped export (professional / personal / minimal)
GET  /v1/context?categories={cat1},{cat2}  — category-filtered export
POST /v1/context/correct                   — submit correction from runtime
GET  /v1/context/diff?since={ISO8601}      — incremental sync
```

---

## Scoped Exports

Users share only what they choose:

| Scope | Included |
|-------|----------|
| `full` | Everything |
| `professional` | communication, work_style, projects, decision_making, values |
| `personal` | relationships, health, learning, custom |
| `financial` | financial only |
| `minimal` | identity + communication only |
| `custom` | User-defined category list |

---

## Full Specification

See [`spec/v0.1.md`](spec/v0.1.md) for the complete RFC — full schema, field definitions, API contract, versioning rules, and example export.

---

## Reference Implementation

[8mem](https://8mem.com) is the reference implementation of the Engram standard.

| Feature | Status |
|---------|--------|
| Memory capture (BELIEFS) | Live |
| Correction loop (CORRECTIONS) | Live |
| Contradiction detection | Live |
| /passport (IDENTITY view) | Live |
| Evolution tracking | In testing |
| GET /v1/context API | In development |
| Scoped exports | Planned |
| Signed exports | Planned |

---

## Why This Standard Exists

Every AI company has incentive to lock your memory inside their product. Your memory is their moat. No big company will ever publish a portable memory standard — it helps competitors.

Engram can only come from a neutral party.

---

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)

Free to implement. Free to fork. Attribution required.

---

## Contributing

This is an open standard. Feedback, issues, and proposals welcome via GitHub Issues.

*Initiated by [8mem](https://8mem.com). Open to all.*
