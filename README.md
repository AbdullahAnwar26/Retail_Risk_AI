# Retail Banking AI Intelligence Subsystem

Planning and contract repository for an advisory retail-banking intelligence subsystem (customer and transaction intelligence, segmentation, financial-needs detection, churn, product suggestions and retention). It runs standalone or as a plugin to a host banking platform, on one shared core.

**Status: planning only.** There is no application code yet. Implementation waits for explicit approval (see `planning/12-open-questions-and-decisions.md` §H). All data is synthetic. Nothing here is legal or Sharia approval.

## Scope

- **Track A, conventional:** New York City (US federal, NY State, NYC) and London (UK), with separate US and UK data cells.
- **Track B, Islamic bank:** assumed UAE onshore, a separate bank-hosted deployment that shares no data or models with Track A.

## How to read this repository

The commit history follows the order the work was done:

1. Discovery and research analysis
2. Scope and technology evaluation
3. Architecture (standalone vs plugin, system design, modules)
4. Data, security, privacy and compliance design
5. MVP, epics and stories, roadmap
6. Decision register and NYC regulatory matrix with the jurisdiction policy layer
7. Integration contracts (prose)
8. Validation reviews
9. BMAD method setup and planning artefacts (PRD, architecture spine, UX, epics, readiness)
10. Manager summary and work log
11. Two-track design: UK/London and UAE Islamic bank matrices
12. Machine-readable contracts (OpenAPI 3.1, CloudEvents, data model, role/scope matrix)
13. Validation report, status report and manager approval pack

## Where to start

| You want | Open |
|---|---|
| The story of how the work was done | `planning/00-work-log-and-methodology.md` |
| A short summary for managers | `planning/18-manager-summary.md` and `handoff/manager-approval-pack-2026-11-10.pdf` |
| Every decision and its status | `planning/12-open-questions-and-decisions.md` (§H is authoritative) |
| The two-track design | `planning/21-two-track-deployment-design.md` |
| The API contract | `contracts/openapi/retail-intelligence-v1.yaml` |
| BMAD planning artefacts | `_bmad-output/planning-artifacts/README.md` |
