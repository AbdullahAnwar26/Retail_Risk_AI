# Work Log and Methodology: Retail Banking AI Intelligence Subsystem (Planning Phase)

Audience: the project manager. Purpose: show how the planning work was carried out, in what order, by which tools/models, and what is verified versus not. Living document; updated as work proceeds. Last updated: 2026-11-09.

## 1. Status at a glance

| Item | Status |
|---|---|
| Application code | None written. Planning only, as instructed. |
| BMAD | Not installed (workspace had no BMAD folder). Not installed without approval. Planning files use a generic BMAD-style format until decided. |
| Core planning pack (files 01-12) | Drafted, awaiting review. |
| NYC scope update (files 13-15 + updates to existing files) | Drafted (see section 5); awaiting review. Not yet read end to end by a human or by me. |
| Approval to start implementation | Not given. Work stops at the approval gate. |

## 2. Inputs and instructions followed

1. Manager's research pack: "AI Risk Score Card Global - Complete Pack" (81-page PDF). It is mainly a credit-risk scorecard product; it defines nothing about churn, retention, needs identification or product recommendation.
2. Developer's brief: dual-mode subsystem (standalone and plugin) on one shared intelligence core; 12 deliverables; discovery and planning only; wait for approval.
3. Clarification 1: the main banking platform has no implementation or repo, so the technology stack is open and is evaluated rather than inherited.
4. Clarification 2 (latest): initial market is New York City; federal + NY State + NYC only; no 50-state rulebooks; design for future states through configurable, versioned jurisdiction policies. Not legal advice; legal interpretations need confirmation by the bank's legal and compliance teams.
5. Standing rules: synthetic data only, no invented regulations or eligibility rules, no credit decisions by AI, LLM only for orchestration and explanation, teens (under 18) protected with no commercial profiling without explicit legal approval, no unrelated environment changes.

## 3. Method

Every statement in the planning files carries one label so the reader can see how much to trust it:

| Label | Meaning |
|---|---|
| [DOC p.N] | Supported by the research PDF (page cited) |
| [REC] | Recommendation made in this work |
| [ASSUME] | Assumption that needs checking |
| [UNKNOWN] | Not known from available material |
| [MANAGER] | Needs manager approval |
| [LEGAL-CONFIRM] | Legal/compliance must confirm (NYC update) |

The label "supported by code" is not used because there is no code yet.

## 4. What was done, in order

| Step | Work | Who/what did it | Output |
|---|---|---|---|
| 1 | Checked the workspace for BMAD, repos, manifests, presentation | Main session (Sonnet) | Workspace empty; BMAD not installed; presentation not provided |
| 2 | Recovered the PDF from the session record and saved it as a working copy | Main session | Working copy in the session scratchpad (not in the project folder) |
| 3 | Read the PDF in four page ranges in parallel and tagged each item PLATFORM / CREDIT / RETAIL / SHARED / OPEN | 4 Sonnet agents | Four extraction note files (scratchpad) |
| 4 | Drafted analysis, scope, module matrix, MVP, epics/stories, roadmap | 1 Sonnet agent | Files 01, 02, 06, 09, 10, 11 |
| 5 | Drafted technology report, standalone-vs-plugin architecture, diagrams, data design, security/privacy | 1 Opus agent | Files 03, 04, 05, 07, 08 |
| 6 | Compiled open questions and manager decisions | Main session | File 12 |
| 7 | NYC scope: researched regulations from primary sources on the web | 1 Opus agent (running) | File 13 (pending) |
| 8 | NYC scope: designed jurisdiction policy layer and integration contracts | 1 Opus agent (running) | Files 14, 15 (pending) |
| 9 | Update existing files for NYC scope (02-12) and add a change log | Next step, after 7 and 8 finish | Updated files (pending) |

Model allocation change (2026-11-09, during the final validation pass): at the developer's instruction, all further work runs on Opus only. Work before that point followed the earlier split. Earlier model allocation (as instructed): Sonnet for document analysis, requirements and routine planning; Opus for architecture, compliance and legal-source research; Haiku reserved for mechanical tasks (not needed so far). Steps 3 to 5 and 7 to 8 ran in parallel to save time; parallel work was split by file so agents did not edit the same file.

## 5. NYC update (completed as drafts)

Outcome: file 13 (42-row regulatory matrix: 25 verified against sources, 8 partial, 9 unverified; 24 legal questions; 57 sources), file 14 (policy layer), file 15 (integration contracts), and in-place updates to files 02-12 (each ends with a "Change log (NYC scope update)"). I fixed a module-numbering mismatch in file 05 to match files 06 and 14. Remaining mentions of US/UK in the pack are history, superseded decisions or the PDF's own content; a full read-through for stragglers is still to do. Mermaid diagrams are unrendered.

How it was done: the research agent is consulting official sources (eCFR, NYDFS, NY Senate/AG, NYC Administrative Code, CFPB/FTC and bank regulators). It must mark each matrix row as VERIFIED-SOURCE (official text opened this session) or UNVERIFIED. The policy-layer agent is designing separate Federal / NY State / NYC / future-state policy levels, each policy recording jurisdiction, source, scope, effective date, review date, version, applicability status, required technical controls and approval status, with a fail-safe default when applicability is uncertain.

## 5A. Final validation pass (in progress; this section is updated as each task finishes)

Requested before approval: full-pack review, US/UK sweep, regulatory re-verification, diagram rendering and link checks, a manager-shareable summary, and a BMAD install recommendation. No BMAD install, no application code.

| Task | Model | Output file | Status |
|---|---|---|---|
| Full review of all files; US/UK reference sweep and classification | Opus | 16-validation-review-report.md | Done. 44 defects (2 High, 15 Medium, 27 Low), 10 missing-requirement gaps, 17 leftover lines to correct. No credit-decisioning leak in data, contracts, MVP or epics; one affordability leak in story S8.2 and related wording. Shared core confirmed. |
| Re-check SR 26-2 / OCC 2026-13, Reg B April 2026 amendment, NYC Human Rights Law scope against primary sources | Opus | edits file 13 only | Done. Confirmed SR 26-2 / OCC 2026-13 / FDIC FIL-15-2026 (17 Apr 2026) as supervisory guidance, not binding regulation; generative AI is outside its scope (footnote 3), so a separate LLM governance track is a recommended control, not a requirement. Confirmed Reg B final rule 91 FR 21620, effective 21 Jul 2026, under court challenge. Corrected two errors: NYCHRL 8-107(5)(d) covers housing/commercial-space financing only; TCPA citation is 91 FR 67214. Tally now 27 verified / 6 partial / 9 unverified. |
| Render and validate all Mermaid diagrams; check links and cross-references | Sonnet | 16b-diagram-and-link-validation.md | Done. 13 Mermaid diagrams rendered to SVG with mermaid-cli 11.16.0 (installed in the session scratch folder only, not the project): 13 pass, 0 fail. Layout not visually inspected. 20 reference issues: 1 wrong row ID, 4 missing/stale entries, 7 ID-scheme collisions, 8 style notes. |
| BMAD installation research (read-only; nothing installed) | Sonnet | 17-bmad-installation-recommendation.md | Done. Recommends: install later, pin v6.12.4, trial in a scratch copy first. Several install effects could not be verified from official docs. |
| Apply corrections from the reports, update change logs, N1-N9, approval checklist, manager-shareable summary | Opus only (from here on) | 01-15 split between two Opus agents by file; 00 and summaries by main session | Done. All 44 defects addressed except where noted in file 18; single decision register DEC-01..DEC-43 in file 12; shared ID scheme (PH-, GATE-, PR-, PG-, CI-, DEC-); 13/13 diagrams re-rendered, all pass. Main session cross-checked the two agents' decision references and fixed two unmapped items (08-D12, 15-I8) and D-12 (this file). |
| Approval pass (2026-11-09): give every DEC-01..DEC-43 and BQ-1..BQ-11 item a status under the developer's delegated authority; decide the planning gate | Opus | edits to files 12 (§H), 18 (§1A, checklist), 00 | Done. 15 DECIDED, 27 DECIDED-DEFAULT (manager to ratify at named gates), 12 BLOCKED-EXTERNAL (safe assumptions recorded). Working definitions written for churn, blue-collar, needs/playbook, retention outcomes and demo KPIs. Verdict: conditional GO to BMAD setup, PH-2 and PH-3; GATE-1 provisionally passed. No facts about the bank, approver or legal applicability assumed. |

## 5B. BMAD installation (2026-11-09)

The developer authorised installation ("if BMAD is the best decision then install it"). BMAD was chosen because the original brief requires BMAD conventions and the roadmap is built on its phases.

| Step | Result |
|---|---|
| Backup | `planning/` copied to the session scratch folder before any install |
| Trial in a scratch copy | `npx bmad-method@6.12.4 install --modules bmm --tools claude-code --yes`; npm cache redirected to scratch. Planning files unchanged (hash-checked); 235 files added (1.6 MB) in `_bmad/`, `_bmad-output/`, `.claude/skills/` (29 skills); nothing written to the user's home folder |
| Real install into the project | Same command and options. BMad Core and BMad Method v6.12.4 installed; 29 Claude Code skills in `.claude/skills/`. Planning files hash-identical to the backup |
| Rollback | Delete `_bmad/`, `_bmad-output/`, `.claude/` (no other files touched; no documented uninstall command) |
| Safeguard | Code-producing skills (`bmad-build`, `bmad-build-auto`, `bmad-agent-dev`, `bmad-qa-generate-e2e-tests`) are not to be run until implementation is approved and the manager ratifies the GATE-1 defaults |
| Note | Installer default `project_knowledge` is `docs/`; our pack is in `planning/` (handled in the import step). Installer warned that a dependency (glob 11.1.0) is deprecated; v7 is in preview and not used |
| BMAD import (2026-11-09) | Planning only. `project_knowledge` set to `{project-root}/planning` in `_bmad/custom/config.toml` (team override; `_bmad/config.toml` untouched; verified with `resolve_config.py`). Artefacts in `_bmad-output/planning-artifacts/`: brief, PRD (+addendum), UX `DESIGN.md`/`EXPERIENCE.md`, architecture spine (lint clean), `epics.md` (11 epics, 71 stories, draft), `implementation-readiness.md`, `README.md`; `AGENTS.md` context block at the root. No code; no code-producing skill run |
| Readiness gate | FAIL (independent read-only review). Blockers: demo needs permitted recommendations but pending packs are BLOCK (inherited from files 09/14; MGR/COMP decision needed); undefined "loaded but unapproved" pack state while DEC-11 is open; governance/simulator API endpoints missing from file 15; model approval while DEC-19 is blocked. Epic-level findings fixed. Details: `_bmad-output/planning-artifacts/implementation-readiness.md` |
| Readiness fixes and re-run (2026-11-09) | Planning only (Opus). New decisions in file 12 §H.6: DEC-44 ILLUSTRATIVE demo pack state (watermarked, standalone/synthetic only, refused at load elsewhere), DEC-45 pack lifecycle, DEC-46 governance/simulator endpoints (file 15 §2.1), DEC-47 rule baseline serves and ML SHADOW only, DEC-48 fairness group size 100, DEC-49 credit-building need removed, DEC-50 target market NYC vs UAE vs both (BLOCKED-EXTERNAL). Applied to files 02, 06, 07, 08, 09, 12, 14, 15 and the BMAD brief, PRD, EXPERIENCE, spine (lint clean) and epics. Independent read-only re-run: **CONCERNS** (was FAIL); follow-ups N-1..N-9 fixed; R-4, R-7 low and open. Build still blocked by GATE-1 ratification; jurisdiction pack work blocked by DEC-50 |
| PH-3 contract specification (2026-11-09) | Planning only (Opus). Normative OpenAPI 3.1 contract `contracts/openapi/retail-intelligence-v1.yaml` (42 operations = every v1 resource in file 15 §2 and §2.1; OIDC bearer + OAuth2 scope catalogue; RFC 9457 errors; jurisdiction context, consent purpose, correlation id, idempotency, cursor pagination; `policy.decision_id` + pack versions on customer responses; `not_for_credit_decisioning: true` on every body and as a header; closed recommendation kinds with no approve/decline value; market-neutral codes as reference data), 15 CloudEvents schemas + envelope in `contracts/events/`, `contracts/README.md` and `contracts/redocly.yaml`. Validation: Redocly CLI 1.34.5 run from the session scratch folder (npm cache there, nothing installed in the project): 0 errors, 0 warnings; independent Python `jsonschema` 4.26 check of all schemas, examples, label coverage and forbidden credit terms: clean. Readiness: P-4 contract part done (ERD, port list, role-matrix file still open), R-4 closed (BMAD Stories 2.10, 3.6; planning/10 S19.6, S19.7), R-7 closed (PRD FR-20, FR-23, FR-55, FR-57). No application code, no server/client generation, no code-producing skill, no `.claude/` or settings changes |
| PH-3 ERD, ports and role matrix (2026-11-10) | Planning only (Opus). `contracts/data-model/erd.md` (63 entities, 6 Mermaid diagrams; PII class, `CTL-RETENTION` code, deployment/jurisdiction scoping and erasure method per entity; crypto-shredding via per-subject keys, post-restore DSR re-apply), `contracts/ports.md` (23 use-case ports for all 42 operations; 17 driven ports incl. 5 new: ReferenceData, ResultStore, BulkStaging, SubjectKey, CreditBoundary), `contracts/role-scope-matrix.md` / `.json` (9 roles x 27 scopes x 13 screens x 12 actions x 8 jurisdiction conditions; 393 deny cells, 393 negative tests). Checks from the session scratch folder (nothing installed in the project): mermaid-cli 11.16.0 6/6 PASS; matrix check (Python `jsonschema` 4.26) PASS, scopes identical to the OpenAPI catalogue; ERD/ports/contract cross-check PASS. Mismatches: planning/07 §2 (16 gaps) and 04 §4 (5 ports) superseded by `contracts/`; 15 §5.1 auditor `ri.recommendations.generate` set to deny (read-only rule); spine, EXPERIENCE, epics, contracts README and readiness updated (spine lint 0). GATE-3: ready for review, not signable until GATE-1/GATE-2, a named integration owner, ratification of DEC-03/12/31/38 and a decision on three contract gaps (customer list, stored transactions, portfolio). No application code, no DDL, no `.claude/` or settings changes |
| Two-track scope and delegated decisions (2026-11-10) | Planning only (Opus). Developer scope decision "NYC and London in one, and on the other side the Islamic bank" recorded: DEC-50 DECIDED; DEC-13 and DEC-28 SUPERSEDED; DEC-51..DEC-67 added (file 12 §H.7: deployment profiles, separate US/UK data cells, separate bank-hosted Track B, tighten-only Sharia overlay with 4 controls, Sharia approval state and `sharia_approver`, per-track synthetic catalogues and models, LLM hosting per track, ar/en + RTL, age threshold per jurisdiction (AE [ASSUME 21]), contract `deployment_profile`, GATE-3 gaps, interim role approvers, two demos, UK entity type (BLOCKED-EXTERNAL), Track A human review, Track B telemarketing caps). GATE-1/2/3 batches DECIDED (developer-delegated), manager countersignature recommended. New file 21 (two-track design). **Files 19 (UK, rows U01-U25) and 20 (UAE Islamic, rows E01-E21, SH01-SH14, SDC-01..SDC-18, IQ1-IQ21) written by parallel agents** and referenced by ID (file 21 §4.4 SDC mapping). Propagated to files 02, 03, 07, 08, 09, 11, 14, 15, 18 (change logs), `AGENTS.md`, contracts (OpenAPI 1.1.1: 46 operations incl. customer list, stored transactions, portfolio summary, Sharia approval; matrix 0.2.0: 10 roles, 30 scopes; ports; ERD 66 entities; event envelope extensions) and BMAD artefacts (PRD FR-69..FR-82, spine AD-23..AD-27, 86 stories, EXPERIENCE, DESIGN, brief, README). Checks: Redocly 0/0, contract checks OK, matrix PASS, cross-check PASS, Mermaid 6/6, spine lint 0. Independent readiness re-run: CONCERNS, 11 findings fixed; GATE-3 signed on delegated authority. No application code; no `.claude/` or CLAUDE.md changes |
| Independent two-track validation (2026-11-10) | Planning only (Opus, independent of the authoring session). Findings V-1..V-9 (report: `handoff/two-track-validation-report-2026-11-10.md`). Fixed as design: portfolio aggregate disclosure control AG-1..AG-10 (file 21 §10A), cell isolation checklist and token scheme (§2.1, §2.2), profile signing, keys and fail-closed rules (§3.1), DEC-66 suppression review cannot release an offer (§11A), Sharia SoD aligned with SOD-9 plus per-user checks (files 14, 21), ML per cell in the synthetic MVP (§6); recorded as tightenings in file 12 §H.7.3 (no new DEC) with an authority note on DEC-01/02/12/63. Cross-reference fixes in files 11 and 20; file 15 note. Contract 1.1.2 (descriptions, `min_cell_size` floor 20), role matrix text (cells unchanged), PRD, spine, epics, EXPERIENCE, readiness updated. Checks: Redocly 0/0, contract check OK, matrix PASS, cross-check PASS, spine lint 0. Readiness CONCERNS, no Blocker; V-8 (customer-list scope) and V-9 (SDC-11 catalogue fields) open. No application code; `.claude/`, CLAUDE.md and the manager approval pack untouched |
| Contract 1.2.0: V-8 and V-9 closed (2026-11-10) | Planning and contract only (Opus). V-8: purpose-bound customer directory `GET /v1/customer-directory` (`searchCustomerDirectory`, new staff-only scope `ri.customers.list`, closed `access_purpose` registered per role/client, exact lookup or narrowing filter, page <= 25, query total <= 100, per-user quotas, audit record for every call incl. denials, new error codes); `listCustomers` deprecated and not implemented; new DEC-68 (file 12 §A, §H.7.4, token-enumeration inventory). V-9: `Recommendation.sharia_disclosure` (file 20 SDC-11; approved or illustrative shape; ISLAMIC only; never approved without `SHARIA_APPROVED` and approval refs), ERD `product` disclosure columns and new `message_template` (67 entities). Synced: OpenAPI 1.2.0, recommendation.proposed event, ERD, ports, role matrix 0.3.0 (31 scopes, 680 cells, 514 deny), file 21 §2.3/§4.4/§10, file 15 row, file 20 change-log line, PRD, spine, epics, EXPERIENCE, readiness, handoff reports. Checks: see `handoff/two-track-validation-report-2026-11-10.md` addendum. No code; PH-4 not started; no Sharia or legal ruling |

## 5C. New source documents received (2026-11-09)

Three documents from colleagues and the manager were reviewed (in conversation; not yet saved to the project):
1. "Qatar Retail Lending Policy Summary" (5 pages; personal loans, cards, car loans; figures partly reconstructed from photos; forecast years 2010-2011).
2. "Retail Credit Policy and Procedures Manual - UAE Islamic Bank - Master Edition v1.0" (118 pages; release candidate dated 9 Nov 2026; not in force; built on the AI Risk Score Card's 65 rules).
3. "Retail Credit Policy - UAE Islamic Bank - Plain-Language Guide" (28 pages; explains policy v0.1).

Finding: all three are credit-decisioning material (the credit platform's domain), for UAE and Qatar markets. None defines churn, retention, needs or recommendation requirements. They conflict with the NYC-first instruction, so the target market must be confirmed by the manager before further market-specific work (new decision raised: DEC-50 in file 12 §H.6). Useful inputs for our subsystem: early-warning triggers (UAE manual ch. 27), fair-treatment and vulnerable-customer rules (ch. 31), data-residency rule that the AI runs on the bank's own machines (ch. 33.2), Sharia wording rules (ch. 4), tighten-only rule layering (ch. 1.3, 1.6), replayable decision records (ch. 10.7).

## 6. Key design decisions to date (all proposals pending approval)

- One shared intelligence core, with standalone mode and plugin mode as two adapter sets; the host calls versioned HTTP contracts and never imports our code, so the host's technology stack does not matter.
- Recommended stack: Python and FastAPI, PostgreSQL, React dashboard, OpenAPI contracts, OIDC login, containers; ML starts with rules and simple baselines; LLM optional with fixed-text templates as the default. Platform-wide items are flagged for manager approval.
- Rules before models. AI never approves or declines credit; no credit-decision output exists in the contract.
- Jurisdictions become configuration (policy packs), so new states or countries are added without changing the core. This replaces the earlier "separate US and UK installs" framing; UK and others become future expansion [MANAGER confirm].

## 7. Limits and honesty notes (read before presenting)

- The PDF could not be rendered page by page because the PDF image tool (poppler) is not installed. Text was extracted with pdftotext; some tables and charts were garbled and are marked open.
- The Sonnet drafting agent worked from the extraction notes, not the PDF directly.
- Mermaid diagrams in file 05 have not been rendered or checked in a viewer.
- The PDF has internal inconsistencies (15 vs 12 steps, $150 vs $45, GBP750 vs GBP1,000, page order) listed in files 01 and 12.
- No source for prd.md (FR-39, FR-50/51, FR-79..86) or the presentation was available.
- Legal content is research for legal/compliance to confirm, not a legal conclusion. Applicability of each law depends on the bank's charter, products and activities, which are unknown.

## 8. File index

00 this log | 01 research analysis | 02 scope | 03 technology | 04 standalone vs plugin | 05 architecture diagrams | 06 module matrix | 07 data design | 08 security/privacy | 09 MVP | 10 epics/stories | 11 roadmap (source of PH-/GATE- IDs) | 12 decision register and open questions (source of DEC- IDs) | 13 NYC regulatory matrix | 14 policy layer | 15 integration contracts | 16 validation review report | 16b diagram and link validation | 17 BMAD installation recommendation | 18 manager summary (shareable)

## 9. Change log

| Date | Change |
|---|---|
| 2026-11-10 | Contract 1.2.0 (section 5B): V-8 and V-9 closed as planning/contract design; DEC-68 added (file 12 §H.7.4); contract, events, ERD, ports, role matrix, files 15/20/21 and BMAD artefacts synced; readiness re-run limited to these changes. No application code. |
| 2026-11-10 | Independent two-track validation (section 5B): V-1..V-7 fixed as design tightenings (file 21 §2.1, §2.2, §3.1, §6, §10A, §11A; file 12 §H.7.3; files 11, 14, 15, 20; contract 1.1.2; BMAD artefacts); V-8, V-9 open; readiness CONCERNS, no Blocker. GATE-1/2/3 remain developer-ratified, manager countersignature recommended. No application code. |
| 2026-11-10 | Two-track scope (section 5B): DEC-50 decided, DEC-51..DEC-67 added, GATE-1/2/3 developer-ratified (manager countersignature recommended); file 21 added; files 19 and 20 written in parallel; planning 02-18, contracts and BMAD artefacts updated; readiness CONCERNS with all findings fixed. No application code. |
| 2026-11-10 | PH-3 ERD, port list and role-scope matrix added in `contracts/` (section 5B); readiness P-4 complete pending GATE-3 sign-off. BMAD spine, EXPERIENCE, epics, README and readiness updated; planning files 01-18 unchanged. No application code. |
| 2026-11-09 | PH-3 contract specification: normative OpenAPI 3.1 + CloudEvents schemas in `contracts/` (lint clean); readiness P-4 (contract part), R-4 and R-7 closed (section 5B). Files 10, 15, BMAD epics, PRD, README and readiness updated. No application code; no `.claude/` or settings changes. |
| 2026-11-09 | Readiness fixes: DEC-44..DEC-50 added (file 12 §H.6) and applied across the pack and BMAD artefacts; readiness re-run CONCERNS (section 5B). No application code; no `.claude/` or settings changes. |
| 2026-11-09 | BMAD import: planning artefacts created in `_bmad-output/planning-artifacts/` from files 00-18 and 12 §H; `project_knowledge` -> `planning/`; readiness FAIL recorded with 4 open blockers (section 5B). No application code. |
| 2026-11-09 | Approval pass: file 12 §H records a status for every DEC and BQ item; file 18 §1A gives the outcome; conditional GO to BMAD setup, PH-2 and PH-3. Nothing installed; no application code. |
| 2026-11-09 | Files 01-12 drafted (global US/UK framing). |
| 2026-11-09 | Final validation pass completed (section 5A): files 16, 16b, 17, 18 added; files 01-15 corrected; awaiting manager approval. Nothing installed; no application code. |
| 2026-11-09 | Final validation pass started (section 5A). File 17 (BMAD recommendation) written; nothing installed. |
| 2026-11-09 | NYC scope clarification received. Files 13-15 written; files 02-12 updated; file 05 module numbering aligned. All awaiting manager review. |
