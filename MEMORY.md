# Project Memory

Persistent context for this project. Read this file first before making changes. Update it whenever a significant decision is made, a constraint is discovered, or the architecture changes.

## What This Project Is

An **HTTPS-based web application** for **entry logging on aviation premises**. Core capabilities:

1. **Entry logging** — record every entry/exit event at the premises.
2. **New visitor registration** — capture and approve first-time visitors.
3. **Automatic clearance** — visitors who were previously approved and whose clearance is **unexpired** are automatically issued clearance on return.
4. **ANPR (Automatic Number Plate Recognition)** — a camera at the gate reads the vehicle's number plate and identifies the visitor, feeding the automatic clearance flow.

## Core Rules (Business Logic)

- Clearance is only auto-issued when **both** conditions hold: the visitor was previously **approved** AND the clearance is **not expired**.
- Expired, revoked, or never-approved visitors must go through the registration/approval flow — never auto-admit them.
- Every entry decision (auto or manual, granted or denied) must produce a log entry. No silent entries.
- Entry logs are append-only; they may never be edited or deleted by application users.
- A plate match alone is a *candidate* identification, not proof — the clearance record attached to that plate is what authorizes entry.

## Hard Constraints

- **HTTPS only.** All endpoints served over TLS; HTTP requests redirect or are refused. No mixed content.
- Aviation premises are security-sensitive: assume audits, so favor traceability and explicit denial reasons over convenience.
- Camera/ANPR input is untrusted sensor data — handle unreadable, partial, or spoofed plates gracefully (fall back to manual verification).

## Key Entities (Domain Model)

| Entity | Notes |
|---|---|
| **Visitor** | Person record: name, ID document, company, host, contact |
| **Vehicle** | Number plate (primary ANPR key), type, color; linked to visitor(s) |
| **Clearance** | Approval with validity window (issued_at, expires_at), status (approved / expired / revoked / pending) |
| **EntryLog** | Immutable event: who, when, which gate, vehicle, decision, decided-by (auto/ANPR or operator) |
| **Operator** | Guard/admin user of the system, with role-based permissions |
| **Gate / Checkpoint** | Physical entry point, optionally with an ANPR camera attached |

## Decisions Made

| Date | Decision | Rationale |
|---|---|---|
| 2026-07-04 | Project documented before code; docs-first approach | Establish shared understanding of scope and security rules |
| 2026-07-04 | HTTPS mandated from day one, including local dev | Security-sensitive aviation context; avoids retrofit |

## Open Questions

- Which ANPR approach: off-the-shelf engine (e.g. OpenALPR/ALPR SDK, cloud vision API) vs. custom model? Depends on camera hardware and offline requirements.
- Clearance validity policy: fixed duration, per-visit, or host-defined expiry?
- Regulatory requirements at the specific premises (e.g. ICAO Annex 17 / national aviation security programme) — confirm audit/retention rules before finalizing the log schema.
- Self-service kiosk registration vs. guard-operated only?
- Tech stack not yet chosen — see BUILD_PLAN.md Phase 0.

## Conventions

- All project documentation lives in the repository root as Markdown.
- Log every meaningful development session in `DEVLOG.md` (newest entry first).
- Keep this file short and current — prune superseded decisions into DEVLOG.md rather than letting this file grow stale.
