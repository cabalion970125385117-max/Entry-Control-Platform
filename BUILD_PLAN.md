# Build Plan

Phased roadmap for building the Entry Control Platform. Each phase has a clear deliverable and exit criteria. Phases are sequential, but Phase 5 (ANPR) can be prototyped in parallel from Phase 2 onward.

## Phase 0 — Foundations & Decisions

**Goal:** Remove ambiguity before writing code.

- [ ] Choose tech stack (suggested baseline: TypeScript + Node.js/Next.js or Python/FastAPI backend, React frontend, PostgreSQL database — decide and record in MEMORY.md)
- [ ] Choose ANPR approach: off-the-shelf engine (OpenALPR / Plate Recognizer / cloud vision API) vs. custom OpenCV + OCR pipeline
- [ ] Define clearance expiry policy (fixed duration vs. host-defined vs. per-visit)
- [ ] Confirm regulatory/audit requirements for aviation premises (log retention period, data protection for ID documents and plate data)
- [ ] Set up repository structure, linting, formatting, CI pipeline skeleton

**Exit criteria:** All open questions in MEMORY.md answered and recorded.

## Phase 1 — Application Skeleton & HTTPS

**Goal:** A running, secure, empty application.

- [ ] Scaffold backend API and frontend app
- [ ] Enforce HTTPS: TLS termination, HTTP→HTTPS redirect, HSTS header, secure cookies
- [ ] Local development with self-signed/mkcert certificates so dev also runs over HTTPS
- [ ] Database setup with migrations tooling
- [ ] Authentication for operators (guards/admins) with role-based access control
- [ ] CI: build, lint, and test on every push

**Exit criteria:** Operator can log in over HTTPS; plain HTTP is refused; CI is green.

## Phase 2 — Core Domain: Visitors, Vehicles, Clearances

**Goal:** The data model and CRUD flows.

- [ ] Implement entities: Visitor, Vehicle, Clearance, EntryLog, Operator, Gate
- [ ] New visitor registration form (name, ID document, company, host, purpose, vehicle plate)
- [ ] Approval workflow: pending → approved (with validity window) / rejected
- [ ] Clearance lifecycle: approved → expired (automatic, time-based) / revoked (manual)
- [ ] Visitor & vehicle search for gate operators

**Exit criteria:** A visitor can be registered, approved with an expiry date, and looked up by plate number.

## Phase 3 — Entry Logging

**Goal:** Immutable, complete entry records.

- [ ] Append-only EntryLog: timestamp, visitor, vehicle, gate, decision (granted/denied), reason, decided-by (operator or system)
- [ ] Manual entry/exit logging UI for gate operators
- [ ] Log browsing with filters (date range, gate, visitor, plate, decision)
- [ ] Export for audits (CSV/PDF)
- [ ] Enforce immutability at the database layer (no UPDATE/DELETE grants on the log table)

**Exit criteria:** Every grant/deny produces a log row; logs cannot be altered via the application or its DB role.

## Phase 4 — Automatic Clearance

**Goal:** Returning approved visitors get in without re-registration.

- [ ] Clearance check service: given a visitor/plate, return ALLOW only if an approved AND unexpired clearance exists
- [ ] Auto-issue entry clearance on valid match and write the EntryLog entry marked as system-decided
- [ ] Deny path: expired/revoked/unknown → route to registration or re-approval, with an explicit denial reason logged
- [ ] Operator override with mandatory reason (logged)
- [ ] Notifications to host (optional, e.g. email) when their visitor arrives

**Exit criteria:** All decision-matrix cases in TEST_PLAN.md §Automatic Clearance pass.

## Phase 5 — ANPR (Number Plate Recognition)

**Goal:** Camera identifies the vehicle and drives the clearance flow.

- [ ] Camera ingestion: capture frames from the gate camera (RTSP stream or snapshot API)
- [ ] Plate detection + OCR via the engine chosen in Phase 0
- [ ] Normalization of plate strings (spacing, ambiguous characters like O/0, I/1) and confidence thresholding
- [ ] Match pipeline: plate → registered vehicle → visitor → clearance check (Phase 4)
- [ ] Low-confidence / unreadable / unregistered plate → flag for manual operator verification
- [ ] Store the plate image alongside the entry log entry as evidence
- [ ] Gate hardware integration hook (barrier open signal) — abstracted behind an interface, simulated in dev

**Exit criteria:** A registered vehicle driving up to the (simulated) camera is recognized and processed end-to-end; unreadable plates fall back to manual flow.

## Phase 6 — Hardening & Deployment

**Goal:** Production-ready.

- [ ] Security review: OWASP Top 10 pass, dependency audit, secrets management
- [ ] Rate limiting and input validation on all endpoints
- [ ] Backup and retention policy for logs and images per Phase 0 regulatory findings
- [ ] Monitoring, alerting, and health checks
- [ ] Load testing at expected gate throughput (vehicles/minute at peak)
- [ ] Deployment (target TBD in Phase 0), TLS certificates via managed CA (e.g. Let's Encrypt or organizational PKI)
- [ ] Operator training documentation

**Exit criteria:** Full TEST_PLAN.md pass, security review sign-off, deployed behind HTTPS with monitoring.

## Milestone Summary

| Milestone | Phases | Deliverable |
|---|---|---|
| M1 — Secure skeleton | 0–1 | HTTPS app with operator login |
| M2 — Visitor management | 2–3 | Registration, approval, entry logging |
| M3 — Smart entry | 4–5 | Auto-clearance + ANPR end-to-end |
| M4 — Production | 6 | Hardened, deployed, documented |
