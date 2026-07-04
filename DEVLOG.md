# Development Log

Chronological record of development sessions and milestones. **Newest entries first.** Each entry should note what was done, why, and anything the next session needs to know.

---

## 2026-07-04 — Project inception & documentation

**Done:**
- Created the initial project documentation set:
  - `README.md` — project overview and feature summary
  - `MEMORY.md` — persistent project memory (scope, business rules, constraints, open questions)
  - `BUILD_PLAN.md` — phased build roadmap (Phase 0 → 6)
  - `TEST_PLAN.md` — test strategy and case catalogue
  - `DEVLOG.md` — this file

**Scope established:**
- HTTPS-based web application for entry logging at aviation premises.
- Three core flows: new visitor registration, automatic clearance for previously approved/unexpired visitors, and camera-based ANPR to recognize visitors by vehicle number plate.

**Key rules locked in (see MEMORY.md):**
- Auto-clearance requires *approved AND unexpired* — no exceptions.
- Every entry decision is logged; logs are append-only.
- ANPR plate reads are candidate identifications only; the clearance record authorizes entry.

**Next up:**
- Resolve Phase 0 decisions in BUILD_PLAN.md: tech stack, ANPR engine, database, deployment target.
- Answer the open questions in MEMORY.md (clearance expiry policy, regulatory retention requirements, kiosk vs. guard-operated registration).
- Scaffold the application skeleton (Phase 1).

---

<!-- Template for future entries:

## YYYY-MM-DD — Short title

**Done:**
- ...

**Decisions:**
- ...

**Problems / blockers:**
- ...

**Next up:**
- ...

-->
