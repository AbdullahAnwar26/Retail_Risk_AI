---
title: PRD addendum - Retail Banking AI Intelligence Subsystem
created: 2026-11-09
updated: 2026-11-10
---

# PRD Addendum

This addendum holds technical detail and rationale that the PRD deliberately leaves out. The planning pack remains the full record; this file only points to it.

| Topic | Where it lives | Note |
|---|---|---|
| Technology choices and rationale (Python 3.12, FastAPI, PostgreSQL 16 per deployment, DuckDB/Parquet batch, scikit-learn/Fairlearn, React+TS+Vite, OIDC, OCI image, OpenTelemetry) | planning/03 §3-§5 | DEC-05 is DECIDED-DEFAULT; the manager ratifies it at GATE-2. Platform-wide items (IdP, event bus, LLM provider, orchestrator, KMS) are listed in planning/03 §5 |
| Mechanism evaluation: library vs REST vs events vs batch | planning/04 §2 | REST is the primary mechanism; the event transport is deferred |
| Ports-and-adapters source tree and port list | planning/04 §4 | Seed for the architecture spine |
| Full entity list, data-quality rules, synthetic approach, erasure method | planning/07 §2, §4.1, §7, §8.2 | |
| Control catalogue, combining rules, policy record schema, YAML example, approval workflow, fail-safe table | planning/14 §3-§10 | |
| API surface, envelope JSON, error model, role-to-scope matrix, events, credit-platform candidates CI-1..CI-9 | planning/15 §2-§11 | |
| Regulatory matrix rows and legal questions Q1-Q24 | planning/13 §3, §6 | All [LEGAL-CONFIRM] |
| Rejected alternatives | planning/04 §2 (library embedding rejected); planning/03 §3.13 (OPA/Cedar deferred, DEC-29); planning/12 §H.2 BQ-2 (shift-pattern inference of blue-collar rejected) | |
| Validation history (44 defects, 10 missing requirements) | planning/16, planning/16b | |
| Two-track deployment design: deployment profile sketch, cells, Sharia overlay controls and approval state, per-track catalogues, models, LLM hosting, shared-vs-separate table, demo plan | planning/21; planning/12 §H.7 (DEC-50..DEC-64) | UK and UAE/Sharia applicability in planning/19 and planning/20 (written 2026-11-10) |
