# BMAD Planning Artefacts - Retail_Risk_AI

> **Scope (DEC-50, DECIDED by the developer 2026-11-10; `planning/21-two-track-deployment-design.md`).** Two separate tracks on one shared core and contract: **Track A "Conventional"** = NYC + London (one profile, separate `A-US` and `A-UK` data cells); **Track B "Islamic"** = an Islamic bank, assumed UAE onshore [ASSUMPTION], separate bank-hosted deployment with a tighten-only Sharia overlay. The GATE-1, GATE-2 and GATE-3 DECIDED-DEFAULT batches are DECIDED (developer-delegated), manager countersignature recommended (planning/12 §H.7). UK applicability (`planning/19-uk-london-regulatory-applicability-matrix.md`) and UAE + Sharia applicability (`planning/20-islamic-bank-uae-regulatory-and-sharia-matrix.md`) were written on 2026-11-10; GB and AE pack content waits for them (P-10, P-11). No application code until the developer gives an explicit implementation instruction.

Produced 2026-11-09 with BMAD v6.12.4 (core + bmm) by importing the approved-with-conditions planning pack in `planning/` (00-18). The import was planning only: no application code, no infrastructure, and no code-producing skills were run. Each workflow was facilitated in fast path, with every answer taken from the pack and the decisions in planning/12 §H. Gaps are recorded as open questions, not invented.

## Artefact map

| BMAD artefact (skill) | Path | Main planning/ sources |
|---|---|---|
| Product brief (`bmad-product-brief`) | `briefs/brief-Retail_Risk_AI-2026-11-09/brief.md` + `.memlog.md` | 01, 02 §1-§8, 09 §1-§4, 12 §H, 18, 21 |
| PRD (`bmad-prd`) | `prds/prd-Retail_Risk_AI-2026-11-09/prd.md`, `addendum.md`, `.memlog.md` | 02, 05 §9, 06, 07, 08 §2, 09, 10, 12 §H (DEC/BQ; §H.7 two-track), 13 (by row ID), 14, 15, 21 (FR-69..FR-80) |
| UX spine pair (`bmad-ux`) | `ux-designs/ux-Retail_Risk_AI-2026-11-09/DESIGN.md`, `EXPERIENCE.md`, `.memlog.md` | 04 §5.1, 07 §2.1, 08 SEC-19, 09 §5 §7, 10 E10, 14 §10 §13, 15 §5.1, DEC-41 |
| Architecture spine (`bmad-architecture`) | `architecture/architecture-Retail_Risk_AI-2026-11-09/ARCHITECTURE-SPINE.md`, `.memlog.md` (`lint_spine.py`: 0 findings; AD-1..AD-27) | 03 §4-§6, 04, 05, 06, 07 §1 §5 §8, 08, 14, 15, 21, 12 §H |
| Epics and stories (`bmad-create-epics-and-stories`) | `epics.md` (11 epics, 86 stories, status draft) | 10 (all 93 stories, re-homed; map below), 11, 12 §H |
| Normative contract (PH-3, P-4) | `../../contracts/` (OpenAPI 3.1 `openapi/retail-intelligence-v1.yaml`, CloudEvents schemas `events/`, `README.md`; Redocly lint clean) | 15 (incl. §2.1), 14 §3 §7 §8.4, 07 §2 |
| Readiness gate (`bmad-sprint-planning`, readiness intent) | `implementation-readiness.md` (verdict **CONCERNS** after re-run; first run FAIL) | all of the above + 12 §H, 14 §8 §10, 15 §2 |
| Project context (`bmad-project-context`, setup) | `../../AGENTS.md` (project root, managed block) | 00 §5B, 12 §H.4, 17 §5 |
| Project knowledge setting | `_bmad/custom/config.toml` → `[modules.bmm] project_knowledge = "{project-root}/planning"` | 00 §5B note; installer comments in `_bmad/config.toml` |

Every artefact section cites its source as "Source: planning/NN §x".

**Why `project_knowledge` points at `planning/`:** the installer default `docs/` does not exist, and the pack lives in `planning/`. The documented team override file is the simplest correct option. It is never touched by the installer, and the resolver output was checked with `resolve_config.py`. Copying the pack into `docs/` would have created a second, drifting copy.

## planning/10 → BMAD story map

| planning/10 | BMAD |
|---|---|
| E1 S1.1-S1.4, E2 S2.1-S2.4, E3 S3.1-S3.4 (design documents), S19.1 (document) | Non-development prerequisites P-1..P-4 |
| E13 S13.1, S13.3 / S13.2 / S13.4 | P-5 / Story 2.9 / Story 7.3 |
| E4 S4.1-S4.4 | Stories 1.1-1.4 |
| E5 S5.1-S5.5 | Stories 3.1-3.5 |
| E6 S6.1, S6.2, S6.3, S6.4 | Stories 4.1, 4.2, 4.3, 4.4 |
| E7 S7.1, S7.2, S7.3a, S7.3b, S7.4 | Stories 5.2, 5.3 + 5.4, 5.1, 5.6, 5.5 |
| E8 S8.1-S8.6 | Stories 6.1, 6.2 (+ 6.3), 6.4, 6.5, 6.6, 6.7 |
| E9 S9.1-S9.4 | Stories 8.1-8.4 |
| E10 S10.1, S10.2, S10.3, S10.4, S10.5 | Stories 9.1 + 9.2, 9.4, 9.6, 9.7, 9.8 |
| E11 S11.1, S11.2 (= S19.3), S11.3, S11.4 | Stories 10.1, 10.5, 10.2, P-8 |
| E12 S12.1, S12.2, S12.3, S12.4, S12.5, S12.6, S12.7, S12.8 | Stories 2.7 + 11.5, P-8, 11.5, 11.5, 11.1, 11.2, 11.3, 11.4 |
| E14 S14.1-S14.6 / S14.7 | Stories 2.1, 2.2, 2.3, 2.6, 2.4, 2.6 / 10.4 |
| E15 S15.1, S15.2, S15.3, S15.4, S15.5 | Stories 2.2, 7.1, 7.2, 7.3, 7.8 |
| E16 S16.1-S16.3, S16.4, S16.5 | Stories 7.4, 2.5, 2.5 |
| E17 S17.1, S17.2 + S17.3, S17.4 | Stories 7.6, 7.5, 7.7 |
| E18 S18.1, S18.2, S18.3, S18.4 | Stories 4.3 + 2.5, 2.5, 4.3, P-7 |
| E19 S19.2, S19.4, S19.5, S19.6, S19.7 | Stories 2.8, 10.3 + 10.7, 10.6, 2.10, 3.6 |
| New in BMAD (no planning/10 equivalent) | 6.3 (credit referral, from 14 §13.1), 6.8 (consent withdrawal, 08 PRV-03), 6.9 (composite evaluate, 15 §2), 9.3, 9.5, 9.9 (UX screens), 11.6 (NFR baselines) |
| Two-track scope (planning/21, 12 §H.7) | 1.5 (per-cell seeds and catalogues), 2.11 (deployment profile), 2.12 (data cells), 2.13 (institution overlay), 5.7 (per-track registry), 6.10 (DEC-62 read endpoints), 7.9 (Sharia approval), 7.10 (UK packs), 7.11 (AE packs and overlay content), 8.5 (Arabic/English templates), 9.10 (Track A demo), 9.11 (Track B demo), 11.7 (isolation suite) |

## Preserved constraints

Each constraint is in the PRD and the spine, and in the stories where relevant:
- **Two tracks, jurisdiction packs per track, tighten-only institution overlay:** PRD §4.7-§4.8, §4.13; AD-3, AD-5, AD-23..AD-27.
- **Single shared core:** AD-1, AD-2.
- **No credit decisioning:** FR-21, FR-55; AD-6.
- **Minors two-tier protection:** FR-12; AD-7.
- **Synthetic data only:** AD-19.
- **`[LEGAL-CONFIRM]` items:** carried throughout.
- **DECIDED-DEFAULT items:** GATE-1/2/3 batches ratified by developer delegation (PRD §8 OQ-17; manager countersignature recommended); later-gate items still DECIDED-DEFAULT.

## Open items

1. **Former readiness blockers, now resolved in the plan (planning/12 §H.6):**
   - **R-1 / OQ-18** → DEC-44: `ILLUSTRATIVE` demo pack state. It works only in standalone with `demo=true` on synthetic data, every output is watermarked "illustrative - not legal advice - not for real customers", and plugin and production refuse it at load.
   - **R-2 / OQ-19** → DEC-45: pack lifecycle with `VALIDATED` as the "loaded but unapproved" state (simulator only), plus the runtime bundle, `/ready` and a restrictive-only L0.
   - **R-3** → DEC-46: governance, simulator, decision-log and model-registry endpoints, in planning/15 §2.1.
   - **R-6 / OQ-20** → DEC-47: the rule baseline serves; ML runs in SHADOW only until DEC-19 is resolved.
   - The remaining CONCERNS are listed in `implementation-readiness.md`.
2. **Ratification status (planning/12 §H.7.2):**
   - DECIDED (developer-delegated, 2026-11-10; manager countersignature recommended): GATE-1 DEC-01, 02, 04, 09, 43, 44 (COMP still signs off the watermark before GATE-3A), 49, BQ-1..BQ-7, BQ-11; GATE-2 DEC-05, 08 (stack version pins still to do); GATE-3 DEC-03, 12, 31, 38 and the contract gaps (DEC-62). DEC-13, DEC-28 superseded.
   - Still DECIDED-DEFAULT: GATE-7 DEC-47 (MRM); GATE-8 DEC-37, 40, BQ-8; GATE-11 DEC-27; GATE-12 DEC-36.
3. **BLOCKED-EXTERNAL:** DEC-65 (UK operating entity type, planning/19 UQ1), DEC-07, 10 (per track), 11 (named legal and Sharia approvers; interim roles in DEC-63), 18, 19, 20, 21, 22, 23, 25, 30, 33 (planning/12 §H.7). DEC-50 resolved (two tracks). Per-track content gates: planning/19 (P-10), planning/20 (P-11), both written 2026-11-10.
4. **UX:** UX-OQ-1..UX-OQ-6. DESIGN.md tokens are intentionally empty (no brand, palette or UI system in the pack), and key-screen mocks were not rendered.
5. **Pack inconsistencies found during the import:**
   - Fairness minimum group size: resolved as 100 per cell (DEC-48, OQ-14).
   - "Credit-building need": removed from v1 (DEC-49, OQ-15); planning/02 and 06 updated.
   - File 10 S6.2 predates the BQ-2 `VARIABLE_INCOME` segment.
   - BQ-3 says "from date of birth" while data stores birth year-month (low).
6. **Missing sources:** MS1-MS4 (the platform prd.md and others).

## BMAD tooling issues and workflow deviations

- **Glob mismatch in BMAD v6.12.4:** `bmad-create-epics-and-stories` step 1 searches `{planning_artifacts}/*prd*.md` and `*architecture*.md`, but `bmad-prd` and `bmad-architecture` write into run subfolders (`prds/prd-*/prd.md`, `architecture/architecture-*/ARCHITECTURE-SPINE.md`). Their paths were therefore listed explicitly in the `epics.md` `inputDocuments`. No shim files were created.
- **Workflow steps skipped or substituted:**
  - Web research, the reviewer-gate subagent lenses for brief, PRD, UX and spine, and stack-version web verification were not run.
  - The only automated checks were `lint_spine.py` and an FR/UX-DR coverage script.
  - The interactive [C] approvals were not given by a human. Epics stay `draft`, and human sign-off notes (planning/17 §5 step 4) are pending.
- **Readiness fixes (2026-11-09):** DEC-44..DEC-50 were applied to planning/02, 06, 07, 08, 09, 12, 14 and 15 and to the brief, PRD, EXPERIENCE, spine (`lint_spine.py`: 0 findings) and epics. The gate was then re-run read-only by an independent subagent, and its CONCERNS-level follow-ups N-1..N-9 were fixed where they needed no MGR/LEG/MRM decision.
- **AGENTS.md is not loaded by Claude Code automatically.** Adding a `CLAUDE.md` containing `@AGENTS.md`, and a deny rule for the build skills in `.claude/settings.json` (planning/17 §5 step 5), is recommended. Neither was done; both need the user's approval.

## Change log
- 2026-11-09: import (initial).
- 2026-11-09 (readiness fixes): market note (DEC-50) added. Open items updated for DEC-44..DEC-49. Readiness verdict is now CONCERNS (was FAIL). No application code; no `.claude/` or settings changes.
- 2026-11-09 (PH-3 contract specification): normative contract in `contracts/` added to the artefact map; Stories 2.10 and 3.6 (readiness R-4) mapped from planning/10 S19.6, S19.7; counts 73 / 93. No application code; no `.claude/` or settings changes.
- 2026-11-10 (two-track scope): market note replaced (DEC-50 resolved; planning/21); artefact map, story map (13 new stories, 86 total), preserved constraints and open items updated for 12 §H.7 (developer-delegated GATE-1/2/3 ratification). No application code; no `.claude/` or settings changes.
- 2026-11-10 (alignment with planning/19 and planning/20, now written): 19/20 cited as written; DEC-65 added to the blocked list; PRD FR-81 (DEC-66, Track A human review) and FR-82 (DEC-67, Track B contact caps), OQ-24/OQ-25; AD-27 Track B data stays in the UAE (IQ3); epics Stories 7.9-7.11 ACs extended (86 stories unchanged).
