# Entry Control Platform

A secure, HTTPS-based web application for managing visitor entry at aviation premises. The platform logs every entry event, streamlines registration of new visitors, automatically issues clearance to previously approved visitors whose authorization has not expired, and integrates camera-based Automatic Number Plate Recognition (ANPR) to identify arriving vehicles.

## Key Features

- **Entry Logging** — Every entry and exit event is recorded with a timestamp, visitor identity, vehicle details, gate/checkpoint, and the operator or system that granted access. Logs are immutable and auditable.
- **New Visitor Registration** — Guards or self-service kiosks capture visitor details (name, ID document, company, host, purpose of visit, vehicle plate) and submit them for approval.
- **Automatic Clearance** — Returning visitors who were previously approved and whose clearance has **not expired** are automatically granted entry without re-registration. Expired or revoked clearances force re-approval.
- **ANPR / Number Plate Recognition** — A camera at the gate reads the vehicle's number plate, matches it against registered visitors, and triggers the automatic clearance flow when a valid match is found.
- **HTTPS Everywhere** — All client-server communication is encrypted with TLS. No plaintext HTTP endpoints are exposed.

## How It Works

```
Vehicle arrives at gate
        │
        ▼
 ANPR camera reads plate ──► Plate matched against visitor database
        │                              │
        │ no match / unreadable        │ match found
        ▼                              ▼
 Manual registration           Clearance valid & unexpired?
 (new visitor flow)                    │
        │                    ┌─────────┴─────────┐
        ▼                    ▼ yes                ▼ no (expired/revoked)
 Approval workflow    Auto-issue clearance   Re-approval required
        │                    │                    │
        └────────────────────┴────────────────────┘
                             │
                             ▼
                  Entry logged, gate opens
```

## Intended Environment

The platform is designed for controlled aviation premises (airports, airfields, MRO facilities, cargo terminals) where perimeter access is regulated and every entry must be traceable for security and compliance purposes.

## Project Documents

| Document | Purpose |
|---|---|
| [MEMORY.md](MEMORY.md) | Persistent project memory: decisions, constraints, and context for contributors and AI assistants |
| [DEVLOG.md](DEVLOG.md) | Chronological development log |
| [BUILD_PLAN.md](BUILD_PLAN.md) | Phased plan for building the application |
| [TEST_PLAN.md](TEST_PLAN.md) | Test strategy and test cases |

## Status

📋 **Planning phase** — requirements and documentation are being established. See [BUILD_PLAN.md](BUILD_PLAN.md) for the roadmap and [DEVLOG.md](DEVLOG.md) for progress.

## License

To be determined.
