---
status: draft
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-Retail_Risk_AI-2026-11-09/prd.md
  - _bmad-output/planning-artifacts/prds/prd-Retail_Risk_AI-2026-11-09/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-Retail_Risk_AI-2026-11-09/ARCHITECTURE-SPINE.md
  - _bmad-output/planning-artifacts/ux-designs/ux-Retail_Risk_AI-2026-11-09/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-Retail_Risk_AI-2026-11-09/EXPERIENCE.md
  - planning/10-bmad-epics-and-stories.md
  - planning/11-implementation-roadmap.md
  - planning/12-open-questions-and-decisions.md
  - planning/21-two-track-deployment-design.md
validation: "Two-track update 2026-11-10 (planning/21, 12 §H.7): 13 stories added (1.5, 2.11-2.13, 5.7, 6.10, 7.9-7.11, 8.5, 9.10, 9.11, 11.7), FR69-FR82 (FR81-FR82 added in the 19/20 alignment), NFR14-NFR15, UX-DR18-UX-DR20 covered; 86 stories; no forward dependency introduced. Earlier: Step-4 checks: FR and UX-DR coverage complete (73 stories; 2.10 and 3.6 added for readiness R-4); forward dependencies flagged by the readiness gate were fixed (2.7, 4.2, 4.4, 4.3/5.4, 7.2/10.7, 7.5); Stories 6.9, 9.9 and 11.6 were added. Readiness blockers R-1, R-2, R-3, R-6 resolved in the plan by DEC-44..DEC-47 (planning/12 §H.6) and applied to stories 2.2, 2.4, 2.5, 2.8, 4.4, 5.1, 5.2, 5.4, 7.1, 7.2, 7.4, 7.6, 7.7, 9.2, 9.4, 9.5, 9.8, 9.9, 10.5, 11.5. Status stays draft: GATE-1 is ratified by developer delegation (manager countersignature recommended), DEC-50 is resolved (two tracks), but no explicit implementation instruction has been given."
---

# Retail_Risk_AI - Epic Breakdown

## Overview

This document breaks down the epics and stories for Retail_Risk_AI. It decomposes the requirements from the PRD, the UX design contract (DESIGN.md + EXPERIENCE.md) and the architecture spine into implementable stories.

It re-homes planning/10 (19 epics, 91 stories, generic format) into BMAD format:
- Stories are grouped by user value.
- Stories are ordered so that none depends on a later story.
- Planning, design and legal items from planning/10 that produce no code (E1, E2, E3 design documents, E13 legal intake, S11.4, S12.2, S18.4) move to the **Non-development prerequisites** list under Additional Requirements.

The full planning/10 → BMAD story map is in `README.md`. Each story cites its planning/10 source as "Source: Sx.y".

**No story may be built until** implementation is explicitly approved by the developer (planning/12 §H.4 condition 5). The GATE-1, GATE-2 and GATE-3 DECIDED-DEFAULT batches were ratified by developer delegation on 2026-11-10 (12 §H.7.2; manager countersignature recommended). Code-producing BMAD skills stay off.

**Scope (DEC-50, 2026-11-10):** two tracks on one core. Track A "Conventional" = NYC (`A-US` cell) + London (`A-UK` cell); Track B "Islamic" = an Islamic bank, UAE onshore [ASSUMPTION], separate bank-hosted deployment (`B-AE`). Stories from Epic 1-11 are market-neutral unless they name a jurisdiction; NYC-named stories apply to the `A-US` cell, and the two-track stories below extend them.

## Requirements Inventory

### Functional Requirements

FR1: The tester can generate, from a seed, NYC adult customers and accounts plus jurisdiction and age test records (`US-NY-NYC`, `US-CA`, missing, `UNKNOWN`, minors 16-17, age unknown). Output is byte-reproducible and every row has `is_synthetic=true`.
FR2: The tester can generate transactions with configurable edge-case rates (duplicates, pending, refunds, reversals, transfers, multi-currency, irregular income) and ground-truth labels.
FR3: The tester can load a SYNTHETIC NYC catalogue with `jurisdictions_offered`, plus synthetic consent and contact-preference records. Purpose codes are validated against config.
FR4: The data scientist can generate time-structured churn and need scenarios and a validation report (distribution sanity, coverage, leakage audit, balance, reproducibility hash).
FR5: Transactions are normalised and de-duplicated idempotently, with pending items handled.
FR6: Internal transfers are paired and refunds and reversals linked. Currencies are never mixed in sums.
FR7: Transactions are categorised with confidence, method and rule trace. The taxonomy has no forbidden-inference labels.
FR8: Income, expenses, savings rate and recurring items are detected with a documented calculation.
FR9: Data-quality rules DQ-01..DQ-10 return `DATA_QUALITY_INSUFFICIENT` per module on a blocking breach. Thresholds are config.
FR10: Staff see a reproducible customer profile with health indicators and a feature dictionary. Excluded attributes are stripped before storage.
FR11: A customer can hold several segments with reasons (Gen Z 18-29, blue-collar from consented occupation only, `VARIABLE_INCOME`). Segments are never used as ranking or churn features until DEC-21 and DEC-23 are resolved.
FR12: Minors and unknown-age customers follow the two-tier block list in every module. Under-13 records are rejected or quarantined at ingest. Every block is audited.
FR13: Compliance sees a fairness view (selection rate, score distribution) computed only on synthetic group labels held in a restricted store, with a configurable minimum group size (100 per cell, DEC-48).
FR14: Staff see BQ-6 rule-based needs with 6-month evidence and a rule version. There is no affordability or debt-to-income calculation, and no credit-building need in v1 (DEC-49).
FR15: Churn has a rules baseline and a transparent ML model evaluated on a time split with the BQ-1 label. The rule baseline serves in v1. ML runs in SHADOW only and becomes eligible for MRM review only if it beats the baseline by the configured margin (DEC-47).
FR16: A model registry with `kind` RULESET (DRAFT/ACTIVE/RETIRED) or ML_MODEL (DRAFT/SHADOW/APPROVED/RETIRED) is in place. Only RULESET ACTIVE or ML_MODEL APPROVED serve; ML approval is refused while DEC-19 is open. Each feature list is checked against the exclusion lists.
FR17: A drift indicator is computed per model and batch.
FR18: Each prediction returns stable top-driver reason codes.
FR19: Recommendations exist only for active catalogue products offered in the customer's resolved jurisdiction.
FR20: Every candidate passes consent, PDP jurisdiction, minor, suppression (opt-out, pause-offers stub, vulnerability) and channel gates. The gate trace is stored and suppressed rows are kept.
FR21: Credit appears only as `CREDIT_REFERRAL`, never for under-18s. The forbidden-term linter guards customer-outcome schemas.
FR22: Ranked suggestions carry reason codes from the versioned library, with support-first ordering when financial-stress signals exist.
FR23: Retention suggestions come from the fixed playbook with a priority list. There is no automatic contact, and contact limits and quiet hours apply.
FR24: Items flagged `requires_human_review` sit in a queue (QUEUED → IN_REVIEW → CLEARED_TO_SHOW / REJECTED / ESCALATED; EXPIRED / CANCELLED) with an SLA from config. They are never auto-released.
FR25: Retention outcomes (NOT_ACTIONED / CONTACTED / RETAINED / CLOSED / UNKNOWN) are recorded append-only with a fixed reason, time and source.
FR26: Jurisdictions are held as data and resolved from residence and booking. Missing or conflicting values resolve to UNKNOWN. No jurisdiction literals appear outside `core/policy`.
FR27: There is a policy record model and a versioned control catalogue with schema-validated parameters.
FR28: The PDP resolves deterministically. The most restrictive outcome wins, and INDETERMINATE becomes DENY with a conflict record.
FR29: Every module calls the PDP before and after processing. Results need a `decision_id`, and unknown obligations mean DENY.
FR30: Every fail-safe row in planning/14 §10 has a test (pending, expired `review_due`, unknown or unsupported jurisdiction, unknown age, unknown consent, bad signature, PDP error).
FR31: There is a decision log with replay that gives an identical outcome. It holds policy IDs and statuses only.
FR32: A host overlay can only tighten. Loosening parts are rejected with `policy.overlay.rejected`.
FR33: The L0 baseline pack is locked (no credit decisions, LLM guardrail, no sensitive inference, minors Tier 1).
FR34: Packs are versioned, effective-dated YAML with `matrix_ref` and citations. Manifests are signed and invalid signatures are refused at load.
FR35: There is a maker-checker workflow (policy DRAFT → LEGAL_REVIEW → APPROVED → RETIRED; pack DRAFT → VALIDATED → IN_LEGAL_REVIEW → APPROVED → ACTIVE → RETIRED, demo branch VALIDATED → ILLUSTRATIVE). Only `policy_approver` sets `applicability_status`.
FR36: Activation to ACTIVE needs `policy_release` plus recorded MGR/LEG sign-off; activating an unapproved pack is refused. ILLUSTRATIVE activation works only in a standalone demo deployment (`demo=true`, synthetic data), and every output is then watermarked (DEC-44).
FR37: Once `review_due` passes, the policy is treated as PENDING_LEGAL_REVIEW.
FR38: Illustrative federal, NY State and NYC packs from candidate file 13 rows stay PENDING_LEGAL_REVIEW.
FR39: Exclusion, forbidden-inference, proxy and protection-only lists are versioned pack data, with the DEC-23 starter list marked pending.
FR40: A policy simulator runs golden cases and a synthetic population with a diff against the current runtime bundle. Its report is attached when the pack is submitted to IN_LEGAL_REVIEW.
FR41: Golden cases per jurisdiction and property tests (monotonicity, determinism, fail-safe, locked baseline, no credit permission) exist. A failing case blocks the pack build.
FR42: An add-a-jurisdiction rehearsal changes no core code.
FR43: Each file 13 row has an applicability decision record (APPLICABLE / NOT_APPLICABLE / PENDING_LEGAL_REVIEW) that only the legal approver can set.
FR44: The orchestrator uses only a typed, read-only, allow-listed tool layer.
FR45: Explanations use templates. Every number and fact is validated against tool results, with template fallback.
FR46: The assistant refuses credit decisions, account changes, unknown products and legal conclusions (422 `OUT_OF_SCOPE_REQUEST` plus handoff).
FR47: `LLMPort` exists with only the template adapter enabled.
FR48: There is a dashboard customer view showing all module outputs with `decision_id`, jurisdiction context and versions.
FR49: There is an audit and controls view with recommendation replay, gated by role.
FR50: There is a read-only assistant panel using only M9 tools.
FR51: There is a portfolio view of aggregates only (Could).
FR52: There is a scripted demo pack matching planning/09 §5 and §7, run from a clean start.
FR53: There are policy views: a pack viewer (pending shown openly), decision-log view, test-jurisdiction selector and simulator page.
FR54: Contract v1 is published (OpenAPI 3.1, JSON Schema, examples, changelog) along with `GET /v1/capabilities`.
FR55: There is a canonical envelope (jurisdiction context and purpose in; `decision_id`, versions, `permitted_use`, `not_for_credit_decisioning` out), plus an error model and idempotency.
FR56: Host identity (OIDC, `ri_deployment`, `ri_jurisdictions`) and consent are taken through ports. A claim mismatch returns 403 `JURISDICTION_NOT_PERMITTED`.
FR57: A host simulator drives the core through contract v1 and gets results identical to standalone.
FR58: There are recommendation and retention-action outcome endpoints, including plugin human-review results.
FR59: Outbound CloudEvents go through a transactional outbox with no broker (Could).
FR60: Only tokens and the minimum fields cross the boundary. ZIP and fine location never cross in v1.
FR61: There is a deny-by-default role x scope x screen matrix with separation of duties and one negative test per cell.
FR62: Audit is append-only and hash-chained for reads, evaluations, config changes, pack activations, decisions, model status and LLM interactions.
FR63: Consent is enforced per source and purpose. Withdrawal stops use, triggers recompute and is audited.
FR64: Retention lifecycle hooks are driven by `CTL-RETENTION`, with no hard-coded values and purge in report-only mode.
FR65: DSR erasure uses crypto-shredding of per-subject keys. The hash chain still verifies afterwards.
FR66: Encrypted backup and restore, with a restore test that re-verifies hash chains and pack signatures.
FR67: There is a versioned reason-code library (DEV drafts, COMP approves) and operator runbooks.
FR68: A technical evaluation report with the BQ-11 KPIs and synthetic-data limits is generated from the seed, alongside a regression suite.
FR69: Each deployment loads one signed, read-only deployment profile; the bootstrap refuses mismatched or invalid profiles, packs, catalogues, models, templates, regions and LLM hosting (DEC-51, DEC-61).
FR70: Track A runs two data cells (`A-US`, `A-UK`) with separate databases and regions; personal data never crosses cells; cross-cell residence/booking resolves to UNKNOWN; only aggregates combine (DEC-52).
FR71: Track B is a separate, bank-hosted deployment that never shares data, models, keys, tokens or logs with Track A, proven by isolation tests (DEC-53).
FR72: The institution overlay `PACK-INST-ISLAMIC` is a tighten-only layer with `CTL-CATALOGUE-ELIGIBILITY`, `CTL-TERMINOLOGY`, `CTL-PROHIBITED-CATEGORY` and `CTL-SHARIA-APPROVAL`; religion is never inferred (DEC-54).
FR73: Sharia approval state on the overlay pack, Track B catalogue items, templates and reason-code texts, recorded by `sharia_approver`; dual legal + Sharia approval; BLOCK `SHARIA_REVIEW_PENDING` otherwise (DEC-55, DEC-63).
FR74: Per-cell synthetic catalogues (`CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH`, `CAT-B-AE-ISLAMIC-SYNTH`) and one generator seed per cell; Islamic financing only as `CREDIT_REFERRAL`; Mudarabah and Takaful always human review; late-payment charity data only (DEC-56).
FR75: Model registry entries carry `track` and `cell_scope`; training, evaluation and fairness run per track; a mismatched entry is refused (DEC-57).
FR76: Locales and currency per cell; Track B templates in Arabic and English with translation and Sharia terminology review; right-to-left dashboard (DEC-59).
FR77: Illustrative UK packs `PACK-GB`, `PACK-GB-ENG`, `PACK-GB-ENG-LDN` from planning/19 rows, all PENDING_LEGAL_REVIEW (DEC-50).
FR78: Illustrative AE packs and the overlay content from planning/20 rows, PENDING_LEGAL_REVIEW and not SHARIA_APPROVED until named approvers sign (DEC-50, DEC-54, DEC-55).
FR79: Two scripted standalone demos from one build: Track A (two cells) and Track B (DEC-64).
FR80: Dashboard read endpoints: purpose-bound tokenised customer directory (DEC-68, contract 1.2.0), purpose-bound stored-transactions read, aggregate-only portfolio summary (DEC-62).
FR81: Track A human review on offer suppression, retention-offer allocation and vulnerability routing (DEC-66).
FR82: Track B telemarketing contact caps; in-app counted as telemarketing pending planning/20 IQ4 (DEC-67).

### NonFunctional Requirements

NFR1: Evaluate API p95 under 1 s in the demo (12 months of transactions).
NFR2: PDP decision p95 under 5 ms, in-process with a cached bundle [ASSUMPTION].
NFR3: Nightly batch handles 50k synthetic customers in 30 min or less on a laptop.
NFR4: Full function with the LLM unavailable (templates).
NFR5: Not ready without a verified policy bundle. Max-age then `CTL-DEFAULT-DENY`. The bundle is never used past `review_due`.
NFR6: Replay is deterministic with stored versions and `as_of`.
NFR7: Logs are PII-free and structured, with OpenTelemetry and the SEC-21 alert set.
NFR8: The dashboard meets WCAG 2.2 AA.
NFR9: Offline-capable, with no mandatory external calls.
NFR10: Security baseline SEC-01..SEC-21 (OIDC, TLS 1.2+, encryption at rest, signed images and packs, SBOM, secret handling, rate and size limits).
NFR11: Privacy baseline PRV-01..PRV-13 (tokens only, minimisation, no sensitive inference, PIA before non-synthetic use).
NFR12: Contract stability: SemVer, `/v1` in the URL, a deprecation window of at least 6 months, and a default rate limit of 50 req/s with burst 100 [ASSUMPTION].
NFR13: The same seed gives identical data, metrics and reports.
NFR14: Isolation: automated tests prove no personal data, token, key, model or training set crosses a cell or a track, and that `track` tags match at bootstrap.
NFR15: Data location: each cell runs in its declared region; Track B runs only on bank-hosted infrastructure with no external network dependency.

### Additional Requirements

From `ARCHITECTURE-SPINE.md`:
- **No starter template.** Story 1.1 lays down only the repository skeleton in the Structural Seed: `core/`, `ports/`, `adapters/`, `config/`, `bootstrap.py`, plus the import-linter rule for AD-2. Nothing beyond what that story needs.
- **CI guards**, each introduced by the first story that needs it:
  - Import direction (AD-2).
  - Jurisdiction-literal lint (AD-3).
  - Forbidden-term schema linter with the explicit scope list (AD-6).
  - OpenAPI breaking-change diff (AD-13).
  - Feature-list vs exclusion-list check (AD-15).
  - Role-matrix negative tests (AD-21).
- Every module result type has a required `decision_id` (AD-4). The response assembler alone sets `permitted_use` and `not_for_credit_decisioning` (AD-6).
- Money is held in minor units with currency (one currency per cell). Timestamps are TIMESTAMPTZ UTC with the cell's business timezone. Tokens only (AD-9).
- Audit and decision log are append-only and hash-chained, with `subject_ref` tokens and per-subject keys (AD-10).
- `deployment_id` and `cell_id` check constraints on every business row, plus token claim checks and a bootstrap scope check (AD-16, AD-23). Deployment profile, cells, Track B isolation, institution overlay, per-track registry and per-track LLM hosting follow AD-23..AD-27.
- Consent purposes are config reference data (AD-17). Packs are signed, `/ready` is gated (AD-18). Standalone rejects any row that is not synthetic (AD-19). Outbox for events (AD-20). DQ short-circuits per module (AD-22).
- Contract-first: the normative `contracts/openapi/retail-intelligence-v1.yaml` (the spine's `contracts/openapi/v1.yaml`) and `contracts/events/` from PH-3 are the source of truth for every endpoint and event story (AD-13; see `contracts/README.md`).

Non-development prerequisites (governance track; produce no code; gate the stories noted):
- **P-1:** GATE-1 ratification of DEC-01, 02, 04, 09, 43, 44, 49 and BQ-1..BQ-7, BQ-11 (from planning/10 S1.1, S1.2, S1.4). **Done 2026-11-10 by developer delegation** (12 §H.7.2; DEC-13 and DEC-28 superseded; manager countersignature recommended). Build still needs an explicit implementation instruction.
- **P-2:** Missing source material MS1-MS4 re-requested (S1.3). Gap accepted (12 §H.3).
- **P-3:** GATE-2: stack versions pinned (DEC-05, DEC-08) and UI system and visual identity chosen (UX-OQ-1..3) (S2.1-S2.4). Gates Story 1.1 (versions) and Epic 9 (visual).
- **P-4:** PH-3 design documents: full ERD, normative OpenAPI v1, adapter port list, credit-boundary placeholder CI-1..CI-9, draft role matrix (S3.1-S3.4, S19.1 document part); GATE-3 ratifies DEC-03, 12, 31, 38. Gates every endpoint story. **Contract part done 2026-11-09** (`contracts/`, lint clean). **ERD, port list and role-matrix draft done 2026-11-10:** `contracts/data-model/erd.md`, `contracts/ports.md` (incl. the S3.4 `CreditBoundaryPort` placeholder), `contracts/role-scope-matrix.md` / `.json`. GATE-3 batch ratified by developer delegation 2026-11-10 (DEC-03, 12, 31, 38); contract gaps G-RS-1..3 decided by DEC-62 (Story 6.10); interim integration owner the developer (DEC-63).
- **P-5:** Institution profile intake (DEC-10, per track) and priority legal-question owners (S13.1, S13.3); GATE-2A. Gates Story 7.4 pack content.
- **P-9 (RESOLVED 2026-11-10):** target market decided by the developer: two tracks (DEC-50). Replaced by the per-track content gates P-10 and P-11. NYC content (Stories 7.4, 7.7, 9.8) waits only for P-5.
- **P-10:** UK applicability matrix planning/19 (written 2026-11-10), the UK operating entity type (DEC-65, BLOCKED-EXTERNAL; planning/19 UQ1) and a named legal approver for Track A. Gates Story 7.10 content and the London scenes of Story 9.10 beyond ILLUSTRATIVE fixtures.
- **P-11:** UAE regulatory and Sharia matrix planning/20 (written 2026-11-10), Track B bank facts (licence, onshore status, emirate; DEC-10) and named legal and Sharia approvers. Gates Story 7.11 content and the Track B content scenes of Story 9.11.
- **P-6:** Named policy approver (DEC-11) and, for Track B, a named Sharia approver. Interim roles are decided (DEC-63: Track A Head of Compliance; Track B Head of Compliance + Sharia Compliance / ISSC), but they cannot approve. Hard stop for any approval or ACTIVE activation (Stories 7.1, 7.2, 7.9 run with the roles unassigned).
- **P-7:** Minors legal items routed (S18.4; DEC-22). Gates any Tier-2 design.
- **P-8:** Compliance review pack (S12.2) and host adaptation checklist (S11.4): documents produced alongside Epic 11 and Epic 10.

### UX Design Requirements

UX-DR1: Decision badge on every output, showing outcome, `decision_id` and controls applied, which opens the gate trace.
UX-DR2: Gate trace panel listing each gate with result and reason code. Suppressed candidates stay visible with their reason.
UX-DR3: Persistent pending-legal-review banner while any active pack or policy is PENDING_LEGAL_REVIEW. When the bundle is ILLUSTRATIVE, a persistent watermark banner reads "illustrative - not legal advice - not for real customers" (DEC-44).
UX-DR4: Protected-customer notice replaces every blocked section for MINOR_PROTECTED / AGE_UNKNOWN and states which tier blocked it.
UX-DR5: SYNTHETIC / "DEMO - not real terms" label on every dataset, product and persona surface.
UX-DR6: Version strip (rule, model, config, pack versions, `as_of`) on each output.
UX-DR7: Evidence table behind each need and churn driver (6 or 12 month window).
UX-DR8: Review item row with state, SLA countdown, required role and actions. A reason is mandatory for reject and escalate.
UX-DR9: Outcome recorder with fixed outcome and reason pick-lists and no free text.
UX-DR10: Jurisdiction selector limited to the DEC-16 test values.
UX-DR11: Simulator diff of decisions changed by activity and jurisdiction.
UX-DR12: WCAG 2.2 AA: full keyboard navigation, visible focus, screen-reader labels for statuses, and colour never the only signal.
UX-DR13: State rendering for DENIED-minor, DENIED-jurisdiction, BLOCKED-pending, DATA_QUALITY_INSUFFICIENT, consent withdrawn, awaiting review, template fallback and empty-after-gating.
UX-DR14: App-wide not-ready screen when the policy bundle is unverified.
UX-DR15: Voice rules: policy IDs and statuses only, never "permitted by law X"; credit wording "Refer to the credit platform for assessment"; protected wording per EXPERIENCE.md; no approval or celebration styling.
UX-DR16: Screen visibility per the role-to-scope matrix. Customer names are never shown.
UX-DR17: Visual tokens (colour, typography, components) applied once DESIGN.md is decided. **Blocked** by UX-OQ-1..3 (prerequisite P-3).
UX-DR18: Persistent track and cell indicator (track, `institution_type`, `cell_id`) in the app shell; in Track B ILLUSTRATIVE bundles the watermark adds "illustrative - not a Sharia ruling"; Sharia approval state shown next to `pack_state` (DEC-55, DEC-61).
UX-DR19: Track B locale switch Arabic/English with full right-to-left layout, mirrored navigation and WCAG 2.2 AA in both directions (DEC-59).
UX-DR20: Customer picker shows tokens and coarse codes only (from the purpose-bound `GET /v1/customer-directory`, with the access purpose chosen and a token or narrowing filter required; DEC-68); the portfolio view shows aggregates only, with suppressed small cells marked, never tokens (DEC-62).

### FR Coverage Map

FR1-FR4: Epic 1 - synthetic dataset
FR5-FR9: Epic 3 - transaction ledger and data quality
FR10-FR13: Epic 4 - profile, segments, minors routing, fairness
FR14-FR18: Epic 5 - needs, churn, model governance
FR19-FR25: Epic 6 - recommendations, retention, review, outcomes
FR26-FR31, FR33, FR34, FR39, FR43: Epic 2 - policy core, signed packs, baseline, lists, applicability records
FR35-FR38, FR40-FR42: Epic 7 - authoring workflow, NYC packs, simulator, golden cases
FR44-FR47: Epic 8 - orchestrator and assistant
FR48-FR53: Epic 9 - dashboard and demo
FR32, FR54 (publication), FR56-FR60: Epic 10 - plugin contract and parity
FR54 (capabilities endpoint), FR55, FR61, FR62: Epic 2 - envelope, identity, audit
FR63: Epic 6 - consent withdrawal recompute
FR64-FR68: Epic 11 - lifecycle, DSR, backup, runbooks, evaluation
FR69, FR70, FR72: Epic 2 - deployment profile, cells, institution overlay (Stories 2.11-2.13)
FR71: Epic 2 (bootstrap refusals, Story 2.11) and Epic 11 (isolation suite, Story 11.7)
FR73, FR77, FR78: Epic 7 - Sharia approval, UK packs, AE packs and overlay content (Stories 7.9-7.11)
FR74: Epic 1 - per-cell catalogues and seeds (Story 1.5)
FR75: Epic 5 - per-track registry scope (Story 5.7)
FR76: Epic 8 (Story 8.5) and Epic 9 (Story 9.11)
FR79: Epic 9 - per-track demo scripts (Stories 9.10, 9.11)
FR80: Epic 6 - dashboard read endpoints (Story 6.10)
FR81: Epic 7 - Track A human-review policies (Story 7.10)
FR82: Epic 7 - Track B contact caps (Story 7.11)
UX-DR18-UX-DR20: Epic 9 (Stories 9.10, 9.11; UX-DR20 also Stories 9.2, 9.7)

## Epic List

### Epic 1: Reproducible synthetic dataset per cell (NYC, London, UAE)
Testers and data scientists can generate a seeded, labelled, synthetic-only dataset per cell (NYC, London, UAE Islamic) with jurisdiction and age test records, a SYNTHETIC catalogue and a validation report.
**FRs covered:** FR1, FR2, FR3, FR4, FR74

### Epic 2: Every activity decided by policy, fail-safe by default
Compliance officers can rely on every request being authenticated, every activity being decided by the PDP against signed packs (with restrictive defaults, a locked baseline, applicability records and exclusion lists), and every decision being logged, replayable and labelled not for credit decisioning.
**FRs covered:** FR26, FR27, FR28, FR29, FR30, FR31, FR33, FR34, FR39, FR43, FR54 (capabilities), FR55, FR61, FR62, FR69, FR70, FR71 (bootstrap), FR72

### Epic 3: A clean, trustworthy transaction ledger
Analysts get de-duplicated, linked, categorised transactions and income/expense/savings figures, with data-quality short-circuits.
**FRs covered:** FR5, FR6, FR7, FR8, FR9

### Epic 4: Customer profile and segments with minors protection
Relationship staff see a customer profile and multi-membership segments. Compliance sees protected routing and a fairness view.
**FRs covered:** FR10, FR11, FR12, FR13

### Epic 5: Explainable needs and churn under model governance
Staff see evidence-backed needs and churn risk with drivers. Only registry entries cleared to serve (RULESET ACTIVE or ML_MODEL APPROVED) produce visible predictions; ML stays SHADOW while DEC-19 is open.
**FRs covered:** FR14, FR15, FR16, FR17, FR18, FR75

### Epic 6: Safe recommendations and retention with human review
Staff get gated, catalogue-grounded suggestions and retention actions, can clear flagged items, and can record outcomes. Consent withdrawal takes effect immediately.
**FRs covered:** FR19, FR20, FR21, FR22, FR23, FR24, FR25, FR63, FR80

### Epic 7: Per-track policy packs (US, UK, AE, Islamic overlay), simulator and maker-checker
Policy authors, legal approvers and Sharia approvers can draft illustrative federal, NY State and NYC packs (and UK, AE and Sharia overlay packs once P-10/P-11 allow), simulate them, run golden and invariant tests, and route them through maker-checker. ACTIVE activation is impossible without a named approver; only the demo-only ILLUSTRATIVE activation is possible (DEC-44).
**FRs covered:** FR35, FR36, FR37, FR38, FR40, FR41, FR42, FR73, FR77, FR78, FR81, FR82

### Epic 8: Grounded explanations and a safe assistant
Staff get template explanations validated against tool results and an assistant that refuses out-of-scope requests.
**FRs covered:** FR44, FR45, FR46, FR47, FR76

### Epic 9: Standalone dashboard and manager demo
The manager can run the scripted Track A demo (NYC and London cells) and the separate Track B demo (Islamic bank, Arabic/English) end to end in an accessible dashboard, including fail-safe, minor and pending-review scenes.
**FRs covered:** FR48, FR49, FR50, FR51, FR52, FR53, FR76 (RTL), FR79

### Epic 10: Plugin contract v1 and host parity
Host integrators can call contract v1 with host identity and stricter overlays, and get results identical to standalone.
**FRs covered:** FR32, FR54 (publication), FR56, FR57, FR58, FR59, FR60

### Epic 11: Privacy, resilience and evaluation readiness
Data-protection officers and operators get lifecycle hooks, DSR erasure, backup and restore, governed reason codes and runbooks. The manager gets the technical evaluation report.
**FRs covered:** FR64, FR65, FR66, FR67, FR68, FR71 (isolation suite)

## Epic 1: Reproducible synthetic dataset per cell (NYC, London, UAE)

Testers and data scientists can generate a seeded, labelled, synthetic-only NYC dataset. *Source: planning/10 E4; planning/07 §7; planning/09 §1.*

### Story 1.1: Seeded customer and account generator

As a tester,
I want reproducible synthetic NYC adult customers and accounts plus jurisdiction and age test records,
So that demos and tests are repeatable without real data.

**Acceptance Criteria:**

**Given** the repository skeleton from the spine Structural Seed with the AD-2 import-linter rule in CI
**When** the generator runs with a seed
**Then** it writes `customer` and `account` rows (tokens only, `birth_year_month`, `age_status`, residence and booking jurisdiction, `deployment_id`, `is_synthetic=true`)
**And** the same seed produces an identical dataset hash.

**Given** the DEC-16 test set
**When** the generator runs
**Then** it includes `US-NY-NYC`, `US-NY` (outside NYC), `US-CA`, missing and `UNKNOWN` jurisdictions, an NYC resident booked elsewhere, minors 16-17 and age-unknown records
**And** it includes the Alex persona with a consented self-declared occupation and a `VARIABLE_INCOME` persona [ASSUMPTION].

**Given** a row without `is_synthetic=true`
**When** standalone ingest receives it
**Then** the row is rejected (AD-19).

Source: S4.1. FR1.

### Story 1.2: Transaction generator with edge cases

As a tester,
I want transactions with configurable edge cases and ground-truth labels,
So that transaction logic is stress-tested.

**Acceptance Criteria:**

**Given** configured rates for duplicates, pending-then-booked, refunds, reversals, transfers, multi-currency and irregular income
**When** the generator runs
**Then** each rate is within tolerance and every injected case carries a ground-truth label
**And** money is stored as `amount_minor` plus `currency` (AD-9).

Source: S4.2. FR2.

### Story 1.3: Synthetic catalogue, consent and preferences

As a product tester,
I want a SYNTHETIC NYC catalogue and consent/preference records,
So that eligibility and consent gates are testable.

**Acceptance Criteria:**

**Given** the catalogue loader
**When** the catalogue loads
**Then** every product has `jurisdictions_offered`, a `catalogue_version` and the label "DEMO - not real terms", and no real product names or terms appear (BQ-9).

**Given** consent records
**When** they load
**Then** purpose codes are validated against config reference data, not a hard-coded enum (AD-17)
**And** MARKETING, affiliate and channel purposes are present only where explicitly generated.

Source: S4.3. FR3.

### Story 1.4: Behavioural scenarios and validation report

As a data scientist,
I want time-structured churn and need scenarios with a validation report,
So that models can be evaluated without leakage.

**Acceptance Criteria:**

**Given** a seeded run
**When** scenarios are generated
**Then** events are time-stamped and labels come from a documented latent process.

**Given** the run output
**When** the validation report is produced
**Then** it contains distribution sanity per persona, edge-case coverage counts, a label-leakage audit, persona and jurisdiction balance, and a reproducibility hash
**And** it states the synthetic-validity limits (for GATE-4 review).

Source: S4.4. FR4.

### Story 1.5: Per-cell seeds and synthetic catalogues for London and the Islamic bank

As a tester,
I want one generator codebase that produces each cell's data from its own seed, with that cell's synthetic catalogue,
So that both tracks can be demonstrated and tested without real data.

**Acceptance Criteria:**

**Given** generator config listing cells `A-US`, `A-UK` and `B-AE` (cell IDs, jurisdiction codes, currency, locale and timezone as config values only)
**When** each cell is generated from its seed
**Then** the same seed gives byte-identical output per cell, every row has `is_synthetic=true`, and outputs are written separately per cell (never mixed; no FX between cells)
**And** London personas (e.g. a salaried renter, `GB-ENG-LDN`, GBP) and UAE personas (e.g. a salaried customer with a Wakala account, `AE-<emirate>` [ASSUMPTION], AED) exist alongside Alex in `A-US`.

**Given** the catalogues `CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH` and `CAT-B-AE-ISLAMIC-SYNTH` (planning/21 §5)
**When** loaded
**Then** every item is labelled SYNTHETIC with no real names, rates, profit rates, fees or terms
**And** Murabaha and Tawarruq personal finance, Ijarah home finance and the Islamic card carry recommendation kind `CREDIT_REFERRAL` only; Mudarabah investment deposit and Family Takaful carry `requires_human_review: true`; Islamic items carry a `sharia_status` attribute; any `late_payment_treatment` is descriptive text only.

**Given** each cell's control personas
**When** generated
**Then** they include minor, age unknown (with the per-jurisdiction age threshold: US 18, GB 18, AE 21 [ASSUMPTION]), consent withdrawn, unknown jurisdiction, out-of-scope (`US-CA`), cross-cell (NYC residence + `GB` booking), cross-track (`GB` presented to Track B, `AE` presented to Track A), pending-review and credit-request cases
**And** the `B-AE` merchant taxonomy includes the illustrative prohibited categories so the exclusion path can be tested.

Source: planning/21 §5; DEC-56, DEC-60, DEC-16 (as extended in 12 §H.7.2). FR74.

## Epic 2: Every activity decided by policy, fail-safe by default

Compliance officers can rely on every request being authenticated and every activity being decided, logged and labelled. *Source: planning/10 E14, S15.1, S16.4, S16.5, S13.2, S19.2; planning/14; planning/15 §3-§5.*

### Story 2.1: Jurisdiction hierarchy and customer context resolution

As an architect,
I want jurisdictions held as data and each customer's context resolved,
So that no core code names a state.

**Acceptance Criteria:**

**Given** jurisdiction records `*`, `US`, `US-NY`, `US-NY-NYC`
**When** a context with residence and booking `US-NY-NYC` is resolved
**Then** the jurisdiction set is `US-NY-NYC`, `US-NY`, `US`.

**Given** missing or conflicting residence and booking
**When** resolved
**Then** the result is `UNKNOWN_JURISDICTION` (DEC-25 safe assumption)
**And** `US-CA` resolves to `UNSUPPORTED_JURISDICTION`.

**Given** a jurisdiction literal in `core/` outside `core/policy`
**When** CI runs
**Then** the lint fails (AD-3).

Source: S14.1. FR26.

### Story 2.2: Policy model, control catalogue and signed pack loading

As a developer,
I want policy records, a versioned control catalogue and signed pack loading,
So that policies reference mechanisms and tampered packs never load.

**Acceptance Criteria:**

**Given** a pack YAML with `matrix_ref`, citation, URL, effective dates and controls
**When** it is loaded
**Then** it is schema-validated and the control parameters are validated per control.

**Given** an unsigned or hash-mismatched pack
**When** loaded
**Then** it is refused and `/ready` stays false (AD-18).

**Given** the pack lifecycle (DEC-45)
**When** a pack is in DRAFT, VALIDATED, IN_LEGAL_REVIEW, APPROVED or RETIRED
**Then** it is never in the runtime bundle, and only ACTIVE or (demo) ILLUSTRATIVE packs decide.

**Given** a pack in state ILLUSTRATIVE
**When** the deployment is plugin mode, has `demo=false`, lacks the synthetic-only guard, or the pack is signed with a DEMO key in a non-demo deployment
**Then** the pack is refused at load with `ILLUSTRATIVE_PACK_REFUSED` and `/ready` stays false
**And** a bundle mixing ILLUSTRATIVE and ACTIVE packs is refused (DEC-44).

Source: S14.2, S15.1; planning/14 §8.4. FR27, FR34.

### Story 2.3: PDP resolution and conflict rules

As a compliance engineer,
I want a deterministic PDP,
So that outcomes are repeatable.

**Acceptance Criteria:**

**Given** two applicable policies with conflicting controls
**When** decided
**Then** the most restrictive wins per the control combining rule (DEC-26).

**Given** an unresolvable conflict
**When** decided
**Then** the outcome is DENY with reason `POLICY_INDETERMINATE` and a conflict record.

**Given** identical inputs and `as_of`
**When** decided twice
**Then** the outcomes are identical.

Source: S14.3. FR28.

### Story 2.4: Fail-safe behaviours

As a compliance officer,
I want restrictive defaults for every uncertain condition,
So that uncertainty never permits.

**Acceptance Criteria:**

**Given** each row of planning/14 §10
**When** its condition occurs (pending, `review_due` passed, unknown or unsupported jurisdiction, unknown age, unknown consent, no matching policy, bad signature, PDP exception, unknown control or obligation)
**Then** commercial activities are DENY or BLOCK, with one test per row.

**Given** a last-good bundle past its own `review_due`
**When** used
**Then** it is refused, with no grace period (DEC-14).

**Given** an ILLUSTRATIVE bundle with one policy carrying `illustrative_assumption: ASSUME_APPLICABLE` and one PENDING_LEGAL_REVIEW policy without an assumption
**When** decided
**Then** the first contributes its controls with flag `ILLUSTRATIVE` and reason `POLICY_ILLUSTRATIVE_ASSUMPTION`, and the second still BLOCKs
**And** with `honour_illustrative_assumptions: false` both BLOCK (DEC-44)
**And** with no jurisdiction pack in the bundle every non-governance activity is DENY, because the L0 whitelist is empty (DEC-45).

Source: S14.5. FR30.

### Story 2.5: Locked L0 baseline with minors two-tier list and starter exclusion lists

As a product owner,
I want internal safeguards that no pack can loosen,
So that credit decisions, minors' commercial processing and sensitive inference are impossible.

**Acceptance Criteria:**

**Given** `PACK-GLOBAL-BASELINE` with locked controls (`CTL-MINOR-PROTECT` Tier 1, `CTL-CREDIT-REFERRAL-ONLY`, `CTL-ADVERSE-ACTION-HANDOFF`, `CTL-LLM-GUARDRAIL`, `CTL-NO-INFERENCE`)
**When** any pack or overlay tries to permit a locked-blocked activity
**Then** the outcome stays DENY (property test).

**Given** a MINOR_PROTECTED or AGE_UNKNOWN subject
**When** a Tier-1 activity is requested, even with a `MINOR-APPROVAL-*` record
**Then** the outcome is DENY
**And** a Tier-2 activity is DENY unless a signed approval record is referenced; a config flag alone never relaxes it.

**Given** the DEC-23 starter lists (`EXCL-SENSITIVE`, `EXCL-FORBIDDEN-INFERENCE`, `EXCL-PROXY`, `EXCL-PROTECTION-ONLY`)
**When** loaded
**Then** they are versioned pack data marked PENDING_LEGAL_REVIEW.

**Given** an L0 policy that grants any activity
**When** CI runs
**Then** the build fails (L0 is restrictive-only)
**And** the L0 approval record has basis `INTERNAL_POLICY`, approver != maker, and the signature uses the DEMO-class key only where `demo=true`, otherwise the DEPLOYMENT key (DEC-45).

Source: S16.4, S16.5, S18.2. FR33, FR39.

### Story 2.6: PEP contract and decision log with replay

As an auditor,
I want every module output tied to a logged, replayable decision,
So that a recommendation can be reproduced.

**Acceptance Criteria:**

**Given** the module result base type
**When** a module returns without a `decision_id`
**Then** it fails type-checking and a module-skip test fails (AD-4).

**Given** an obligation the PEP does not understand
**When** enforced
**Then** the result is DENY.

**Given** a logged decision
**When** replayed with its pack versions and `as_of`
**Then** the outcome is identical
**And** the hash-chained `policy_decision_log` stores policy IDs and statuses only, keyed by `subject_ref`.

Source: S14.4, S14.6. FR29, FR31.

### Story 2.7: Local identity, deny-by-default role matrix and hash-chained audit

As a security officer,
I want every API call authenticated, scoped and audited,
So that access is least-privilege and tamper-evident.

**Acceptance Criteria:**

**Given** the local demo issuer with SEC-02 roles
**When** a call lacks a valid token
**Then** it returns 401.

**Given** an unauthenticated-endpoint allow-list held in config (empty in this story; Story 2.8 adds the health endpoints)
**When** the allow-list is checked
**Then** every other route needs a token.

**Given** `contracts/role-scope-matrix.json` (draft per prerequisite P-4; supersedes planning/15 §5.1)
**When** CI runs
**Then** one negative test per cell passes, and separation of duties holds (maker != approver, `policy_release` cannot author or approve, auditors read-only).

**Given** a token whose `ri_deployment` or `ri_jurisdictions` mismatches the instance or request
**When** called
**Then** it returns 403 `JURISDICTION_NOT_PERMITTED`, audited.

**Given** any authenticated request or PDP decision
**When** served
**Then** an append-only, hash-chained `audit_record` is written through `AuditSinkPort` (later customer-level endpoints reuse it), and a chain-verification job passes.

**Given** `POST /v1/policy/decisions` (dry run) and `GET /v1/policy/packs` / `GET /v1/policy/packs/:pack_id/:version`
**When** called with `ri.policy.query` / `ri.policy.read`
**Then** the outcome, obligations and pack metadata return, and no customer data is processed.

Source: S12.1 (role matrix part), SEC-01..SEC-03, SEC-07. FR61, FR62.

### Story 2.8: Canonical envelope, health, readiness and capabilities

As an integrator,
I want one response envelope and discovery endpoints,
So that every client sees versions, decisions and the purpose label consistently.

**Acceptance Criteria:**

**Given** any customer-level response
**When** assembled by the shared-core response assembler
**Then** it carries `contract_version`, `model_version`, `config_version`, a `policy` block (`decision_id`, `bundle_state`, `jurisdiction_set`, `packs_applied`, `effective_bundle_sha256`), `permitted_use`, `not_for_credit_decisioning: true` (DEC-42), `illustrative` and `watermark`
**And** when `bundle_state` is ILLUSTRATIVE, `illustrative` is true and `watermark` is "illustrative - not legal advice - not for real customers" on every response, event, export and explanation (DEC-44).

**Given** `GET /health`, `GET /ready` and `GET /v1/capabilities`
**When** called
**Then** readiness is true only with the DB up, L0 verified and every configured pack verified and loadable (DEC-45), and capabilities list activities, the jurisdictions in scope, runtime-bundle packs with `pack_state`, the `demo` flag and the serving registry entries.

**Given** the error model in planning/15 §4
**When** an error occurs
**Then** the response follows it, and idempotency keys are honoured on ingest.

Source: S19.2. FR54 (capabilities), FR55. Prerequisite: P-4 (normative OpenAPI v1).

### Story 2.9: Applicability decision records

As a legal approver,
I want each file 13 row's applicability stored as signed data,
So that packs cite only decided rows.

**Acceptance Criteria:**

**Given** a file 13 row ID
**When** a user without `policy_approver` tries to set `applicability_status`
**Then** the change is refused and audited.

**Given** DEC-10 is unresolved
**When** records are listed
**Then** every row shows PENDING_LEGAL_REVIEW.

Source: S13.2. FR43.

### Story 2.10: Consent and preference ingest endpoints

As a host integrator,
I want to push consent records and contact preferences through the contract,
So that the consent, suppression, channel and contact-limit gates use the host's data and never a guess.

**Acceptance Criteria:**

**Given** `POST /v1/consents:batchUpsert` (scope `ri.consent.write`, `Idempotency-Key` required) with up to 5,000 records per `contracts/openapi/retail-intelligence-v1.yaml`
**When** called
**Then** each record is upserted by `consent_ref` into `consent_record` or rejected individually with issues (no silent correction), purposes are validated against config reference data, not a hard-coded enum (AD-17), and in standalone a record without `is_synthetic=true` is rejected (AD-19)
**And** a record with `withdrawn_at` makes `CTL-CONSENT-PURPOSE` deny that purpose for every new decision from commit time, writes an audit record and lists the customer in `recompute_triggered_for` (the recompute itself is Story 6.8).

**Given** `POST /v1/preferences:batchUpsert` with marketing opt-out, channel preferences, `quiet_hours`, `contact_limits`, `vulnerability_flag_ref` and `suppressions[]`
**When** called
**Then** the values are stored in `contact_preference` as host-owned data, a `PAUSE_OFFERS` suppression with `origin: CREDIT_PLATFORM` (the CI-2 stub; synthetic only) feeds `CTL-MARKETING-SUPPRESS` only and never a feature, and vulnerability details are never accepted (reference only).

**Given** a replay of the same `Idempotency-Key` with a different payload
**When** called
**Then** the response is 409 `IDEMPOTENCY_CONFLICT`, and every response carries `not_for_credit_decisioning: true`.

Source: S19.6 (planning/10; readiness R-4). FR63 (input path), FR20 (suppression input), FR56 (consent through the contract). Prerequisites: P-4 (done), Story 2.7 (identity, scopes), Story 2.8 (envelope, error model).

### Story 2.11: Deployment profile loader and bootstrap refusals

As an operator,
I want each deployment to load one signed, read-only deployment profile,
So that the track, institution type and cells are configuration that no request can change.

**Acceptance Criteria:**

**Given** `PROFILE-TRACK-A` and `PROFILE-TRACK-B` fixtures (planning/21 §3)
**When** the bootstrap starts
**Then** the profile signature is verified, the profile hash is recorded on every decision record, and `GET /v1/capabilities` returns the read-only `deployment_profile` (profile ID and version, track, `institution_type`, `cell_id`, currencies, locales, catalogue, institution overlay; DEC-61)
**And** no request field can set or override track, institution type or cell.

**Given** an invalid signature, `institution_type: ISLAMIC` without an overlay pack, a pack, catalogue, model or template with a different `track` tag, a region not allowed by `CTL-DATA-RESIDENCY`, a disallowed `llm.hosting` value (Track B: only `NONE`/`BANK_ON_PREM`), or `demo: true` with `mode: PLUGIN`
**When** the bootstrap starts
**Then** the instance stays not ready (fail-closed) with a specific reason, and each case has its own test
**And** the same holds for a missing or unreadable profile, a disallowed signature algorithm, a key that is unknown, revoked or bound to the other track, an altered profile (hash mismatch), an unsupported profile schema version and a `profile_version` lower than the one recorded for the cell (planning/21 §3.1)
**And** setting any profile field through an environment variable, feature flag, config file or header has no effect (a test proves each)
**And** the model and template track-mismatch cases use `DeploymentProfilePort.verify_artefact_track` fixtures (contracts/ports.md driven port 18); the real registry and template loaders are wired to the same check in Stories 5.7 and 8.5, with no forward dependency here.

**Given** `core/`
**When** CI runs
**Then** the literal lint (AD-3) also rejects track IDs and `ISLAMIC`/`CONVENTIONAL` outside `core/policy` and the profile loader.

Source: planning/21 §1, §3; DEC-51, DEC-58, DEC-61; AD-23, AD-27. FR69, FR71 (bootstrap part).

### Story 2.12: Data cells and cell routing

As a privacy officer,
I want US and UK personal data held in separate cells with no cross-cell path,
So that no international transfer happens by design.

**Acceptance Criteria:**

**Given** the Track A profile with cells `A-US` (roots `[US]`) and `A-UK` (roots `[GB]`), each with its own database, subject-key store, decision and audit logs, object store and secrets
**When** a request arrives
**Then** it is served only by the cell whose roots contain the resolved residence and booking jurisdictions, and every business row carries `deployment_id` and `cell_id` check constraints (AD-16)
**And** a token from one cell is not found in the other.

**Given** residence in one cell's tree and booking in the other's (e.g. `US-NY-NYC` + `GB`)
**When** resolved
**Then** the context is UNKNOWN and every commercial activity is denied (DEC-24, DEC-25), with reason `CELL_SCOPE_MISMATCH` logged.

**Given** the code base
**When** CI runs
**Then** no query, job or adapter joins or copies personal data across cells, and no component reads another cell's database or store; only each cell's own suppressed aggregate output may be combined (planning/21 §2.1, §10A AG-7). Telemetry carries no customer token, each cell has its own `deployment_id`, outbox and backups, and a token from one cell is not found in the other with no lookup or hint (§2.2). Here the CI rule is tested with a fixture aggregate reader behind `DeploymentProfilePort.verify_artefact_track` (contracts/ports.md driven port 18); Story 6.10 wires the real portfolio path to the same rule.

Source: planning/21 §2; DEC-52; AD-23. FR70.

### Story 2.13: Institution-type overlay and the four Sharia controls

As a compliance officer,
I want Islamic-bank rules applied as a tighten-only policy layer,
So that Track B is stricter where it must be without any core branch.

**Acceptance Criteria:**

**Given** a profile with `institution_type: ISLAMIC` and a fixture `PACK-INST-ISLAMIC`
**When** the PDP resolves a decision
**Then** the order is L0 → jurisdiction packs → institution overlay → host overlay, combined per control with most restrictive wins (DEC-26)
**And** with `institution_type: CONVENTIONAL` no overlay is loaded and decisions are unchanged.

**Given** an overlay policy that grants an activity, widens an allow-list or raises an upper bound, or touches a locked L0 control or the minors list
**When** the pack is built
**Then** CI rejects it, and the monotonicity property test includes the overlay.

**Given** the control catalogue
**When** the four controls are added
**Then** `CTL-CATALOGUE-ELIGIBILITY` removes candidates lacking required attributes (e.g. `sharia_status = SHARIA_APPROVED`) before ranking; `CTL-TERMINOLOGY` blocks text containing listed terms and falls back to the template (`TERMINOLOGY_VIOLATION_FALLBACK`); `CTL-PROHIBITED-CATEGORY` keeps listed categories out of suggestions, offers, ranking and targeting features while budgeting labels stay neutral (`PROHIBITED_CATEGORY_EXCLUDED`); `CTL-SHARIA-APPROVAL` gives BLOCK `SHARIA_REVIEW_PENDING` for any customer-facing artefact without `SHARIA_APPROVED` (fixture states)
**And** `EXCL-FORBIDDEN-INFERENCE` contains "religion or religious observance" in every track, and a test proves no feature or segment infers it.

Source: planning/21 §4 (SDC-01..SDC-18 mapping in §4.4); DEC-54, DEC-55; AD-25. FR72. Term and category lists are SYNTHETIC placeholders [SHARIA-CONFIRM]; real content is Story 7.11.

## Epic 3: A clean, trustworthy transaction ledger

Analysts get correct transactions and aggregates. *Source: planning/10 E5; planning/07 §4, §4.1.*

### Story 3.1: Ingest, normalisation and de-duplication

As an analyst,
I want duplicates and pending items handled,
So that balances and totals are right.

**Acceptance Criteria:**

**Given** a synthetic batch through the M2 PEP (`TXN_ENRICHMENT`)
**When** processed twice
**Then** the results are identical (idempotent), exact duplicates on `dedupe_key` are skipped, and schema-invalid rows are rejected to the validation report (DQ-01, DQ-02).

**Given** a minor's transactions
**When** enriched
**Then** only age and routing checks run and no categorisation output is used downstream.

Source: S5.1. FR5.

### Story 3.2: Transfers, refunds, reversals and FX

As an analyst,
I want internal transfers paired and refunds and reversals linked,
So that spending is not overstated.

**Acceptance Criteria:**

**Given** fixture transactions
**When** processed
**Then** transfer pairs and refund/reversal links match ground truth
**And** no sum mixes currencies without a dated `fx_rate`.

Source: S5.2. FR6.

### Story 3.3: Categorisation with confidence and rule trace

As a customer-insight user,
I want categories with confidence and a rule trace,
So that spend mix is explainable.

**Acceptance Criteria:**

**Given** the rules taxonomy
**When** categorised
**Then** each transaction has a category, confidence, method and `rule_id`, and precision/recall are reported on synthetic labels.

**Given** `EXCL-FORBIDDEN-INFERENCE`
**When** CI checks the taxonomy
**Then** no forbidden-inference labels exist.

Source: S5.3. FR7.

### Story 3.4: Income, expense, savings and recurring detection

As a user,
I want salary, bills, recurrence and savings rate detected,
So that profiles are accurate.

**Acceptance Criteria:**

**Given** golden cases
**When** computed
**Then** results match the documented calculations, with money in minor units.

**Given** `POST /v1/transactions:enrich`
**When** called with up to 5,000 rows
**Then** the categories and flags return in the canonical envelope.

Source: S5.4. FR8.

### Story 3.5: Data-quality rules with per-module short-circuit

As a data owner,
I want DQ rules at ingest and before each module,
So that no output is built on thin or broken data.

**Acceptance Criteria:**

**Given** the DQ-01..DQ-10 thresholds as THRESHOLDS config (DEC-39)
**When** a customer has less history than a module's minimum (e.g. churn under 12 months)
**Then** that module returns `DATA_QUALITY_INSUFFICIENT` and no output, while other modules continue.

**Given** any DQ result
**When** stored
**Then** `data_quality_result` records `rule_version` for replay.

Source: S5.5. FR9.

### Story 3.6: Bulk data ingest batches

As a host integrator,
I want to load customers, accounts, transactions, goals and the catalogue in multi-part batches,
So that stateful-mode volumes arrive with lineage, de-duplication and a validation report.

**Acceptance Criteria:**

**Given** `POST /v1/ingest/batches`, `PUT /v1/ingest/batches/:id/parts/:n` (NDJSON, `X-Content-SHA256`, max 100 MB) and `POST /v1/ingest/batches/:id:commit` (scope `ri.ingest.write`) per the normative contract
**When** a batch is opened, its parts uploaded and committed
**Then** an `ingest_batch` lineage row exists, every row passes through the Story 3.1 normalisation and the Story 3.5 DQ rules (DQ-01..DQ-10), exact duplicates on `dedupe_key` are skipped, and rejects go to the validation report and are never silently corrected
**And** re-uploading a part with the same digest is idempotent, while a different digest for a committed part returns 409 `IDEMPOTENCY_CONFLICT`.

**Given** `GET /v1/ingest/batches/:id` (scope `ri.ingest.read`)
**When** called after commit
**Then** it returns the state (COMMITTED, HELD_FOR_REVIEW when DQ-03 exceeds 10 %, or FAILED), counts (received, accepted, rejected, duplicates skipped) and DQ results with `rule_version`.

**Given** a standalone deployment
**When** a row without `is_synthetic=true` or with an under-13 subject arrives
**Then** the row is rejected or quarantined (AD-19; planning/07 §8.1), and a host mapping spec (`mapping_ref`) is applied only if it is versioned.

Source: S19.7 (planning/10; readiness R-4). FR5, FR9, FR60. The `ingest.completed` event is written once the outbox exists (Story 10.7).

## Epic 4: Customer profile and segments with minors protection

*Source: planning/10 E6, E18; planning/12 §H.2 BQ-2, BQ-3.*

### Story 4.1: Customer profile and features

As a banker,
I want a profile with health indicators,
So that I understand the customer.

**Acceptance Criteria:**

**Given** an adult NYC customer with sufficient history
**When** M1 runs under `PROFILE_BUILD`
**Then** a `feature_snapshot` is stored with the feature dictionary version, `feature_list_sha256`, exclusion-list versions and `decision_id`
**And** excluded attributes are absent.

**Given** `GET /v1/customers/:token/insights` (or `:evaluate`)
**When** called
**Then** the profile returns in the envelope.

Source: S6.1. FR10.

### Story 4.2: Multi-membership segmentation

As an analyst,
I want overlapping segments with reasons,
So that segments reflect real customers.

**Acceptance Criteria:**

**Given** a customer aged 18-29 at the snapshot date (bounds in config)
**When** M3 runs
**Then** they are in Gen Z with a reason, separate from any credit GENZ tag.

**Given** a consented self-declared occupation in SOC groups 37/45/47/49/51/53 [ASSUMPTION]
**When** M3 runs
**Then** they are in blue-collar with the reason citing only the declared occupation
**And** income-variability rules assign `VARIABLE_INCOME` independently.

**Given** segment definitions
**When** stored
**Then** Gen Z and blue-collar carry `purpose: INSIGHT` and a `not_a_feature` tag (BQ-2, BQ-3). Stories 5.4 and 6.4 assert the tag against their feature lists.

**Given** `GET /v1/customers/:token/segments`
**When** called
**Then** the memberships return with reasons, validity and purpose.

Source: S6.2. FR11.

### Story 4.3: Protected routing for minors and unknown age

As a compliance officer,
I want minors and unknown-age customers routed to the protected path in M1-M3 and at ingest,
So that minors are safe.

**Acceptance Criteria:**

**Given** a 16-year-old synthetic customer
**When** M1 and M3 run
**Then** no commercial segment, profile or insight segment is produced without a signed `MINOR-APPROVAL-*` record, and each block is audited.

**Given** an under-13 record
**When** ingested
**Then** it is rejected or quarantined.

Source: S6.3, S18.1, S18.3. FR12. (The training filter for minors is verified in Story 5.4, the first story that builds a training set.)

### Story 4.4: Segment fairness view

As a compliance officer,
I want output distributions by segment and synthetic group,
So that disparities are visible.

**Acceptance Criteria:**

**Given** synthetic group labels set by the generator in a restricted store that models cannot read
**When** a batch runs
**Then** segment membership rate per group is reported, with cells below the configured minimum size [ASSUMPTION 100 per cell; DEC-48] shown as "insufficient", never back-filled and never counted as passing. Score distributions are added in Story 5.4 and surfacing and suppression rates in Story 6.4, reusing this view.

**Given** any attempt to infer a group attribute
**When** CI checks
**Then** none exists.

Source: S6.4. FR13.

## Epic 5: Explainable needs and churn under model governance

*Source: planning/10 E7; planning/12 §H.2 BQ-1, BQ-6, BQ-11.*

### Story 5.1: Model registry with approval status

As an owner,
I want registry entries of kind RULESET (DRAFT/ACTIVE/RETIRED) and ML_MODEL (DRAFT/SHADOW/APPROVED/RETIRED),
So that only entries cleared to serve produce customer-visible predictions (DEC-47).

**Acceptance Criteria:**

**Given** a `model_version` entry with its declared feature list
**When** CI runs
**Then** the feature list is checked against the exclusion lists (AD-15).

**Given** a DRAFT or SHADOW entry, or a RULESET that is not ACTIVE
**When** a prediction is requested
**Then** the PEP denies it under `CTL-MODEL-APPROVED-ONLY`.

**Given** `POST /v1/models/:id/versions/:v:transition` to APPROVED for an ML_MODEL without an MRM approval reference (DEC-19 unresolved)
**When** called by `model_admin`
**Then** it returns 409 `MODEL_APPROVAL_UNAVAILABLE`, is audited, and the status is unchanged
**And** RULESET DRAFT → ACTIVE and ML_MODEL DRAFT → SHADOW succeed and emit `model.status_changed` (planning/15 §2.1).

**Given** config `ruleset_requires_mrm_approval: true`
**When** a RULESET is activated without an MRM reference
**Then** it is refused.

Source: S7.3a; planning/12 §H.6 DEC-47. FR16.

### Story 5.2: Rule-based financial needs

As a user,
I want needs with evidence,
So that suggestions are explainable.

**Acceptance Criteria:**

**Given** the BQ-6 taxonomy with thresholds in config
**When** M4 runs over the last 6 months
**Then** each need carries evidence and a rule version
**And** `NEED_FINANCIAL_STRESS_SUPPORT` maps only to signposting plus human review, never a credit product, and no affordability or debt-to-income value is computed
**And** there is no credit-building need in the taxonomy (DEC-49)
**And** the needs ruleset is a RULESET registry entry in ACTIVE status (DEC-47).

**Given** `GET /v1/customers/:token/needs`
**When** called
**Then** the needs return in the envelope.

Source: S7.1. FR14.

### Story 5.3: Churn label and rules baseline

As a data scientist,
I want the BQ-1 label and a rules baseline,
So that ML has a fair comparator.

**Acceptance Criteria:**

**Given** a monthly snapshot
**When** labels are built
**Then** closure, salary diversion (60 days) or dormancy (90 days) within the 90-day window sets churn, with the BQ-1 exclusions applied.

**Given** the baseline
**When** evaluated on a time split
**Then** PR-AUC, ROC-AUC and Brier score are reported.

Source: S7.2 (part). FR15.

### Story 5.4: Transparent churn ML with time-split evaluation

As a data scientist,
I want a logistic-regression model compared with the baseline,
So that ML is used only when justified.

**Acceptance Criteria:**

**Given** features up to the snapshot only
**When** trained and evaluated
**Then** the leakage checks pass, calibration is reported and subgroup metrics are produced
**And** the training dataset excludes every MINOR_PROTECTED and AGE_UNKNOWN row (Tier-1 `MODEL_TRAINING`, FR12)
**And** CI asserts that no segment tagged `not_a_feature` is in the feature list, and the fairness view (Story 4.4) gains score distribution by group (FR13).

**Given** the model in SHADOW status (DEC-47)
**When** the batch runs on synthetic data
**Then** its outputs are written only to the shadow log, never to `prediction`, the API, events, M6, M7 or M9, and only aggregate metrics appear in the evaluation report and registry screen
**And** a shadow run on non-synthetic data is refused.

**Given** the PR-AUC uplift over the baseline
**When** adoption is evaluated
**Then** below the configured margin [ASSUMPTION 0.02] the model is not eligible for MRM review, and at or above it the model becomes eligible but stays SHADOW until an MRM approval reference exists (DEC-19); in both cases the RULESET baseline serves.

**Given** `GET /v1/customers/:token/churn-score`
**When** called
**Then** the score, band and the serving registry entry (the RULESET baseline in v1) return.

Source: S7.2. FR15.

### Story 5.5: Prediction explanations

As a user,
I want top drivers per prediction,
So that it is explainable.

**Acceptance Criteria:**

**Given** a prediction
**When** produced
**Then** the top drivers are returned as library reason codes, and they are identical across repeated runs with the same inputs.

Source: S7.4. FR18.

### Story 5.6: Drift indicator

As an owner,
I want a drift indicator per batch,
So that changes are visible.

**Acceptance Criteria:**

**Given** two batch runs
**When** compared
**Then** a drift value per model is stored, and a breach falls back to the baseline and raises an alert (CMP-07).

Source: S7.3b. FR17.

### Story 5.7: Per-track model registry scope

As a model-risk reviewer,
I want every registry entry tagged with its track and cell scope,
So that no model trained on one track's data ever serves the other.

**Acceptance Criteria:**

**Given** registry entries with `track` and `cell_scope`
**When** a deployment loads its serving registry
**Then** an entry whose `track` differs from the profile is refused, and `GET /v1/capabilities` lists only matching entries.

**Given** the synthetic MVP
**When** churn and needs run
**Then** one Track A rule baseline serves both `A-US` and `A-UK`, ML runs in SHADOW and is evaluated per cell, Track B has its own baseline and SHADOW models, and fairness reports are per cell (minimum group 100, DEC-48)
**And** pooled US+UK training is refused unless a recorded transfer assessment reference exists [LEGAL-CONFIRM].

Source: planning/21 §6; DEC-57, DEC-47; AD-26. FR75.

## Epic 6: Safe recommendations and retention with human review

*Source: planning/10 E8, S19.5; planning/14 §6, §13; planning/12 §H.1 DEC-37, §H.2 BQ-6, BQ-7.*

### Story 6.1: Catalogue-grounded candidates

As a banker,
I want recommendations only from the catalogue,
So that no product is invented.

**Acceptance Criteria:**

**Given** needs, segments and churn
**When** M6 generates candidates
**Then** every product code exists in the active `catalogue_version` and its `jurisdictions_offered` includes the customer's resolved jurisdiction.

**Given** an `ISLAMIC` deployment (contract 1.2.0, planning/20 SDC-11, planning/21 §4.4)
**When** a `PRODUCT_SUGGESTION` or `CREDIT_REFERRAL` is returned
**Then** it carries `sharia_disclosure` from the catalogue item version (Sharia basis and KFS references for every profile locale, ISSC approval and certificate references, contract type with no `CONVENTIONAL_*` code), and an item lacking any of them is suppressed (`SHARIA_REVIEW_PENDING` or `CATALOGUE_ITEM_NOT_ELIGIBLE`), never returned
**And** outside an `ILLUSTRATIVE` bundle only the approved shape (`sharia_status: SHARIA_APPROVED` with `sharia_approval_ref` and `issc_approval_ref`) validates; the demo shape carries `illustrative_sharia_assumption: ASSUME_COMPLIANT`, "illustrative - not a Sharia ruling" and no approval reference; a `CONVENTIONAL` (Track A) response never carries the block (contract schema tests). Disclosure wording and lists stay [SHARIA-CONFIRM].

Source: S8.1. FR19, FR72.

### Story 6.2: Eligibility gates and gate trace

As a compliance officer,
I want consent, jurisdiction, minor, suppression and channel gates,
So that only allowed offers show.

**Acceptance Criteria:**

**Given** each gate
**When** its failing case is run (no marketing consent, `US-CA`, minor, opt-out, pause-offers stub flag, host vulnerability flag, channel without consent)
**Then** the candidate is SUPPRESSED with a reason, kept for audit, and the gate trace holds the `decision_id` and control IDs.

**Given** the schema
**When** inspected
**Then** no affordability input or field exists.

Source: S8.2. FR20.

### Story 6.3: Credit only as referral

As a compliance officer,
I want credit products expressed only as referrals,
So that no credit decision is ever implied.

**Acceptance Criteria:**

**Given** a credit product candidate for an adult
**When** output
**Then** its kind is `CREDIT_REFERRAL` with no eligibility, limit, rate or likelihood.

**Given** a customer-outcome schema containing a forbidden term
**When** CI runs the linter with the planning/14 §13.1 scope list
**Then** the build fails.

Source: S8.2 (adjusted AC), planning/14 §13.1. FR21.

### Story 6.4: Ranking and reason codes

As customer-facing staff,
I want ranked products with reasons,
So that I can explain them.

**Acceptance Criteria:**

**Given** eligible candidates
**When** ranked
**Then** each has reason codes from the versioned library (DEC-40)
**And** support-first items rank above commercial ones when financial-stress signals exist
**And** CI asserts that no `not_a_feature` segment is a ranking feature, and the fairness view gains surfacing and suppression rates per group (FR13).

**Given** `POST /v1/customers/:token/recommendations:generate`
**When** called
**Then** the ranked list returns in the envelope.

Source: S8.3. FR22.

### Story 6.5: Retention suggestions and priority list

As a retention officer,
I want playbook suggestions and a priority list,
So that I can help at-risk customers.

**Acceptance Criteria:**

**Given** churn and needs outputs
**When** M7 runs
**Then** actions come only from the BQ-6 playbook, with no price, fee or rate change
**And** `RA_HUMAN_CHECKIN` is flagged `requires_human_review`, and contact limits and quiet hours apply.

**Given** `GET /v1/customers/:token/retention-actions`
**When** called
**Then** the suggestions and priority return.

Source: S8.4. FR23.

### Story 6.6: Human review queue

As relationship staff (`retention_reviewer`),
I want flagged items in a queue,
So that review obligations are met.

**Acceptance Criteria:**

**Given** a flagged item
**When** created
**Then** a `review_item` is QUEUED with `sla_due_at` from config [ASSUMPTION 2 business days], and the item is hidden until CLEARED_TO_SHOW.

**Given** a transition to REJECTED or ESCALATED without a reason, or by the requester
**When** submitted through `POST /v1/review-items/:id:transition`
**Then** it is refused.

**Given** the SLA passes
**When** checked
**Then** the item becomes EXPIRED and is never shown.

Source: S8.5. FR24.

### Story 6.7: Retention outcome capture

As a retention officer,
I want to record outcomes,
So that effectiveness can be tracked.

**Acceptance Criteria:**

**Given** a retention action
**When** staff record an outcome
**Then** an append-only `retention_outcome` row is written with an outcome from BQ-7, a fixed reason, time and source STAFF, and the write is audited.

**Given** an outcome
**When** recorded
**Then** no contact or other action is triggered.

Source: S8.6. FR25.

### Story 6.8: Consent withdrawal takes effect

As a compliance officer,
I want consent withdrawal to stop use immediately,
So that customers' choices are honoured.

**Acceptance Criteria:**

**Given** a customer with recommendations
**When** marketing consent is withdrawn through `POST /v1/consents:batchUpsert` (Story 2.10)
**Then** affected results are recomputed, recommendations are removed, open review items for them become CANCELLED, and an audit record is written.

Source: PRV-03 (planning/08); planning/09 §5 step 11. FR63.

### Story 6.9: Composite evaluate endpoint

As an integrator,
I want one stateless evaluate call that returns every module result,
So that the host and the dashboard get the whole picture in one request.

**Acceptance Criteria:**

**Given** `POST /v1/customers/:token/evaluate` with jurisdiction context, consent purpose and a 12-month transaction window
**When** called
**Then** insights, segments, needs, churn and recommendations return, each sub-result with its own `decision_id`, and blocked sections return `status: DENIED` with a reason
**And** p95 latency is under 1 s on the demo dataset (NFR1).

Source: planning/15 §2 (composite evaluate); DEC-32. FR55, FR57 (input to parity).

### Story 6.10: Dashboard read endpoints: customer list, stored transactions, portfolio summary

As relationship staff and analysts,
I want to pick customers, see their stored enriched transactions and view portfolio aggregates through `/v1`,
So that the dashboard needs no back door and shows the least data possible.

**Acceptance Criteria:**

**Given** `GET /v1/customer-directory` (`searchCustomerDirectory`, contract 1.2.0, DEC-68; scope `ri.customers.list` for analyst and relationship staff only, row filter by jurisdiction claim, cursor-paged, per cell)
**When** called
**Then** only customer token, resolved jurisdiction code, age-status class, data-quality flag and last-updated return; no name, contact, address, ZIP or account number; minors appear only as protected
**And** a missing or unknown `access_purpose` returns 400 `ACCESS_PURPOSE_INVALID`, a purpose not registered for the role or client 403 `ACCESS_PURPOSE_NOT_REGISTERED`, an enumeration without `segment_code` or `data_quality_status` 400 `NARROWING_FILTER_REQUIRED`, an exceeded per-user quota 429 `ACCESS_QUOTA_EXCEEDED`, and `auditor` or `service_host` 403 `FORBIDDEN_SCOPE`; pages never exceed 25 rows and a query never exceeds `max_results` (100)
**And** every call, allowed or denied, writes an `audit_record` with the access block (account, client, role, scope, purpose, access kind, filters, result count, outcome, denial code), readable by the auditor through the audit export; an unknown, other-cell or out-of-claim token gives the same empty page; `GET /v1/customers` (deprecated) returns 404 for every role (role matrix `deprecated_operation_tests`).

**Given** `GET /v1/customers/{customer_token}/transactions` (new purpose-bound scope `ri.transactions.read`, relationship staff and analyst only, consent purpose `INSIGHT`)
**When** called
**Then** the enriched view returns date, amount, currency, kind, category code and flags only; no raw description, counterparty or merchant name; sensitive or prohibited categories appear only as neutral category codes; a minor gets the protected response (age/routing only).

**Given** `GET /v1/portfolio/summary` (new scope `ri.portfolio.read`, viewer and analyst)
**When** called
**Then** only aggregate counts and distributions by segment, band, need and jurisdiction return; cells below the configured minimum (default 20 [ASSUMPTION]) are suppressed; minors are excluded from commercial aggregates; Track A may combine its two cells, never tracks
**And** aggregate disclosure control AG-1..AG-10 holds (planning/21 §10A): one fixed dimension per call; a nested jurisdiction filter or a non-snapshot `as_of` returns 400; `min_cell_size` below 20 is refused at config load; counts are rounded (default base 5 [ASSUMPTION]); complementary suppression leaves either zero or at least two suppressed cells per exhaustive dimension; each cell suppresses before combining and a code suppressed in any cell stays suppressed; every call is audited and the per-user quota returns 429; differencing tests (overlapping filters, adjacent snapshots, cross-dimension totals) recover no suppressed count.

**And** every response carries `not_for_credit_decisioning: true`, and each new deny cell in the role-scope matrix has a negative test (Story 2.7).

Source: planning/12 §H.7 DEC-62 (closes role matrix G-RS-1..G-RS-3); normative operations in `contracts/openapi/retail-intelligence-v1.yaml`. FR80. Must precede Stories 9.2 and 9.7.

## Epic 7: Per-track policy packs (US, UK, AE, Islamic overlay), simulator and maker-checker

*Source: planning/10 E15, E16, E17, S13.4; planning/14 §8, §9, §14. Pack content gated by P-5 and GATE-2A; approval and activation gated by P-6 (DEC-11).*

### Story 7.1: Maker-checker workflow

As a legal approver,
I want DRAFT → LEGAL_REVIEW → APPROVED → RETIRED with checker != maker,
So that no one self-approves.

**Acceptance Criteria:**

**Given** `policy_author` and `policy_approver` roles (DEC-12 default: compliance)
**When** the author tries to approve their own change
**Then** it is refused.

**Given** no user holds `policy_approver` (DEC-11 unresolved) and a pack fixture already in IN_LEGAL_REVIEW
**When** approval is attempted
**Then** it is refused, the pack stays in IN_LEGAL_REVIEW, and the pack status shows `approver_assigned: false` in the UI and API.

**Given** `POST /v1/policy/packs/:id/:version/approvals` (planning/15 §2.1)
**When** the approval targets an `illustrative: true` policy
**Then** it returns 422 `ILLUSTRATIVE_NOT_APPROVABLE`, and the call is role-gated and audited (DEC-45, DEC-46). (Upload, `:validate` and `:submit` are delivered in Story 7.6, after the golden-case runner in 7.5, so this story has no forward dependency.)

Source: S15.2. FR35.

### Story 7.2: Activation control

As a manager,
I want activation to need ops plus recorded sign-off,
So that unapproved packs never go live.

**Acceptance Criteria:**

**Given** a pack that is not APPROVED
**When** `policy_release` tries to activate it
**Then** activation is refused and audited.

**Given** an APPROVED pack with recorded MGR/LEG sign-off
**When** activated
**Then** a `policy.pack.activated` CloudEvents row is written to a transactional outbox table, created by this story as the first event producer (AD-20).

**Given** a VALIDATED, all-illustrative pack fixture (the upload and validate pipeline arrives in Story 7.6)
**When** `policy_release` (not the maker), confirming each `illustrative_assumption`, activates it with `target_state: ILLUSTRATIVE`
**Then** it becomes ILLUSTRATIVE only in a deployment with `mode=STANDALONE`, `demo=true` and the synthetic-only guard; elsewhere it returns 409 `ILLUSTRATIVE_NOT_PERMITTED`
**And** the event carries `pack_state: ILLUSTRATIVE` (DEC-44).

Source: S15.3. FR36.

### Story 7.3: Periodic review and expiry

As a compliance officer,
I want review reminders and expiry,
So that stale policies fail safe.

**Acceptance Criteria:**

**Given** a policy whose `review_due` has passed
**When** decided
**Then** it is treated as PENDING_LEGAL_REVIEW (BLOCK), and a reminder event exists before expiry.

Source: S13.4, S15.4. FR37.

### Story 7.4: Illustrative federal, NY State and NYC packs

As a policy author,
I want draft packs mapped from candidate file 13 rows,
So that every policy is sourced.

**Acceptance Criteria:**

**Given** candidate rows (federal F01, F04-F07, F09-F12, F14; NY S01, S04-S08, S11-S13, S16; NYC C02, C04-C07, with placeholders for C01, C03)
**When** packs are authored
**Then** each policy has a `matrix_ref`, citation and URL, status PENDING_LEGAL_REVIEW and `illustrative: true`
**And** no legal text is invented. Rows not decided at GATE-2A stay as placeholders
**And** any `illustrative_assumption` used for the demo is set by the maker, confirmed by the `policy_release` activator (not the maker) in the ILLUSTRATIVE activation record (Story 7.2), and never changes `applicability_status` (DEC-44).

Prerequisite: P-5 (P-9 resolved 2026-11-10: DEC-50 two tracks; NYC is the `A-US` cell). Until P-5 only the pack mechanism and test fixtures are built, not jurisdiction content.

Source: S16.1-S16.3. FR38.

### Story 7.5: Golden cases and invariant tests

As a tester and compliance engineer,
I want golden cases and property tests,
So that packs are safe to build.

**Acceptance Criteria:**

**Given** golden cases for an NYC resident, an unsupported state, an unknown jurisdiction and a cross-jurisdiction customer
**When** CI runs
**Then** they pass in the standalone harness (Story 10.5 reuses them unchanged for plugin parity), and a failing case blocks the pack build.

**Given** property tests
**When** run
**Then** they hold: monotonicity, determinism, fail-safe, locked baseline, and no policy permitting a credit decision.

Source: S17.2, S17.3. FR41.

### Story 7.6: Policy simulator

As a policy author,
I want to run a candidate pack against golden cases and a synthetic population,
So that I see what changes before review.

**Acceptance Criteria:**

**Given** a candidate pack (any state except RETIRED) and the current runtime bundle
**When** the simulator runs (CLI, or `POST /v1/policy/simulations` then `GET /v1/policy/simulations/:id`, planning/15 §2.1)
**Then** a diff report shows decisions changed by activity and jurisdiction, on synthetic data only
**And** LEGAL_REVIEW submission is refused without this report attached
**And** a run whose candidate hits PENDING_LEGAL_REVIEW policies writes an audit record and emits `policy.review.required` with `source: SIMULATION`.

**Given** the pack endpoints in planning/15 §2.1 (upload, `:validate`, `:submit`), the golden-case runner from Story 7.5 and this simulator
**When** a pack moves DRAFT → VALIDATED → IN_LEGAL_REVIEW
**Then** each transition is role-gated and audited, `:validate` runs the Story 7.5 checks, and `:submit` is refused without a `simulation_id` (DEC-45, DEC-46).

Source: S17.1. FR40.

### Story 7.7: Legal-review-pending demo case

As a manager,
I want the fail-safe visible,
So that the demo shows honest behaviour.

**Acceptance Criteria:**

**Given** Alex with a watermarked commercial recommendation permitted under the ILLUSTRATIVE demo bundle (DEC-44)
**When** a new candidate version of the relevant pack without the illustrative assumption (plain PENDING_LEGAL_REVIEW) is uploaded and run in the simulator against the runtime bundle
**Then** the diff shows the recommendation blocked and flagged, and a `policy.review.required` legal-queue event (`source: SIMULATION`) appears in the audit log.

Runs on ILLUSTRATIVE fixture packs built from file 13 candidate rows (planning/13; content marked illustrative, PENDING_LEGAL_REVIEW); the fixtures are replaced by Story 7.4 content when P-5 clears. Not gated by P-5. P-9 is resolved (DEC-50).

Source: S17.4. FR38, FR40.

### Story 7.8: Add-a-jurisdiction rehearsal

As an architect,
I want a dummy jurisdiction added as data,
So that extensibility is proven.

**Acceptance Criteria:**

**Given** a dummy pack and jurisdiction record
**When** added per planning/14 §12
**Then** there are zero diffs under `core/` outside `core/policy` data and the contract tests are unchanged.

Source: S15.5. FR42.

### Story 7.9: Sharia approval state and the `sharia_approver` role

As a Sharia approver,
I want to record Sharia approval separately from legal approval,
So that no Track B artefact reaches a customer without both.

**Acceptance Criteria:**

**Given** the overlay pack, a Track B catalogue item, a template or a reason-code text (en or ar)
**When** a `policy_author` submits it (in an ISLAMIC deployment `:submit` sets `IN_SHARIA_REVIEW` on the overlay members and on the released Track B artefacts in the same step as `IN_LEGAL_REVIEW`; contract 1.1.1) and a `sharia_approver` acts
**Then** the state moves `NOT_REVIEWED` → `IN_SHARIA_REVIEW` → `SHARIA_APPROVED` or `SHARIA_REJECTED` (reason mandatory), audited and hash-chained
**And** `sharia_approver` holds no `policy_author`, `policy_approver` or `policy_release` scope, and the vendor/developer never holds it (planning/20 SDC-17; role-scope matrix SOD-9, separation-of-duties check)
**And** per user, a Sharia approval by the maker or by the legal approver of the same pack version returns 409 `MAKER_CHECKER_VIOLATION`, and an activator who is either of them is refused (planning/21 §4.3).

**Given** a Track B pack
**When** activation is requested
**Then** `APPROVED` requires both legal and Sharia approval; while named persons are not recorded (DEC-11, DEC-63) approval and ACTIVE activation show as unavailable with that reason.

**Given** an `ILLUSTRATIVE` Track B bundle
**When** an item carries `illustrative_sharia_assumption: ASSUME_COMPLIANT` confirmed by the `policy_release` activator
**Then** it is honoured only there, and outputs add the watermark "illustrative - not a Sharia ruling"
**And** the illustrative assumption is always the separate attribute `illustrative_sharia_assumption: ASSUME_COMPLIANT`; `sharia_status` / `sharia_approval_status` never takes an illustrative value (readiness finding 5, 2026-11-10).

Source: planning/21 §4.3; DEC-55, DEC-63; AD-25. FR73. Depends on Stories 2.13, 7.1, 7.2.

### Story 7.10: Illustrative UK packs (GB, GB-ENG, GB-ENG-LDN)

As a policy author,
I want draft UK packs mapped from planning/19 rows,
So that the London cell is governed by sourced, pending policies.

**Acceptance Criteria:**

**Given** candidate rows of planning/19 (written 2026-11-10)
**When** `PACK-GB`, `PACK-GB-ENG` and `PACK-GB-ENG-LDN` are authored
**Then** each policy has a planning/19 row reference, citation and URL, status PENDING_LEGAL_REVIEW and `illustrative: true`; no legal text is invented
**And** any `illustrative_assumption` follows the Story 7.4 rule (maker sets, activator confirms).

**Given** the `A-UK` cell
**When** golden cases run (Story 7.5)
**Then** London resident, unknown, out-of-scope and cross-cell cases pass, and no US policy is borrowed.

**Given** `CTL-HUMAN-REVIEW` policies in `PACK-GB`, mirrored in `PACK-US` as internal policy (DEC-66; planning/19 U02, UQ4 [LEGAL-CONFIRM])
**When** golden cases run in either Track A cell
**Then** an offer suppression applies immediately and creates a `review_item` for staff confirmation; every retention action except `RA_NO_ACTION` is withheld until a reviewer clears it; vulnerability routing ends with a person
**And** no customer-facing text says a product is unavailable because of the subsystem
**And** a suppression-confirmation review item cannot move to `CLEARED_TO_SHOW` (409 `REVIEW_STATE_INVALID`; it ends as `REJECTED` = suppression confirmed, or `ESCALATED`), and these policies carry no `illustrative_assumption: ASSUME_NOT_APPLICABLE` (planning/21 §11A).

Prerequisite: P-10 (content scope also waits for the UK entity type, DEC-65, BLOCKED-EXTERNAL). Until then only fixtures. Source: planning/21 §2, §11A; DEC-50, DEC-65, DEC-66. FR77, FR81.

### Story 7.11: Illustrative AE packs and Sharia overlay content

As a policy author working with the Sharia function,
I want draft AE packs and overlay content mapped from planning/20 rows,
So that Track B is governed by sourced, pending policies and bank-approved Sharia lists.

**Acceptance Criteria:**

**Given** candidate rows of planning/20 (written 2026-11-10)
**When** `PACK-AE`, `PACK-AE-<emirate>` and the content of `PACK-INST-ISLAMIC` (term list, prohibited categories, catalogue eligibility attributes) are authored
**Then** every jurisdiction policy is PENDING_LEGAL_REVIEW, every overlay artefact is `NOT_REVIEWED` or `IN_SHARIA_REVIEW`, each cites its planning/20 row, and no legal or Sharia ruling is invented [LEGAL-CONFIRM] [SHARIA-CONFIRM].

**Given** the `B-AE` cell
**When** golden cases run
**Then** UAE resident, unknown, out-of-scope, cross-track, `SHARIA_REVIEW_PENDING`, terminology fallback and prohibited-category cases pass
**And** overlay content covers the planning/20 SDC-01..SDC-18 mapping in planning/21 §4.4 (e.g. `contract_type` with no conventional types, Sharia basis/ISSC/certificate/KFS refs, Sharia-question handoff, re-review on Sharia change).

**Given** `PACK-AE` `CTL-CONTACT-LIMIT` telemarketing parameters (DEC-67; planning/20 row E08 [LEGAL-CONFIRM])
**When** suggested outreach is evaluated for a `B-AE` customer
**Then** consent is checked by purpose x channel x product type x contact method x language, frequency caps apply, and in-app/push/digital messages count toward the caps until planning/20 IQ4 is answered
**And** the DNCR check is recorded as a host obligation, never performed or assumed by the subsystem.

Prerequisite: P-11. Until then only fixtures (Story 2.13). Source: planning/21 §4, §4.4; DEC-50, DEC-54, DEC-55, DEC-67. FR78, FR82.

## Epic 8: Grounded explanations and a safe assistant

*Source: planning/10 E9; planning/12 §H.1 DEC-06.*

### Story 8.1: Typed, read-only tool layer

As an orchestrator developer,
I want typed tools wrapping module outputs,
So that only controlled functions are callable.

**Acceptance Criteria:**

**Given** the fixed allow-list
**When** a tool not on it, or with an invalid schema, is called
**Then** the call is rejected and audited
**And** no tool can write, send, change accounts or decide credit.

Source: S9.1. FR44.

### Story 8.2: Template explanations with validation

As a user,
I want plain-language explanations from module outputs,
So that I understand results.

**Acceptance Criteria:**

**Given** a result
**When** `GET /v1/explanations/:result_id` is called
**Then** a template explanation is returned whose every number, product code and fact matches the tool results.

**Given** generated text with a mismatch
**When** validated
**Then** it is discarded and the template is used, with `llm_interaction.validation_result` set to FALLBACK.

Source: S9.2. FR45.

### Story 8.3: Refusals and safety

As a compliance officer,
I want the assistant to refuse out-of-scope requests,
So that it cannot overstep.

**Acceptance Criteria:**

**Given** "approve this loan", an account change, an unknown product or "is this permitted by law?"
**When** sent to `POST /v1/assistant/messages`
**Then** credit intents return 422 `OUT_OF_SCOPE_REQUEST` with the handoff template, and the others get a refusal citing policy IDs only.

**Given** the red-team and prompt-injection sets (free text treated as data)
**When** run
**Then** all pass.

Source: S9.3. FR46.

### Story 8.4: LLM port with only the template adapter enabled

As an architect,
I want a pluggable LLM port,
So that a future approved model needs no core change.

**Acceptance Criteria:**

**Given** the bootstrap
**When** started in either mode
**Then** `LLMPort` binds to the template adapter, and enabling any other adapter needs a config change that is refused unless an approval reference exists (DEC-06).

Source: S9.4. FR47.

### Story 8.5: Arabic and English templates with terminology checks

As Track B staff,
I want explanations, reason-code texts and assistant templates in Arabic and English,
So that I can use the system in either language without wording that breaks the bank's Sharia rules.

**Acceptance Criteria:**

**Given** a Track B deployment (`locales: [ar-AE, en-AE]`, `CTL-EXPLANATION.languages = [ar, en]`)
**When** an explanation is requested with `language=ar` or `language=en`
**Then** a template exists for each output type in both languages, numbers and product codes pass the same validation as English (Story 8.2), and Arabic numerals are a locale option.

**Given** a template containing a listed term (e.g. "loan" or "interest")
**When** it is loaded or rendered
**Then** template CI rejects it, and at runtime `CTL-TERMINOLOGY` blocks the text and falls back (`TERMINOLOGY_VIOLATION_FALLBACK`).

**Given** a non-demo Track B bundle
**When** a template is not `SHARIA_APPROVED`
**Then** it is not used (Story 7.9); every Arabic template also records a translation-review reference.

**Given** a Track B assistant question about whether a product, term or transaction is Sharia-compliant, or a Sharia complaint (Arabic or English)
**When** sent to `POST /v1/assistant/messages`
**Then** the approved handoff template routes it to the bank's Sharia function or complaints process, and no answer states a ruling or a compliance claim beyond the approved `sharia_status` text (planning/20 SDC-10).

Source: planning/21 §8, §4.4 (SDC-10); DEC-59, DEC-54. FR76.

## Epic 9: Standalone dashboard and manager demo

*Source: planning/10 E10; EXPERIENCE.md; planning/09 §5, §7. Visual identity gated by P-3 (UX-DR17).*

### Story 9.1: App shell, role-gated screens, accessibility and not-ready state

As a staff user,
I want an accessible dashboard that shows only my permitted screens,
So that I can work safely.

**Acceptance Criteria:**

**Given** the demo issuer roles
**When** a user signs in
**Then** only the screens permitted by the role matrix appear, and customer names never appear (UX-DR16).

**Given** an unverified policy bundle
**When** the app loads
**Then** the not-ready screen shows and no customer data loads (UX-DR14).

**Given** the WCAG 2.2 AA checks
**When** the shell is tested
**Then** keyboard navigation, visible focus and status labels pass, and colour is never the sole signal (UX-DR12).

Source: S10.1 (part). FR48. UX-DR12, UX-DR14, UX-DR16, UX-DR17.

### Story 9.2: Customer picker and customer view

As a manager,
I want a customer view with all module outputs,
So that I see end-to-end value.

**Acceptance Criteria:**

**Given** the picker fed by `GET /v1/customer-directory` (Story 6.10; access purpose set from the user's registered purpose, token lookup or narrowing filter; tokens and coarse codes only, UX-DR20) with the DEC-16 jurisdiction selector (as extended in 12 §H.7.2), and stored transactions read through `GET /v1/customers/{customer_token}/transactions` (Story 6.10)
**When** Alex is opened
**Then** transactions, profile, segments, needs (with evidence table), churn (drivers and version), recommendations and retention show, each with a decision badge, gate trace and version strip, plus the SYNTHETIC label (UX-DR1, UX-DR2, UX-DR5, UX-DR6, UX-DR7, UX-DR10)
**And** while the bundle is ILLUSTRATIVE, the watermark banner is persistent on every screen (UX-DR3, DEC-44), and no SHADOW model output appears anywhere (DEC-47).

**Given** a minor, a `US-CA` customer, an unknown-jurisdiction customer or a customer with a DQ breach
**When** opened
**Then** the matching state from UX-DR13 renders with the UX-DR4 and UX-DR15 wording.

Source: S10.1. FR48.

### Story 9.3: Review queue and outcome recording screens

As a retention reviewer,
I want queue and outcome screens,
So that I can act within the SLA.

**Acceptance Criteria:**

**Given** queued items
**When** the queue is opened
**Then** each row shows state, SLA countdown and actions, and reject or escalate needs a reason (UX-DR8).

**Given** a retention action
**When** an outcome is recorded
**Then** only fixed pick-lists are offered (UX-DR9).

Source: S8.5, S8.6 (UI part). FR24, FR25 (UI).

### Story 9.4: Audit, decision log and replay view

As a compliance reviewer,
I want the audit trail, gate results and consent state,
So that I can inspect decisions.

**Acceptance Criteria:**

**Given** a recommendation
**When** replay is requested (`GET /v1/policy/decisions/:id`, `POST /v1/policy/decisions/:id:replay`, planning/15 §2.1)
**Then** inputs reference, versions, gates, `bundle_state` and the decision are shown, and the replay outcome matches.

**Given** a user without audit scope
**When** they navigate to the view
**Then** access is denied.

Source: S10.2. FR49.

### Story 9.5: Policy pack viewer, decision view and simulator page

As a policy author or manager,
I want pack, decision and simulator views,
So that the policy state is transparent.

**Acceptance Criteria:**

**Given** packs in any state (`GET /v1/policy/packs?state=`)
**When** viewed
**Then** `pack_state`, versions, `applicability_status`, any `illustrative_assumption` and `review_due` show, with the pending and watermark banners persistent (UX-DR3).

**Given** a simulator run
**When** completed
**Then** the diff renders per UX-DR11.

Source: planning/09 §1, §7; S17.1 (UI). FR53.

### Story 9.6: Assistant panel

As a user,
I want a grounded chat,
So that I can ask why.

**Acceptance Criteria:**

**Given** the panel
**When** asking "why this product?" and "approve a loan"
**Then** the first gets a grounded template answer and the second a refusal with handoff, using only Epic 8 tools.

Source: S10.3. FR50.

### Story 9.7: Portfolio view (Could)

As an analyst,
I want aggregate segment and churn distributions,
So that I see the portfolio.

**Acceptance Criteria:**

**Given** the portfolio view backed by `GET /v1/portfolio/summary` (Story 6.10, `ri.portfolio.read`)
**When** opened
**Then** only aggregates show, with small cells suppressed (configured minimum, default 20 [ASSUMPTION]) and marked as suppressed, counts shown as rounded, and no token, name or row-level data appears (UX-DR20, DEC-62; planning/21 §10A)
**And** in Track A the view may combine the `A-US` and `A-UK` cells; it never combines tracks.

Source: S10.4. FR51.

### Story 9.8: Scripted demo pack

As the owner,
I want a seeded, scripted demo,
So that the presentation is reliable.

**Acceptance Criteria:**

**Given** a clean start with the demo seed
**When** the script for planning/09 §5 steps 1-12 and the §7 changes runs
**Then** every step completes without error, including the fail-safe, minor, pending-review, consent-withdrawal and outcome scenes
**And** the demo deployment runs with `demo=true` and ILLUSTRATIVE packs, and every scene carries the watermark (DEC-44).

Runs on ILLUSTRATIVE fixture packs built from file 13 candidate rows, upgraded to Story 7.4 content when P-5 clears (not gated by P-5); P-9 resolved (DEC-50). Same pattern as Stories 9.10 (Story 7.10 fixtures) and 9.11 (Story 7.11 fixtures). This is the `A-US` part of the Track A demo; Story 9.10 adds the London cell.

Source: S10.5. FR52.

### Story 9.9: Policy approval and model registry screens

As a policy author, approver, release operator or model admin,
I want screens for the maker-checker workflow and model status,
So that governance actions are visible and role-gated.

**Acceptance Criteria:**

**Given** the governance endpoints in planning/15 §2.1 (DEC-46; normative in OpenAPI v1, prerequisite P-4)
**When** a user acts
**Then** only their role's actions show, maker != checker is enforced, and the reason or confirmation dialogs appear
**And** while DEC-11 is unresolved, approval and ACTIVE activation show as unavailable with that reason; ILLUSTRATIVE activation is offered only in the demo deployment
**And** while DEC-19 is unresolved, ML_MODEL → APPROVED shows as unavailable, and SHADOW models show aggregate metrics only (DEC-47).

**Given** an ISLAMIC deployment and a user holding only `sharia_approver`
**When** they open the policy approval screen (S08)
**Then** only action A14 (record Sharia approval: `SHARIA_APPROVED` or `SHARIA_REJECTED` with mandatory reason, via `POST /v1/policy/packs/{pack_id}/{version}/sharia-approvals`, scope `ri.sharia.approve`) is offered, legal approval and release actions are hidden (SOD-9)
**And** while DEC-63 records no named Sharia approver, A14 shows as unavailable with that reason; in a CONVENTIONAL deployment the role and action do not appear.

Source: EXPERIENCE.md IA (Policy approval, Model registry). FR35, FR36, FR16 (UI), FR73.

### Story 9.10: Track A demo script (NYC and London cells)

As the owner,
I want the Track A demo to show both cells and their isolation,
So that the manager sees NYC and London served by one profile without data crossing.

**Acceptance Criteria:**

**Given** a clean start with `PROFILE-TRACK-A`, `demo=true`, both cell seeds and ILLUSTRATIVE packs
**When** the script from planning/21 §11 (Track A) runs after the Story 9.8 scenes (which run on file 13 ILLUSTRATIVE fixture packs, not on P-5)
**Then** a London customer flows through the same screens in en-GB and GBP, capabilities show `cell_id: A-UK` and GB packs PENDING_LEGAL_REVIEW with illustrative assumptions, and every output carries the watermark
**And** the London token is not found in `A-US`, a NYC-resident London-booked customer is denied commercial activity (UNKNOWN), and the portfolio view shows aggregates across both cells only.

**Given** the app shell
**When** any screen is open
**Then** the track and cell indicator is visible (UX-DR18).

Prerequisite: Story 7.10 fixtures and the Story 9.8 fixture packs; real content upgrades when P-5 and P-10 clear, neither gates the demo. Source: planning/21 §11; DEC-64. FR79, UX-DR18, UX-DR20.

### Story 9.11: Track B dashboard locale and demo script

As the owner,
I want a separate Track B demo in Arabic and English,
So that the manager sees the Islamic-bank track on the same core.

**Acceptance Criteria:**

**Given** a clean start of the separate Track B deployment (`PROFILE-TRACK-B`, own URL, realm and keys, `demo=true`)
**When** the script from planning/21 §11 (Track B) runs
**Then** capabilities show `institution_type: ISLAMIC` and `PACK-INST-ISLAMIC`; the customer view switches between Arabic (right-to-left, mirrored navigation) and English; suggestions come only from `CAT-B-AE-ISLAMIC-SYNTH`; Mudarabah and Takaful items enter the review queue; financing items appear only as `CREDIT_REFERRAL`
**And** the terminology fallback, prohibited-category exclusion with neutral budgeting wording, and a `SHARIA_REVIEW_PENDING` block are shown, with the extra watermark "illustrative - not a Sharia ruling" (UX-DR18)
**And** a `GB` jurisdiction is UNSUPPORTED, a Track A token is unknown, the minor scene uses the AE threshold, and "approve my financing" is refused with the handoff template
**And** every product suggestion and credit referral card shows its Sharia basis, certificate and KFS links in the active language from `sharia_disclosure` (Story 6.1) with the "illustrative - not a Sharia ruling" notice, and a card without a complete disclosure is never shown (planning/20 SDC-11).

**Given** both directions
**When** checked
**Then** WCAG 2.2 AA holds in right-to-left and left-to-right layouts (UX-DR19).

Prerequisite: Story 7.11 fixtures (same fixture pattern as Stories 9.8 and 9.10; real content upgrades when P-11 clears, it does not gate the demo). Source: planning/21 §8, §11; DEC-59, DEC-64. FR76, FR79, UX-DR18, UX-DR19.

## Epic 10: Plugin contract v1 and host parity

*Source: planning/10 E11, E19, S14.7; planning/15. Host facts gated by DEC-30 and DEC-33 (safe assumptions used).*

### Story 10.1: Contract v1 publication

As a host developer,
I want the published contract with examples,
So that I can integrate.

**Acceptance Criteria:**

**Given** `contracts/openapi/retail-intelligence-v1.yaml` and `contracts/events/` from P-4
**When** published
**Then** sample payloads, a changelog and the role matrix ship with it, the CI breaking-change diff is active, and no credit-decision type exists.

Source: S11.1, S19.1. FR54.

### Story 10.2: OIDC identity adapter and claims

As a security officer,
I want host identity honoured through a port,
So that we hold no authority of our own.

**Acceptance Criteria:**

**Given** a generic OIDC adapter with a mock IdP and configured claim mapping (DEC-30 safe assumption)
**When** a token has a wrong `ri_deployment` or `ri_jurisdictions`
**Then** the response is 403 `JURISDICTION_NOT_PERMITTED`
**And** an expired token is rejected, and denied access returns no data.

Source: S11.3. FR56.

### Story 10.3: Host data mapper and boundary minimisation

As a privacy officer,
I want host payloads mapped to canonical data with minimum fields,
So that excess data never enters.

**Acceptance Criteria:**

**Given** a host-shaped payload containing ZIP or fine location
**When** mapped
**Then** only the resolved jurisdiction code is kept (DEC-34), and fields outside the activity allow-list are rejected in strict mode.

Source: S19.4. FR60.

### Story 10.4: Host overlay tighten-only

As an integrator,
I want stricter host overlays applied and looser ones rejected,
So that plugin mode is never looser.

**Acceptance Criteria:**

**Given** a signed stricter overlay through `PUT /v1/policy/overlays/:id`
**When** merged
**Then** the stricter controls apply under the combining rules.

**Given** an overlay that loosens a control
**When** merged
**Then** the loosening part is rejected and `policy.overlay.rejected` is emitted.

Source: S14.7. FR32.

### Story 10.5: Host simulator and parity suite

As a tester,
I want a mock host driving the core through contract v1,
So that integration is proven without the bank platform.

**Acceptance Criteria:**

**Given** the golden persona set
**When** run through the standalone path and the host simulator
**Then** the outputs and policy outcomes are identical.

**Given** the simulator scenarios (consent withdrawal, out-of-scope token, replayed request, malformed or duplicate transactions, an ILLUSTRATIVE pack offered to the plugin deployment)
**When** run
**Then** the expected errors or denials occur, and the plugin deployment refuses the ILLUSTRATIVE pack at load (DEC-44)
**And** the parity comparison injects the same signed fixture bundle into both wirings through a test-only `PolicyStorePort` double (a CI test seam, never a deployment adapter). The bundle is ACTIVE-shaped (`illustrative: false`), signed with a TEST-class key that only the double trusts, and holds synthetic placeholder policies (planning/15 §13). The load guard itself is proven in Story 2.2.

Source: S11.2 = S19.3. FR57.

### Story 10.6: Plugin outcome endpoints

As a host developer,
I want to report recommendation and retention outcomes and review results,
So that effectiveness and review are tracked in plugin mode.

**Acceptance Criteria:**

**Given** `POST /v1/recommendations/:rec_id/outcomes` and `POST /v1/retention-actions/:action_id/outcomes` with scope `ri.recommendations.feedback` / `ri.retention.feedback`
**When** called by the host
**Then** the outcomes are stored append-only with source HOST, review results follow the `review_item` rules, and the contract tests pass.

Source: S19.5. FR58.

### Story 10.7: Outbox events (Could)

As a host developer,
I want CloudEvents for outcomes and policy changes,
So that I can react without polling.

**Acceptance Criteria:**

**Given** a customer-outcome state change (recommendation or retention outcome, review transition), a consent change applied, or a committed ingest batch (`ingest.completed`, Story 3.6)
**When** committed
**Then** a CloudEvents 1.0 row validated against `contracts/events/` is written to the existing outbox (from Story 7.2) in the same transaction, with no broker dependency (DEC-33 safe assumption), and a transport-neutral publisher port drains it.

Source: S19.4 (events). FR59.

## Epic 11: Privacy, resilience and evaluation readiness

*Source: planning/10 E12; planning/07 §8; planning/08 SEC-20.*

### Story 11.1: Retention lifecycle hooks

As a data protection officer,
I want configurable retention per entity,
So that policies can be applied once legal sets the values.

**Acceptance Criteria:**

**Given** `CTL-RETENTION` entries with no approved values (DEC-20)
**When** the purge job runs
**Then** it reports only and deletes nothing, and no value is hard-coded.

Source: S12.5. FR64.

### Story 11.2: DSR erasure with hash-chained logs

As a data protection officer,
I want deletion honoured without breaking audit integrity,
So that DSR and tamper evidence both hold.

**Acceptance Criteria:**

**Given** an approved synthetic DELETE request through `POST /v1/dsr-requests`
**When** executed
**Then** the operational rows are deleted or anonymised, the `subject_key` is DESTROYED, a `dsr_request` row and an "erasure executed" audit record are appended, and the hash chain still verifies (DEC-35; legal sufficiency [LEGAL-CONFIRM]).

Source: S12.6. FR65.

### Story 11.3: Backup and restore

As an operator,
I want encrypted backups and a tested restore,
So that the subsystem can recover.

**Acceptance Criteria:**

**Given** a backup of the database, decision log and signed packs
**When** restored
**Then** the instance stays not-ready until the hash chains and pack signatures verify, and destroyed subject keys stay destroyed. RPO/RTO come from config (DEC-36: demo none; pilot 24 h / 8 h [ASSUMPTION]).

Source: S12.7. FR66.

### Story 11.4: Reason-code library and operator runbooks

As an operator and compliance owner,
I want a governed reason-code library and runbooks,
So that operation and explanations are governed.

**Acceptance Criteria:**

**Given** the library
**When** changed
**Then** a new version is created, DEV drafts it and COMP approval is recorded (DEC-40).

**Given** the runbooks for pack activation and rollback, signing-key rotation and fail-closed recovery
**When** a rehearsal is run on synthetic data
**Then** each procedure completes.

Source: S12.8. FR67.

### Story 11.5: Regression suite, security baseline and technical evaluation report

As a manager,
I want a full regression suite and an evaluation report,
So that I can decide on a pilot.

**Acceptance Criteria:**

**Given** a clean checkout
**When** the suite runs
**Then** the unit, contract, scenario, policy golden, safety and role-matrix tests pass, and the threat model and control tests from S12.1 are present.

**Given** the demo seed
**When** the report is generated
**Then** it states BQ-11 KPIs 1-8 with results, risks and synthetic-data limits, and makes no business-performance claims
**And** KPI 3 reports the SHADOW ML metrics against the serving rule baseline, as "eligible for MRM review" or "not eligible", never as "adopted" (DEC-47).

Source: S12.1, S12.3, S12.4. FR68.

### Story 11.6: Performance, observability and supply-chain baselines

As an operator,
I want the cross-cutting NFRs measured in CI,
So that the demo meets its targets and the build is trustworthy.

**Acceptance Criteria:**

**Given** the demo dataset
**When** the performance suite runs
**Then** evaluate p95 is under 1 s (NFR1), PDP p95 is under 5 ms (NFR2) and a 50k-customer batch finishes in 30 min or less (NFR3).

**Given** a running instance
**When** inspected
**Then** logs are PII-free JSON, OpenTelemetry traces and `/metrics` exist, and the SEC-21 alert rules are defined (NFR7).

**Given** a build
**When** CI runs
**Then** dependencies are pinned, the SBOM is produced, the image and packs are signed, the vulnerability scan runs, and rate and size limits are enforced at the defaults in config (NFR10, NFR12).

Source: planning/05 §9; planning/08 SEC-05, SEC-10, SEC-17, SEC-21. NFR1, NFR2, NFR3, NFR7, NFR10, NFR12.

### Story 11.7: Cell and track isolation suite

As a security officer,
I want automated proof that nothing crosses a cell or a track,
So that the separation in DEC-52 and DEC-53 is tested, not assumed.

**Acceptance Criteria:**

**Given** the Track A demo (two cells) and the Track B demo deployment
**When** the isolation suite runs in CI
**Then** it proves that no personal data, token, subject key, decision or audit record, model, feature set or training set appears in another cell or track; that a Track A token, key, pack or model reference is rejected by Track B and vice versa; and that the only cross-cell read is the aggregate portfolio path
**And** the Track B deployment makes no outbound network call outside the bank boundary (NFR15), and a failing case blocks the release.

Source: planning/21 §2, §9; DEC-52, DEC-53, DEC-57; AD-23, AD-24. FR71, NFR14, NFR15.

## Change log

- 2026-11-09 (readiness fixes, planning/12 §H.6): FR13-FR16 and FR35-FR36 inventory lines updated. New prerequisite P-9 for the target market (DEC-50). UX-DR3 adds the watermark banner. Acceptance criteria added or changed in Stories 2.2, 2.4, 2.5 and 2.8 (pack lifecycle, ILLUSTRATIVE guard, empty L0 whitelist, envelope watermark; DEC-44, DEC-45), Story 4.4 (group size 100; DEC-48), Stories 5.1, 5.2 and 5.4 (RULESET/ML_MODEL, SHADOW, no credit-building need; DEC-47, DEC-49), Stories 7.1, 7.2, 7.4, 7.6 and 7.7 (pack endpoints, ILLUSTRATIVE activation, P-9; DEC-44..46), Stories 9.2, 9.4, 9.5, 9.8 and 9.9 (watermark, replay endpoints, governance screens), Story 10.5 (illustrative refusal; parity fixture seam) and Story 11.5 (KPI 3 wording). There are no new stories and the count stays at 71. After the independent re-check: pack endpoints moved from 7.1 to 7.6 (no forward dependency), P-9 now gates 7.7 and 9.8, the assumption check is made at activation, simulation emits the legal-queue event, the parity fixture is specified, and stale wording is fixed in the epic summaries for Epics 5 and 7 and in FR40.
- 2026-11-09 (PH-3 contract specification, readiness P-4/R-4): new Story 2.10 (consent and preference ingest endpoints) and Story 3.6 (bulk data ingest batches), sourced from new planning/10 S19.6 and S19.7; count now 73. Contract references point to the normative `contracts/openapi/retail-intelligence-v1.yaml` and `contracts/events/` (Additional Requirements, P-4 status, Story 10.1); Story 6.8 cites Story 2.10; Story 10.7 adds consent and ingest events and schema validation.
- 2026-11-10 (PH-3, readiness P-4): P-4 status updated (ERD, ports, role matrix delivered in `contracts/`); Story 2.7 reads `contracts/role-scope-matrix.json` (393 negative tests, one per deny cell). Stories 9.2 and 9.7 depend on GATE-3 decisions G-RS-1..G-RS-3 (customer list, stored transactions, portfolio scope).
- 2026-11-10 (two-track scope, planning/21, 12 §H.7): DEC-50 resolved; P-1 and the GATE-3 part of P-4 done by developer delegation; P-9 resolved and replaced by P-10 (UK, planning/19) and P-11 (UAE + Sharia, planning/20); P-5, P-6 updated. FR69-FR80, NFR14-NFR15 and UX-DR18-UX-DR20 added with coverage. New stories 1.5, 2.11, 2.12, 2.13, 5.7, 6.10, 7.9, 7.10, 7.11, 8.5, 9.10, 9.11, 11.7 (count 73 -> 86); Stories 7.4, 7.7, 9.2, 9.7, 9.8 and epic summaries updated. Story 6.10 precedes 9.2 and 9.7; no forward dependency added.
- 2026-11-10 (alignment with planning/19 and planning/20, written 2026-11-10): 19/20 cited as written; Story 7.9 SoD per SDC-17; Stories 2.13 and 7.11 cite the SDC-01..SDC-18 mapping (planning/21 §4.4); Story 7.10 adds FR81 ACs (DEC-66) and the DEC-65 gate; Story 7.11 adds FR82 ACs (DEC-67). Story count unchanged (86); FR81, FR82 in inventory, Epic 7 coverage and the coverage map.
- 2026-11-10 (readiness re-run fixes): Story 7.9 Sharia submit transition (`:submit` sets IN_SHARIA_REVIEW in ISLAMIC deployments) and the attribute form `illustrative_sharia_assumption: ASSUME_COMPLIANT` recorded; Story 9.9 Sharia approval AC (S08/A14); Stories 7.7, 9.8, 9.10, 9.11 run on ILLUSTRATIVE fixture packs (demo gating symmetric; P-5/P-10/P-11 upgrade content only); Stories 2.11/2.12 use `DeploymentProfilePort.verify_artefact_track` fixtures; Epic 1 and Epic 7 retitled per cell / per track.
- 2026-11-10 (independent two-track validation, handoff/two-track-validation-report-2026-11-10.md): ACs tightened in Stories 2.11 (profile fail-closed cases, no env/flag override), 2.12 (no component reads another cell's store; isolation checklist and token scheme), 6.10 (aggregate disclosure control AG-1..AG-10 with differencing tests), 7.9 (per-user Sharia SoD), 7.10 (suppression review cannot release; no not-applicable assumption), 8.5 (SDC-10 Sharia-question handoff) and 9.7 (rounded counts). Story count unchanged (86).
- 2026-11-10 (contract 1.2.0, V-8 and V-9): FR80 and UX-DR20 use the purpose-bound customer directory (DEC-68); Story 6.10 ACs for purpose, narrowing, bounds, quotas, denial codes, audit and the deprecated `listCustomers`; Story 9.2 picker; Story 6.1 SDC-11 disclosure ACs (FR72 added to its source); Story 9.11 disclosure rendering AC. Story count unchanged (86).
