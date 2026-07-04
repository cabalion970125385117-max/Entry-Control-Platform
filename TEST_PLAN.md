# Test Plan

Test strategy and case catalogue for the Entry Control Platform. Test cases are written against the business rules in [MEMORY.md](MEMORY.md) and map to the phases in [BUILD_PLAN.md](BUILD_PLAN.md).

## Strategy

| Level | Scope | Tooling (to confirm with stack choice) |
|---|---|---|
| Unit | Domain logic: clearance validity, plate normalization, expiry calculation | Stack-native test runner (Jest/Vitest or pytest) |
| Integration | API endpoints + database, approval workflow, log immutability | API test suite against a test database |
| End-to-end | Browser flows over HTTPS: registration, gate operation, log browsing | Playwright |
| ANPR | Plate recognition accuracy against a labelled image set | Offline evaluation harness + integration tests with mocked camera |
| Security | HTTPS enforcement, authz, input validation | Automated scans + manual review (Phase 6) |

**Principles:**
- Every business rule in MEMORY.md has at least one test that would fail if the rule were violated.
- The auto-clearance decision matrix (below) is the highest-priority suite — it guards the security-critical path.
- ANPR is tested separately from the clearance decision, so recognition errors never mask authorization bugs.

## 1. HTTPS Enforcement

| ID | Case | Expected |
|---|---|---|
| HTTPS-01 | Request any endpoint over plain HTTP | Redirected to HTTPS or refused; never served content |
| HTTPS-02 | Response headers on any HTTPS page | HSTS present; cookies `Secure` + `HttpOnly` |
| HTTPS-03 | TLS configuration scan | No TLS < 1.2, no weak ciphers |
| HTTPS-04 | Mixed-content check on all pages | No resources loaded over HTTP |

## 2. Visitor Registration

| ID | Case | Expected |
|---|---|---|
| REG-01 | Register new visitor with all required fields (incl. vehicle plate) | Record created with status `pending` |
| REG-02 | Submit with missing required fields | Validation error; nothing persisted |
| REG-03 | Register a plate already linked to another visitor | Flagged for operator review, not silently merged |
| REG-04 | Duplicate registration of the same person | Detected and surfaced; no duplicate visitor records |
| REG-05 | Approve pending visitor with validity window | Clearance becomes `approved` with `expires_at` set |
| REG-06 | Reject pending visitor | Status `rejected`; visitor cannot be auto-cleared |
| REG-07 | Malicious input in text fields (XSS/SQLi payloads) | Stored/escaped safely; no execution or injection |

## 3. Automatic Clearance — Decision Matrix (critical)

Given a visitor identified at the gate:

| ID | Prior approval | Clearance state | Expected decision |
|---|---|---|---|
| AUTO-01 | Yes | Unexpired | **ALLOW** — auto-issue clearance, log as system-decided |
| AUTO-02 | Yes | Expired (even by 1 second) | **DENY** — route to re-approval; reason `expired` logged |
| AUTO-03 | Yes | Revoked | **DENY** — reason `revoked` logged |
| AUTO-04 | Pending (never approved) | — | **DENY** — reason `not approved` logged |
| AUTO-05 | Rejected | — | **DENY** — reason `rejected` logged |
| AUTO-06 | No record at all | — | **DENY** — route to new registration |
| AUTO-07 | Yes | Expires during the visit | Entry decision uses expiry **at time of arrival** |
| AUTO-08 | Multiple clearances (one expired, one valid) | Mixed | **ALLOW** based on the valid one; correct clearance referenced in log |

Additional cases:

| ID | Case | Expected |
|---|---|---|
| AUTO-09 | Operator override of a DENY | Allowed only with mandatory reason; logged as operator-decided with reason |
| AUTO-10 | Clock/timezone edge: expiry stored in UTC, gate in local time | Expiry compared correctly in UTC |
| AUTO-11 | Two gates process the same visitor simultaneously | Both decisions logged; no crash or duplicate clearance issuance |

## 4. ANPR (Number Plate Recognition)

### 4.1 Recognition quality (offline evaluation set)

| ID | Case | Expected |
|---|---|---|
| ANPR-01 | Clean, well-lit frontal plate images | ≥ target accuracy (set threshold in Phase 0, e.g. 95%) |
| ANPR-02 | Night/low-light images | Meets night-time threshold or flags low confidence |
| ANPR-03 | Angled, partially occluded, or dirty plates | Low-confidence result → manual verification, never a false ALLOW |
| ANPR-04 | Ambiguous characters (O/0, I/1, B/8) | Normalization rules applied consistently on both read and stored plates |

### 4.2 Pipeline integration

| ID | Case | Expected |
|---|---|---|
| ANPR-05 | Registered plate, valid clearance | End-to-end: read → match → AUTO-01 ALLOW → log with plate image attached |
| ANPR-06 | Registered plate, expired clearance | Read succeeds, decision is DENY per AUTO-02 |
| ANPR-07 | Unregistered plate | Routed to manual registration; logged as unmatched read |
| ANPR-08 | Unreadable frame / camera offline | Graceful fallback to manual operator flow; incident visible in UI |
| ANPR-09 | Plate matches vehicle linked to multiple visitors | Operator disambiguation required; no automatic guess |
| ANPR-10 | Spoofed/cloned plate concern: plate valid but flagged vehicle attributes mismatch (if attributes recorded) | Flag for operator verification |
| ANPR-11 | Confidence below threshold but plate happens to match | Manual verification required — confidence gate is enforced before matching |

## 5. Entry Logging

| ID | Case | Expected |
|---|---|---|
| LOG-01 | Every ALLOW (auto or manual) | Exactly one log row: timestamp, visitor, vehicle, gate, decision, decided-by |
| LOG-02 | Every DENY | Log row with explicit denial reason |
| LOG-03 | Attempt to UPDATE/DELETE a log row via API | No such endpoint exists / request rejected |
| LOG-04 | Attempt to UPDATE/DELETE via the application's DB role | Denied by database grants |
| LOG-05 | Filter logs by date, gate, plate, visitor, decision | Correct filtered results |
| LOG-06 | Audit export (CSV/PDF) | Matches on-screen data exactly |
| LOG-07 | High-volume entry burst (peak gate throughput) | No lost log entries; write is part of the entry transaction |

## 6. Authentication & Authorization

| ID | Case | Expected |
|---|---|---|
| AUTH-01 | Unauthenticated access to any operator endpoint | 401/redirect to login |
| AUTH-02 | Guard role attempts admin action (e.g. revoke clearance, manage operators) | 403 |
| AUTH-03 | Session expiry / logout | Session invalidated server-side |
| AUTH-04 | Brute-force login attempts | Rate limited / lockout |

## 7. Non-Functional

| ID | Case | Expected |
|---|---|---|
| NFR-01 | Gate decision latency (plate read → decision) | Within budget set in Phase 0 (suggested ≤ 2 s) |
| NFR-02 | Load test at peak vehicles/minute | No errors, latency within budget |
| NFR-03 | Database restore from backup | Logs and clearances fully recovered |
| NFR-04 | Dependency vulnerability scan | No known high/critical CVEs at release |

## Entry & Exit Criteria

- **Per phase:** the suites mapped to that phase (see BUILD_PLAN.md exit criteria) pass in CI.
- **Release:** all suites pass; §3 decision matrix and §1 HTTPS suite pass with zero exceptions; security review (Phase 6) signed off.

## Traceability

| Business rule (MEMORY.md) | Covering tests |
|---|---|
| Auto-clearance requires approved AND unexpired | AUTO-01…08 |
| Every decision produces a log entry | LOG-01, LOG-02, AUTO-* |
| Logs are append-only | LOG-03, LOG-04 |
| Plate match is candidate ID, not authorization | ANPR-05…07, ANPR-11 |
| HTTPS only | HTTPS-01…04 |
