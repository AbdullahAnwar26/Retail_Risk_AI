# 10 - Epics and Initial User Stories (generic BMAD-style)

Note: BMAD is not yet installed. This file uses a generic epic / story ("As a ... I want ... so that ...") / acceptance-criteria format. No BMAD commands or paths are assumed. Once approved and BMAD is installed, these will be re-homed into BMAD's own artifact structure and conventions (file 17). Labels: [DOC p.N] [REC] [ASSUME] [UNKNOWN] [MANAGER] [LEGAL-CONFIRM] legal/compliance must confirm. Priority: M/S/C/W per 09-mvp-scope. Stack choices are open (see architecture docs); stories are stack-neutral. Legal/regulatory content: to be confirmed by legal/compliance.

ID conventions: stories are `S<epic>.<n>` (e.g. story S8.5); file 13 rows are always written "file 13 row S12" and file 13 legal questions "file 13 §6 Q4"; phases `PH-x` and gates `GATE-x` come from file 11; decisions `DEC-xx` from file 12 §A.

Epic-to-phase map: E1=PH-1, E2=PH-2, E3=PH-3, E4=PH-4, E5=PH-5, E6=PH-6, E7=PH-7, E8=PH-8, E9=PH-9, E10=PH-10, E11=PH-11, E12=PH-12. NYC scope update adds E13=PH-2A, E14=PH-3A, E15=PH-3A/PH-4A, E16=PH-4A, E17=PH-8A, E18=PH-6/PH-8A, E19=PH-3/PH-11 (file 11). Initial market is New York City (US federal + NY State + NYC only); no 50-state rulebooks in MVP; other states and UK are future expansion [MANAGER confirm]. Not legal advice; all legal points [LEGAL-CONFIRM].

Minors (all epics): the two-tier block list in file 14 §13.3 applies. Tier 1 (`SEGMENTATION_COMMERCIAL`, `CHURN_PREDICTION`, `PRODUCT_RECOMMENDATION`, `MARKETING_ELIGIBILITY`, `RETENTION_SUGGESTION`, `CREDIT_PRODUCT_REFERRAL`, `MODEL_TRAINING`) always BLOCK and is locked; Tier 2 (`PROFILE_BUILD`, `NEEDS_PREDICTION`, `SEGMENTATION_INSIGHT`, `EXPLANATION_GENERATION`, `ASSISTANT_QA`) BLOCK unless a signed `MINOR-APPROVAL-*` record allows a named non-commercial educational use; `TXN_ENRICHMENT` is limited to age/routing checks.

---
## E1 Project discovery and business requirements (PH-1) [Must]
Goal: approved requirements and open-question closure before design lock.
- **S1.1 Scope approval.** As a project owner, I want the subsystem scope, boundaries and assumptions approved, so that we build only our subsystem. AC: 02-retail-subsystem-scope signed off by manager; each [UNKNOWN] has an owner; credit exclusions confirmed. Priority M.
- **S1.2 Define business definitions.** As a product analyst, I want written definitions of churn, blue-collar, Gen Z, financial need categories, retention actions and retention outcomes, so that models have clear targets. AC: definitions documented (DEC-04, DEC-43; file 12 §G.3 BQ-1..BQ-7); legal/compliance review requested for blue-collar and any sensitive-adjacent signal; manager approval. M.
- **S1.3 Obtain missing source material.** As a developer, I want prd.md (FR-39, FR-50/51, FR-79..86), the manager's presentation and any repo/convention docs, so that shared requirements are exact. AC: documents received or gaps logged; shared requirement register created. M.
- **S1.4 Requirements baseline.** As a stakeholder, I want FR/NFR lists for the subsystem, so that stories trace to requirements. AC: IDs assigned, each traced to [DOC]/[MANAGER]/[REC] label. M.

## E2 Technology compatibility and architecture (PH-2) [Must]
- **S2.1 Stack evidence gathering.** As an architect, I want to ask the team for the host stack plans, so that choices are compatible. AC: question list sent; answers or "none yet" recorded; main platform confirmed to have no implementation. M.
- **S2.2 Technology compatibility report.** As a reviewer, I want each technology justified (purpose, need, host usage, compatibility, complexity), so that we avoid arbitrary choices. AC: report with conditional recommendations; no microservices without justification. M.
- **S2.3 Standalone vs plugin architecture.** As an architect, I want one shared core with two thin shells, so that logic is never duplicated. AC: diagram and decision record; integration option comparison (library, REST, events) with simplest-adequate recommendation. M.
- **S2.4 Architecture approval gate.** As a manager, I want to review and approve architecture, so that implementation can begin. AC: written approval or change list (GATE-2). M.

## E3 Database and integration contract design (PH-3) [Must]
- **S3.1 Data model.** As a data engineer, I want entities (customer, account, transaction, goal, catalogue, features, segments, predictions, recommendations, retention actions and outcomes, model versions, audit, consent) with money types and deployment-scope and jurisdiction-context partitioning, so that data is correct and separable. AC: ERD; decimal/minor-unit money; currency per record; jurisdiction/deployment-scope partition rule (no jurisdiction literals in core), policy tables per file 14 section 3; retention fields. M.
- **S3.2 Contract v1.** As an integrator, I want versioned input/output schemas (customer, transactions in; profile, segments, needs, churn, recommendations out), so that a host can integrate. AC: OpenAPI/JSON Schema-style spec; versioning and deprecation policy; error model. Extended by story S19.1. M.
- **S3.3 Adapter interfaces.** As a developer, I want ports for identity, data source, catalogue, consent, audit sink, so that the core has no host dependency. AC: interface list; standalone and mock implementations specified. M.
- **S3.4 Credit-boundary interface (draft).** As an architect, I want a documented placeholder for reading segment/tags and suppression flags (file 15 §11 CI list), so that shared needs are not lost. AC: marked [UNKNOWN]; optional in v1. S.

## E4 Synthetic data generation (PH-4) [Must]
- **S4.1 Customer and account generator.** As a tester, I want reproducible synthetic NYC-market adult customers (plus cross-jurisdiction and unknown-jurisdiction test records), so that demos and tests are repeatable. AC: seeded; realistic distributions assumed and labelled [ASSUME]; no real data; includes few age 16-17 protected records for guard tests. M.
- **S4.2 Transaction generator with edge cases.** As a tester, I want duplicates, pending, reversals, refunds, internal transfers, multi-currency and irregular income patterns, so that transaction logic is stress-tested. AC: configurable rates; ground-truth labels. M.
- **S4.3 Catalogue and consent data.** As a product tester, I want a synthetic NYC-market catalogue and consent records, so that eligibility/consent gates are testable. AC: catalogue files carry `jurisdictions_offered`; market reference data versioned (NYC only in MVP; other jurisdictions added later as reference data); marked synthetic, not real terms. M.
- **S4.4 Behavioural scenarios.** As a data scientist, I want churn and need scenarios embedded with time structure, so that models can be evaluated without leakage. AC: time-stamped event histories; documented generation assumptions; limits of synthetic validity stated; validation report produced (distribution sanity statistics, edge-case coverage, label-leakage audit, persona balance) and reviewed at GATE-4. M.

## E5 Transaction intelligence (PH-5) [Must]
- **S5.1 Normalisation and dedupe.** As an analyst, I want duplicate and pending items handled, so that balances and totals are right. AC: tests on fixtures; idempotent processing. M.
- **S5.2 Transfers, refunds, reversals, FX.** As an analyst, I want internal transfers paired and refunds/reversals linked, so that spending is not overstated. AC: fixture tests; currency never mixed in sums. M.
- **S5.3 Categorisation.** As a customer-insight user, I want categories with confidence and rule trace, so that spend mix is explainable. AC: rules first; precision/recall reported on synthetic labels; no sensitive-trait inference. M.
- **S5.4 Income, expense, savings, recurring.** As a user, I want salary/bills/recurrence and savings rate, so that profiles are accurate. AC: documented calculation; golden test cases. M.
- **S5.5 Data-quality rules [REC].** As a data owner, I want data-quality dimensions and thresholds (completeness, minimum months of history per module, currency validity, duplicate ratio, stale-pending ratio) applied at ingest and before each module, so that no output is built on thin or broken data. AC: rules per file 07 §4.1; threshold breach returns `DATA_QUALITY_INSUFFICIENT` and no output; thresholds [ASSUME] approved under DEC-39; ingest rejection criteria beyond schema documented. S.

## E6 Customer intelligence and segmentation (PH-6) [Must]
- **S6.1 Customer profile and features.** As a banker, I want a profile with health indicators, so that I understand the customer. AC: feature dictionary; reproducible; data-quality flags per story S5.5. M.
- **S6.2 Multi-membership segmentation.** As an analyst, I want a customer to belong to several segments with reasons, so that segments can overlap. AC: Gen Z 18-29 and blue-collar rules per approved definitions (DEC-04); reasons shown. M.
- **S6.3 Minors protection.** As a compliance officer, I want ages 16-17 routed to a protected path with no commercial profiling, so that minors are safe. AC: aligned to the file 14 §13.3 two-tier list: guard test proves no Tier-1 output (commercial segments, churn, recommendations, marketing eligibility, retention, credit referral, model training) is ever produced, and no Tier-2 output (profile, needs, insight segments, explanations, assistant) is produced without a signed `MINOR-APPROVAL-*` record; a config flag alone is insufficient; audit records the block. M.
- **S6.4 Segment fairness view.** As a compliance officer, I want output distributions by segment, so that disparities are visible. AC: report; proxy review note; no sensitive attribute inference; method per file 08 CMP-09 and DEC-21 (metrics such as surfacing rate, suppression rate and impact ratio; minimum group size; group proxies only from a restricted store, never inferred; cadence per batch; escalation to MRM). S.

## E7 Prediction models (PH-7) [Must]
- **S7.1 Needs rules.** As a user, I want rule-based needs with evidence, so that suggestions are explainable. AC: each need has evidence and rule version. M.
- **S7.2 Churn baseline and model.** As a data scientist, I want rule baseline then a transparent ML model with time-split evaluation, so that ML is justified. AC: leakage checks; calibration; ML adopted only if it beats baseline; metrics by subgroup. M.
- **S7.3a Model registry.** As an owner, I want model versions, parameters and approval status DRAFT/APPROVED/RETIRED, so that `CTL-MODEL-APPROVED-ONLY` can be enforced. AC: version and status stored on every prediction; non-approved model blocked by the PEP. M.
- **S7.3b Drift and monitoring view.** As an owner, I want a basic drift indicator per model, so that changes are visible. AC: simple drift indicator per batch. S.
- **S7.4 Prediction explanations.** As a user, I want top drivers per prediction, so that it is explainable. AC: driver output validated; stable between runs. M.

## E8 Product recommendation engine (PH-8) [Must]
- **S8.1 Catalogue-grounded candidates.** As a banker, I want recommendations only from the catalogue, so that no product is invented. AC: tests prove every output id exists in the catalogue version and its `jurisdictions_offered` includes the customer's resolved jurisdiction. M.
- **S8.2 Eligibility and gating.** As a compliance officer, I want gates for consent, jurisdiction (PDP), minor protection, suppression (incl. credit-platform "pause offers"), so that only allowed offers show. AC: each gate has failing-case test; gate trace stored; eligibility rules configured as data, not invented (to be confirmed by legal/compliance); no affordability assessment by us. M.
- **S8.3 Ranking and reason codes.** As a customer-facing staff user, I want ranked products with reasons, so that I can explain them. AC: reason codes from the fixed, versioned library (DEC-40). M.
- **S8.4 Retention suggestions.** As a retention officer, I want action suggestions and priority list from churn and needs, so that I can help at-risk customers. AC: actions from fixed playbook; no automatic contact. S.
- **S8.5 Human review queue [REC].** As relationship staff, I want items flagged `requires_human_review` in a queue with approve-to-show / reject and a mandatory reason, so that review obligations are met. AC: standalone: dashboard queue with SLA indicator (DEC-37); plugin: `review_required` flag plus outcome via `POST /v1/recommendations/{rec_id}/outcomes` (file 15); flagged items withheld until reviewed; decisions audited (file 08 CMP-14). S.
- **S8.6 Retention outcome capture [REC].** As a retention officer, I want to record the outcome of each retention action (NOT_ACTIONED / CONTACTED / RETAINED / CLOSED / UNKNOWN, reason, time, source HOST or STAFF), so that retention effectiveness can be tracked. AC: outcome fields on `retention_action` (file 07); values per DEC-43; outcomes audited; outcomes never trigger automatic contact; used for reporting only (uplift modelling deferred, file 09 §2). S.

## E9 AI orchestration (PH-9) [Must]
- **S9.1 Tool layer.** As an orchestrator developer, I want typed tools wrapping modules, so that the LLM only calls controlled functions. AC: schema validation; unauthorised tool calls rejected. M.
- **S9.2 Grounded explanations.** As a user, I want plain-language explanations built from module outputs, so that I understand results. AC: every number verified against computation; mismatch triggers template fallback. M.
- **S9.3 Refusals and safety.** As a compliance officer, I want the assistant to refuse credit decisions, account changes and unknown products, so that it cannot overstep. AC: red-team test set passes; prompt-injection tests. M.
- **S9.4 LLM runtime abstraction.** As an architect, I want a pluggable local/approved LLM provider with offline fallback, so that data stays inside the bank. AC: provider interface; no mandatory external calls (DEC-06). S.

## E10 Standalone dashboard and complete workflows (PH-10) [Must]
- **S10.1 Dashboard core.** As a manager, I want a customer view with transactions, segments, needs, churn, recommendations, retention, so that I see end-to-end value. AC: policy decision and jurisdiction context shown per output; test-jurisdiction selector (NYC, out-of-scope, unknown); demo script runs without errors; keyboard navigation and contrast checks pass (WCAG 2.2 AA target, DEC-41). M.
- **S10.2 Audit and controls view.** As a compliance reviewer, I want audit trail, gate results and consent state, so that I can inspect decisions. AC: replay of a recommendation shows inputs, versions, gates; screen access per role-to-scope matrix (file 08 SEC-19). M.
- **S10.3 Assistant panel.** As a user, I want a grounded chat, so that I can ask why. AC: uses E9 tools. S.
- **S10.4 Portfolio view.** As an analyst, I want aggregate analytics, so that I see segment/churn distribution. AC: aggregates only. C.
- **S10.5 Demo pack.** As the owner, I want a scripted demo with seeded customers, so that presentation is reliable. AC: matches 09 narrative. M.

## E11 Plugin integration and host compatibility (PH-11) [Should]
- **S11.1 Contract publication.** As a host developer, I want contract v1 documentation and examples, so that I can integrate. AC: versioned spec, sample payloads, changelog. Extended by story S19.1. S.
- **S11.2 Reference adapters and mock host.** As a tester, I want a mock host driving the core through contract v1, so that integration is tested without the bank platform. AC: result parity with standalone on same input. Same as story S19.3; merge at BMAD re-homing. S.
- **S11.3 Identity and consent pass-through.** As a security officer, I want host identity and consent honoured via ports, so that we do not make our own authority. AC: tests with mock host identity; denied access returns no data. S.
- **S11.4 Host adaptation plan.** As an architect, I want a mapping checklist for when the host stack is known, so that integration cost is predictable. AC: checklist; open risks logged. S.

## E12 Security, compliance, testing and technical evaluation (PH-12) [Must]
- **S12.1 Security baseline.** As a security officer, I want tokenised IDs, encryption, access control and view logging, so that data is protected. AC: threat model; control tests (DOC p.8, 40 patterns); role-to-scope matrix (file 08 SEC-19: roles x API scopes x dashboard screens, deny by default; DEC-38) implemented and tested. M.
- **S12.2 Compliance review pack.** As a compliance officer, I want a document of open legal items and controls built, so that legal/compliance can confirm. AC: list of items "to be confirmed by legal/compliance" with owners. M.
- **S12.3 Test strategy and regression suite.** As a developer, I want unit, contract, scenario and safety tests, so that changes are safe. AC: suite runs from clean checkout; coverage of gates. M.
- **S12.4 Technical evaluation report.** As a manager, I want results, limitations and next steps, so that I can decide on pilot. AC: metrics, risks, synthetic-data limits stated. M.
- **S12.5 Retention and data lifecycle hooks.** As a data protection officer, I want configurable retention and deletion hooks, so that policies can be applied. AC: values configurable, none hard-coded; actual periods to be confirmed by legal/compliance (DEC-20). S.
- **S12.6 DSR erasure with hash-chained logs [REC].** As a data protection officer, I want deletion/objection requests honoured without breaking the append-only, hash-chained audit and decision logs, so that both DSR and audit integrity hold. AC: personal rows deleted or anonymised; audit and decision-log rows hold only `subject_ref` tokens; destroying the token mapping (crypto-shredding) makes them unlinkable; hash chain still verifies after erasure; method per file 07 §8 and DEC-35 [LEGAL-CONFIRM]. S.
- **S12.7 Backup and restore [REC].** As an operator, I want encrypted backups of the database, decision log and signed packs with a tested restore, so that the subsystem can recover. AC: restore test verifies the audit hash chain; RPO/RTO per DEC-36 (standalone demo: none); per file 08 SEC-20. S.
- **S12.8 Runbooks and reason-code library [REC].** As an operator and compliance owner, I want an operator runbook (pack activation and rollback, signing-key rotation, fail-closed recovery), a user guide, and a versioned reason-code library with a named owner, so that operation and explanations are governed. AC: documents versioned; reason-code library owner COMP (DEC-40); changes go through review. S.

---
## Story-count and ordering
E1-E12: 12 epics, 57 stories. Suggested order follows phases. Epic dependencies: E1 -> E2 -> E3 -> E4 -> E5 -> E6 -> E7 -> E8 -> E9 -> E10; E11 after E3 contracts and E8; E12 runs continuously with a final gate.

---
# NYC scope update: new epics (E13-E19)
Source documents: file 13 (applicability matrix, rows F01-F18, S01-S17, C01-C07), file 14 (policy layer, control IDs), file 15 (contracts). No story assumes a law applies; applicability is decided by the bank's legal/compliance teams [LEGAL-CONFIRM].

## E13 Regulatory applicability assessment (PH-2A) [Must]
Goal: legal/compliance decides which file 13 rows apply, given charter, status, products and activities (all [UNKNOWN]).
- **S13.1 Institution profile intake.** As a project owner, I want the bank's charter, primary regulator, Part 500 status, asset-size thresholds, products and channels recorded, so that applicability can be decided (file 13 §2.1 Q-C1..Q-C8, §2.2 Q-P1..Q-P9; §6 Q1-Q3; DEC-10). AC: profile completed by legal/compliance or gap logged with owner; nothing assumed. M.
- **S13.2 Row-by-row applicability decision.** As a legal approver, I want each file 13 row marked APPLICABLE / NOT_APPLICABLE / PENDING_LEGAL_REVIEW with a signed record, so that packs only cite decided rows. AC: decisions stored as data; only the legal approver can set status; developers cannot mark NOT_APPLICABLE. M.
- **S13.3 Priority question closure.** As a manager, I want the top unresolved questions (file 12 §G.2) assigned owners and gates, so that blocking items do not stall PH-4A. AC: owner per question; gate named. M.
- **S13.4 Change monitoring.** As a compliance officer, I want items likely to change (file 13 §7) tracked with review dates, so that packs are re-reviewed. AC: `review_due` set per policy; expiry treated as PENDING_LEGAL_REVIEW. S.

## E14 Jurisdiction policy layer (PH-3A) [Must]
- **S14.1 Jurisdiction hierarchy and context.** As an architect, I want jurisdictions as data (`*`, `US`, `US-NY`, `US-NY-NYC`) and a customer jurisdiction context, so that no core code names a state. AC: CI lint forbids jurisdiction literals in `core/` outside the policy package. M.
- **S14.2 Policy data model and control catalogue.** As a developer, I want policy records and the control catalogue with parameter schemas, so that policies reference mechanisms, not code. AC: schema validation; catalogue versioned (file 14 sections 3, 5). M.
- **S14.3 PDP, resolution and conflict rules.** As a compliance engineer, I want a PDP in M8 resolving applicable policies with deterministic conflict handling, so that outcomes are repeatable. AC: stricter-wins and deny-overrides tests (DEC-26); INDETERMINATE -> DENY with conflict record. M.
- **S14.4 PEPs in all modules.** As a compliance officer, I want every module to call the PDP before and after processing, so that no output exists without a `decision_id`. AC: type-level requirement of `decision_id`; module skip test fails. M.
- **S14.5 Fail-safe behaviours.** As a compliance officer, I want restrictive defaults for pending, unknown or unsupported jurisdiction, unknown age, unknown consent, no matching policy, bad signature and PDP error, so that uncertainty never permits. AC: one test per row of file 14 section 10 (DEC-14, DEC-24). M.
- **S14.6 Decision log and replay.** As an auditor, I want every policy decision logged with pack versions, so that a recommendation can be replayed. AC: replay reproduces decision; log holds policy IDs and statuses only. M.
- **S14.7 Standalone and plugin parity.** As an integrator, I want the same PDP and packs in both modes with host overlays only tightening. AC: overlay loosening rejected with `policy.overlay.rejected` event (DEC-17). S.

## E15 Policy pack authoring and approval workflow (PH-3A, PH-4A) [Must]
- **S15.1 Pack format and build.** As a policy author, I want packs as versioned, effective-dated YAML with citation, URL and matrix_ref, so that every policy is sourced. AC: signed manifest; invalid signature refused at load. M.
- **S15.2 Maker-checker workflow.** As a legal approver, I want DRAFT -> LEGAL_REVIEW -> APPROVED -> RETIRED with checker different from maker, so that no one self-approves. AC: roles `policy_author`, `policy_approver` added to access control [MANAGER confirm] (DEC-11, DEC-12); approval record stored. M.
- **S15.3 Activation control.** As a manager, I want activation to need ops plus manager/legal sign-off, so that unapproved packs never go live. AC: `policy.pack.activated` event; activation of unapproved pack refused. M.
- **S15.4 Periodic review.** As a compliance officer, I want review reminders and expiry fail-safe. AC: `review_due` passed -> treated as pending. S.
- **S15.5 Add-a-jurisdiction rehearsal.** As an architect, I want a documented dry run adding a dummy jurisdiction pack with no core change, so that extensibility is proven (file 14 section 12). AC: zero core diffs; contract tests unchanged. S.

## E16 NYC policy pack (PH-4A) [Must]
Entry: story S13.2 decisions for the rows used. Content is illustrative until approved; no legal text is invented.
- **S16.1 Federal pack (US).** As a policy author, I want draft policies from file 13 federal rows (candidates: file 13 rows F01, F04, F05, F06, F07, F09-F12, F14; selection by legal) mapped to controls. AC: each policy has `matrix_ref`; status PENDING_LEGAL_REVIEW until approved. M.
- **S16.2 NY State pack (US-NY).** As a policy author, I want draft policies from file 13 rows S01, S04-S08, S11-S13, S16 (candidates) mapped to controls. AC: same as story S16.1. M.
- **S16.3 NYC pack (US-NY-NYC).** As a policy author, I want draft policies from file 13 rows C02, C04-C07 (candidates) and placeholders for rows C01, C03 pending legal. AC: same as story S16.1. M.
- **S16.4 Baseline L0 pack.** As a product owner, I want internal safeguards (no credit decisions, LLM guardrails, no sensitive inference, minors two-tier block list) as a non-law baseline. AC: always on; cannot be loosened by any pack. M.
- **S16.5 Exclusion and proxy lists.** As a compliance officer, I want versioned sensitive-attribute and proxy lists as pack data, so that targeting is controlled. AC: lists approved before use (DEC-23); changes versioned. M.

## E17 Policy simulator and tests (PH-8A) [Must]
- **S17.1 Simulator.** As a policy author, I want to run synthetic customers and activities against a pack and see decisions and diffs vs the current pack. AC: report attached to LEGAL_REVIEW submission. M.
- **S17.2 Golden cases.** As a tester, I want cases per jurisdiction incl. NYC resident, unsupported state, unknown jurisdiction, cross-jurisdiction customer. AC: pass in CI; failing case blocks pack build. M.
- **S17.3 Invariants and monotonicity.** As a compliance engineer, I want tests that no pack loosens L0 and no policy permits credit decisions. AC: property tests. M.
- **S17.4 Legal-review-pending demo case.** As a manager, I want the fail-safe visible. AC: pending pack blocks commercial output with flagged outcome and legal queue event. M.

## E18 Minors safeguards (PH-6, PH-8A) [Must]
- **S18.1 Separate protected journey.** As a compliance officer, I want minors routed to a protected journey with no commercial profiling, targeting or marketing. AC: `CTL-MINOR-PROTECT` enforced in M1-M9 per the two-tier list (file 14 §13.3); Tier-1 activities always blocked; no training on minors' data. M.
- **S18.2 Approval-record override (Tier 2 only).** As a legal approver, I want any non-commercial educational use for minors (Tier 2) to need an explicit, signed `MINOR-APPROVAL-*` record referenced in policy. AC: absent record -> DENY; a config flag alone is insufficient; Tier-1 (commercial) activities cannot be relaxed by any record, pack or host overlay. M.
- **S18.3 Unknown age.** AC: unknown age treated as protected for commercial activities; test. M.
- **S18.4 Open legal items.** As a manager, I want file 13 rows S12, S13, F07, F12 and file 13 §6 Q4, Q13, Q14 routed to legal before any non-commercial minor insight is designed (DEC-22). AC: owners assigned. S.

## E19 Integration contracts (PH-3, PH-11) [Should]
- **S19.1 Contract v1 per file 15.** (Supersedes/extends stories S3.2 and S11.1.) As a host developer, I want versioned REST contracts incl. `GET /v1/capabilities` (jurisdictions in scope, active packs) and the policy decision query. AC: OpenAPI published; no credit-decision type exists in the schema. S.
- **S19.2 Envelope and errors.** AC: canonical request/response with jurisdiction context, `decision_id`, error model, idempotency; responses and events carry the purpose-limitation label `not_for_credit_decisioning` (file 15 IC7; DEC-42); contract tests green. S.
- **S19.3 Host simulator and contract testing.** (Same as story S11.2; merge at BMAD re-homing.) AC: mock host drives core; parity with standalone. S.
- **S19.4 Boundary data handling.** AC: tokenised IDs, PII handling per file 15 section 8, CloudEvents outbound. S.
- **S19.5 Retention outcome contract [REC].** As a host developer, I want to report retention-action outcomes, so that retention effectiveness is tracked in plugin mode. AC: `POST /v1/retention-actions/{action_id}/outcomes` with outcome and reason, scope `ri.retention.feedback` (file 15 §2, §5); contract test; same fields as story S8.6. S.
- **S19.6 Consent and preference ingest [REC].** As a host integrator, I want to push consent records and contact preferences (opt-outs, channel preferences, contact limits, quiet hours, vulnerability flag refs, offer suppressions incl. the credit-platform "pause offers" stub, CI-2), so that consent and suppression gates use host data. AC: `POST /v1/consents:batchUpsert` and `POST /v1/preferences:batchUpsert` per `contracts/openapi/retail-intelligence-v1.yaml`, scope `ri.consent.write`; purposes validated against reference data; per-record accept/reject, nothing silently corrected; withdrawal effective for new decisions immediately and audited; suppressions never become features. M (consent is a gate input for every Must module).
- **S19.7 Bulk data ingest [REC].** As a host integrator, I want to load customers, accounts, transactions, goals and the catalogue in multi-part batches (stateful mode), so that large volumes arrive with lineage and a validation report. AC: `POST /v1/ingest/batches`, `PUT .../parts/{n}`, `POST .../{id}:commit`, `GET /v1/ingest/batches/{id}` per the normative contract, scopes `ri.ingest.write` / `ri.ingest.read`; DQ-01..DQ-10 applied; exact duplicates skipped by `dedupe_key`; rejects in the report; idempotent parts; `ingest.completed` event when the outbox exists. S.

## Adjusted acceptance criteria (US/UK references)
- S3.1, S4.1, S4.3, S10.1 changed from US/UK to NYC-market and jurisdiction/policy views.
- S6.3 additionally: aligned to the two-tier block list; Tier-2 relaxation requires a signed approval record (`approval_record_ref`), never a config flag alone.
- S8.2 gates additionally call the PDP; gate trace includes `decision_id` and control IDs.
- S9.3 additionally: assistant never states "permitted by law X"; refuses credit-decision intents.
- S12.2 compliance pack includes applicability decisions and pending items by file 13 row ID.

## Change log (NYC scope update)
- Added epics E13-E19 and epic-to-phase map extension.
- Replaced US/UK wording in S3.1, S4.1, S4.3, S10.1; extended ACs of S6.3, S8.2, S9.3, S12.2.
- E13 gates E16.

## Change log (validation pass 2026-11-09)
- D-01: minors two-tier block list (file 14 §13.3) stated once at the top and applied in S6.3, S16.4, S18.1, S18.2 (approval record relaxes Tier 2 only).
- D-02: S8.2 "affordability-flag input" removed; gates are consent, jurisdiction (PDP), minor protection, suppression incl. credit-platform "pause offers"; AC states we never assess affordability.
- D-08: S7.3 split into S7.3a model registry (M) and S7.3b drift view (S), matching file 09.
- D-09 / G-9: S3.1 adds retention outcomes; new S8.6 retention outcome capture and S19.5 retention outcome contract.
- D-26: story counts corrected and updated: E1-E12 now 57, E13-E19 now 34, total 91 across 19 epics (was "about 50", "about 35", "about 85").
- D-28: S19.1 marked as superseding/extending S3.2 and S11.1; S19.3 marked same as S11.2.
- D-29 / G-1: new S8.5 human review queue. D-30 / G-2: S12.1 and S10.2 ACs add role-to-scope matrix. D-31 / G-3: new S12.7 backup and restore. D-32 / G-4: new S12.6 DSR erasure with hash-chained logs. D-33 / G-5: S6.4 AC adds fairness method. D-34 / G-6: new S5.5 data-quality rules; S6.1 AC references it. D-35 / G-7: S10.1 AC adds accessibility checks. D-39 / G-8: S4.4 AC adds validation report. D-40 / G-10: new S12.8 runbooks and reason-code library. D-11: S19.2 AC adds purpose-limitation label.
- D-41: [LEGAL-CONFIRM] added to legend. D-42: S13.1 question references made explicit. D-06, R-06, R-15: phases/gates renamed PH-x / GATE-x; story vs file 13 row naming convention stated; file 13 rows written "file 13 row ...".
- Sweep 10:22 (S3.1 "market partitioning" -> deployment-scope and jurisdiction-context partitioning), 10:30 (S4.3 `jurisdictions_offered`), 10:52 (S8.1 catalogue version and `jurisdictions_offered`), 10:53 (S8.2) corrected. Future-expansion and historical US/UK mentions kept.

## Change log (PH-3 contract specification 2026-11-09)
- BMAD readiness R-4: new S19.6 consent and preference ingest and S19.7 bulk data ingest (endpoints of file 15 §2 that had no story). Re-homed as BMAD Stories 2.10 and 3.6 in `_bmad-output/planning-artifacts/epics.md`. Story count now 93 (E13-E19: 36).