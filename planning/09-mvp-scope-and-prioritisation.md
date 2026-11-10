# 09 - MVP Scope and Prioritisation

Labels: [DOC p.N] [REC] [ASSUME] [UNKNOWN] [MANAGER] [LEGAL-CONFIRM] legal/compliance must confirm. Phases `PH-x` and gates `GATE-x` per file 11; decisions `DEC-xx` per file 12 §A. Goal [MANAGER]: demonstrate the subsystem standalone to the manager, end to end, with synthetic data, adults 18+, **New York City market only** (US federal + NY State + NYC policy scope). Other states and UK are deferred. Not legal advice [LEGAL-CONFIRM].

> **Two-track scope update (2026-11-10, authoritative; file 21; file 12 DEC-50..DEC-64):** the NYC-only goal is superseded. The MVP now demonstrates **two tracks from one build**: Track A "Conventional" (cells `A-US` = NYC and `A-UK` = London; UK no longer deferred, DEC-13 superseded) and Track B "Islamic" (Islamic bank, UAE onshore [ASSUME], separate deployment). Adults by jurisdiction threshold (DEC-60). Other US states and other countries stay deferred. Everything below stays valid for the `A-US` cell; additions are marked 2026-11-10.

## 1. MoSCoW
| Priority | Item | Rationale |
|---|---|---|
| Must | Synthetic data generator (NYC-resident adults 18+ plus a few cross-jurisdiction, unknown-jurisdiction and minor records for guard tests, edge-case transactions, labelled for testing) | Everything depends; no real data [MANAGER] |
| Must | Data model + versioned core contracts (customer, transaction, insight, recommendation, audit) | Enables Plugin without rework [REC] |
| Must | Transaction Intelligence (dedupe, pending, reversals/refunds, transfers, multi-currency, categorisation, recurring, income/expense/savings) | Foundation for all modules |
| Must | Customer Intelligence profile + features | Core value |
| Must | Segmentation with multi-membership (Gen Z 18-29, blue-collar) + minors protected routing per the two-tier block list (file 14 §13.3: commercial activities and model training always blocked; profile, needs, insight segments, explanations and assistant blocked unless a signed `MINOR-APPROVAL-*` record allows a named non-commercial educational use) | [MANAGER]; safeguard |
| Must | Financial needs (rule-based) with explanations | Core value |
| Must | Churn: rule baseline + one transparent ML baseline with time-split evaluation. The rule baseline serves in v1; the ML model runs only in SHADOW (synthetic data, outputs logged, never shown) until DEC-19 is resolved and MRM approves (DEC-47) | Required capability; ML only after baseline [MANAGER] |
| Must | Recommendation engine with eligibility/jurisdiction/minor-protection/suppression gates (suppression incl. credit-platform "pause offers"), catalogue-grounded, reason codes | Core value; compliance |
| Must | Risk and Compliance gates + audit trail + consent flags (synthetic) | Non-negotiable [MANAGER] |
| Must | Orchestrator with typed tools, structured output, number check, template fallback | Shows controlled AI |
| Must | Standalone API + dashboard (customer view, transactions, segments, needs, churn, recs, audit, policy decision view, jurisdiction selector limited to in-scope and test values) | Manager demo |
| Must | Jurisdiction Policy Layer MVP slice: jurisdiction hierarchy, policy data model, PDP in M8, PEPs in all modules, control catalogue, decision log, fail-safe defaults (file 14) | User; core stays jurisdiction-neutral |
| Must | Illustrative policy packs for US federal, NY State, NYC drawn from file 13 rows, status PENDING_LEGAL_REVIEW until legal approves. The demo runs them in pack state `ILLUSTRATIVE` (standalone, `demo=true`, synthetic data only, every output watermarked "illustrative - not legal advice - not for real customers"; refused at load by plugin/production deployments) (DEC-44, DEC-45) | [LEGAL-CONFIRM]; no invented legal content |
| Must | Regulatory applicability assessment (file 13 rows) by legal/compliance, gating pack approval | [LEGAL-CONFIRM] |
| Must | Policy simulator and golden-case tests; pack authoring and approval workflow (maker-checker) | Needed for legal-review-pending demo and pack approval (file 14 sections 9, 14); matches E15/E17 in file 10 |
| Must | Model registry with `kind` (`RULESET` / `ML_MODEL`) and status (RULESET: DRAFT/ACTIVE/RETIRED; ML_MODEL: DRAFT/SHADOW/APPROVED/RETIRED), version on every prediction (story S7.3a; DEC-47) | Needed by `CTL-MODEL-APPROVED-ONLY` in Must PEPs |
| Must | Role-to-scope/screen permission matrix, deny by default (file 08 SEC-19; story S12.1 AC) [REC] | Validation gap G-2; audit, simulator and approval screens need it |
| Must (2026-11-10) | Deployment profiles and data cells: signed profile loader, `A-US`/`A-UK` cells with separate databases, Track B separate deployment; bootstrap refusals on mismatch (DEC-51..DEC-53) | Two-track scope; same core |
| Must (2026-11-10) | Track A UK cell: synthetic `A-UK` data and `CAT-A-UK-SYNTH` catalogue, illustrative `PACK-GB`, `PACK-GB-ENG`, `PACK-GB-ENG-LDN` (content after file 19, written 2026-11-10; PENDING_LEGAL_REVIEW = BLOCK; scope waits for the UK entity type, DEC-65); Track A human review on offer suppression, retention allocation and vulnerability routing (DEC-66) | DEC-50, DEC-56, DEC-65, DEC-66 [LEGAL-CONFIRM] |
| Must (2026-11-10) | Track B: institution-type overlay `PACK-INST-ISLAMIC` with `CTL-CATALOGUE-ELIGIBILITY`, `CTL-TERMINOLOGY`, `CTL-PROHIBITED-CATEGORY`, `CTL-SHARIA-APPROVAL`; Sharia approval state and `sharia_approver` role; synthetic Islamic catalogue and `B-AE` data; illustrative `PACK-AE` packs (content after file 20, written 2026-11-10; overlay aligned with SDC-01..SDC-18, file 21 §4.4); AE contact caps with in-app counted as telemarketing (DEC-67) | DEC-54..DEC-56, DEC-67 [LEGAL-CONFIRM] [SHARIA-CONFIRM] |
| Must (2026-11-10) | Arabic + English explanations and dashboard RTL for Track B; per-cell currency (DEC-59) | Track B demo |
| Must (2026-11-10) | Per-track model registry and evaluation (rule baselines serve; ML SHADOW per track) (DEC-57) | No model crossing tracks |
| Should | Integration contracts v1 incl. policy decision query and capabilities manifest (file 15) | Plugin path |
| Should | Retention Intelligence (playbook suggestions + priority list + outcome capture, stories S8.4, S8.6, S19.5) | Completes story; rules only; outcomes for reporting, no uplift modelling |
| Should | Human review queue for items flagged `requires_human_review` (story S8.5) [REC] | Until built, flagged items are withheld (fail-safe) |
| Should | Reference adapter + mock host integration test against contract v1 (file 15) | Shows plugin path without host |
| Should | Fairness/proxy monitoring report by segment | DOC Spec s4 pattern; needs care |
| Should | AI assistant chat in dashboard (read-only questions) | Demo value |
| Should | Drift/monitoring view (story S7.3b) | Governance pattern DOC printed 23 |
| Should | Data-quality rules with `DATA_QUALITY_INSUFFICIENT` outcome (story S5.5; file 07 §4.1) [REC] | Avoid outputs on thin data; thresholds [ASSUME] (DEC-39) |
| Should | Backup/restore design and restore test (file 08 SEC-20; story S12.7), DSR erasure method for hash-chained logs (story S12.6) [REC] | Validation gaps G-3, G-4; full DSR support stays deferred |
| Could | Portfolio analytics (aggregate dashboards across customers; 2026-11-10: aggregate-only, no tokens or PII, small cells suppressed, cross-cell inside Track A only, never across tracks; DEC-62) | Brief lists it; cheap once features exist |
| Could | Event-style integration example (webhook/queue) | Only if host needs it [UNKNOWN] |
| Could | Goal tracking features | Nice-to-have |
| Won't (MVP) | Minors commercial journey/profiling or model training (Tier 1, locked); minors non-commercial insight (Tier 2) beyond the protected placeholder | Tier 1 locked by the L0 baseline; Tier 2 needs a signed approval record (DEC-22) [MANAGER] [LEGAL-CONFIRM] |
| Won't (MVP) | Real data, real bank connections, production deployment, SSO with host | [MANAGER] |
| Won't (MVP) | Credit decisions, limits, affordability, credit offers logic | Out of scope; affordability is assessed only by the credit platform and reaches us, if at all, as a suppression flag |
| Won't (MVP) | Deep learning, sequence models, uplift models, LLM fine-tuning | No data/justification [MANAGER] |
| Won't (MVP) | Customer messaging/sending, account actions | Bank systems [DOC p.53] |
| Won't (MVP) | Other segments (retiree, new-to-country, S0-S5 recompute) | Fairness/legal review first |
| Won't (MVP) | 50-state rulebooks; policy packs for any state other than NY; ~~UK or~~ other international packs (2026-11-10: GB/England/London and UAE packs are in, DEC-50; Scotland, Wales, Northern Ireland, EU, Qatar, other GCC and UAE free zones stay out) | Two-track scope; architecture supports later via packs |
| Won't (MVP) | Any hosted LLM; any data or model crossing tracks or personal data crossing Track A cells | DEC-52, DEC-53, DEC-58 |

## 2. Explicitly deferred
1. Minors protected journey beyond routing/exclusion (Tier-2 non-commercial insight only; design later with legal/compliance, DEC-22; FR-39 definition needed).
2. Retention uplift modelling and A/B outcome learning.
3. Consuming real credit feeds (segments S0-S5, tags, early-warning) pending interface definition [UNKNOWN].
4. Microservices split, event streaming platform, message broker (justify only on host evidence).
5. Host SSO/identity integration and production hardening (SOC 2/ISO evidence).
6. Model-risk documentation packs (SR 26-2 style; SR 26-2 is supervisory guidance) - pattern only; applicability to confirm (DEC-19).
7. Real NYC-market product terms and eligibility (legal/compliance + product teams).
8. Other US states (e.g. US-CA, US-NJ) as new packs, other international markets beyond London and the UAE (2026-11-10, DEC-50), additional languages beyond en/ar: future expansion, no core change (file 14 section 12) [MANAGER confirm].
9. Final approval of NYC packs by legal/compliance; until then packs are illustrative and enforced restrictively.
10. Full DSR support, supranational jurisdictions (2026-11-10: multi-deployment residency is now designed via cells and tracks, DEC-52, DEC-53).

## 3. Prioritisation principles
- [REC] Rules and baselines before ML; ML must beat baseline on time-split synthetic validation.
- [REC] Every recommendation carries gate trace and reason code.
- [REC] Smallest deployable shape: one modular codebase with a clean core package; split services only with host evidence.
- [REC] Build vertical slice early (one customer end-to-end), then broaden.

## 4. MVP success criteria (demo)
| Criterion | Measure |
|---|---|
| End-to-end flow | Pick NYC synthetic customers -> profile -> segments -> needs -> churn -> recommendation -> retention -> audit |
| Controls visible | Minor record blocked per the two-tier list (no commercial output, no profile or needs without an approval record); consent withdrawn removes recs; customer with unknown or unsupported jurisdiction gets restrictive default and no NYC rules borrowed |
| Explainability | Each output shows drivers and rule/model version |
| Assistant safety | Assistant refuses "approve this loan" and cannot state products not in catalogue |
| Plugin proof | Same core invoked through contract v1 by a mock host test, results identical to standalone |
| Quality | Baseline model metrics and subgroup report reproducible from seed |

## 5. Demo narrative (10-12 minutes)
1. Framing: "This is the retail intelligence subsystem; credit decisions stay with the credit platform" (scope slide).
2. Pick **Alex, NYC, 24, shift worker** (synthetic; see section 7 for jurisdiction display): dashboard shows normalised transactions (duplicate and reversal handled), salary detection, spending mix.
3. Segments: Gen Z + blue-collar multi-membership with reasons.
4. Needs: emergency buffer shortfall; evidence from last 6 months.
5. Churn: medium-high risk, top drivers (salary diverted, falling balance); rule-baseline version shown (the ML model is in SHADOW and not shown, DEC-47).
6. Recommendation: savings feature and budgeting tool from the NYC-market catalogue, each with reason and gate trace; credit shown only as "refer for assessment" at most. Permitted only under the `ILLUSTRATIVE` demo bundle and shown with the watermark "illustrative - not legal advice - not for real customers" (DEC-44).
7. Retention: suggested human check-in; priority list for the team; record an outcome (e.g. CONTACTED) to show outcome tracking.
8. Assistant: ask "why this product?" -> grounded answer; ask "approve a loan" -> refusal.
9. Out-of-scope or missing-jurisdiction customer: restrictive default (see section 7; replaces the US/UK switch).
10. Minor record (age 16): "protected - no commercial profiling" shown.
11. Audit trail and consent withdrawal demo.
12. Plugin proof: run mock host call against contract v1; show identical output; close with open questions needing approval.

## 6. Dependencies to unlock before build [MANAGER]
Churn definition, blue-collar definition and Gen Z band (DEC-04); ownership of next-best offers (DEC-01); confirmation that synthetic catalogues are acceptable for demo (DEC-15); whether nightly credit signals are in or out (DEC-03); retention outcome values (DEC-43). See file 12 §G.3 for the full business-question list.

## 7. NYC scope: policy layer MVP slice and demo additions
**MVP slice:** hierarchy `*` > `US` > `US-NY` > `US-NY-NYC`; policy record schema, resolution and conflict rules; PDP/PEP wiring in M1-M9; control catalogue; decision log; three illustrative packs (federal, NY State, NYC) built only after the applicability assessment for the relevant file 13 rows; simulator with golden cases (file 14 section 14).
**Deferred:** other states, UK/EU, supranational membership, host overlay tightening beyond a basic test, automated review-due workflow.
**Fail-safe:** any policy whose applicability is not yet decided by legal is `PENDING_LEGAL_REVIEW` and enforced as BLOCK (file 14 section 10).
**Demo posture (DEC-44, resolves the conflict between §5 step 6 and the fail-safe):** the demo deployment (`mode=STANDALONE`, `demo=true`, synthetic only) loads the illustrative packs in pack state `ILLUSTRATIVE`. Policies with an explicit `illustrative_assumption` are evaluated on that assumption, so Alex can receive watermarked suggestions; their real status stays PENDING_LEGAL_REVIEW; policies without an assumption still BLOCK. Plugin, pilot and production deployments refuse `ILLUSTRATIVE` packs at load. If MGR/COMP do not ratify DEC-44 at GATE-1, config switches it off and step 6 shows blocked outcomes with reasons.

Demo narrative changes (replace steps 2, 9 and add steps):
- Step 2: pick **Alex, NYC, 24, shift worker** (synthetic). Show resolved jurisdiction context `US-NY-NYC` and the active pack versions (capabilities manifest).
- Step 6: each recommendation shows policy decision id, controls applied (e.g. `CTL-CREDIT-REFERRAL-ONLY`, `CTL-MARKETING-SUPPRESS`) and gate trace.
- Step 9 (replaces US customer switch): a customer with an out-of-scope or missing jurisdiction (synthetic, e.g. `US-CA`) gets the restrictive default; show that NYC rules are not borrowed.
- New step: **legal-review-pending fail-safe**. In the simulator, remove the illustrative assumption from the relevant policy (it reverts to PENDING_LEGAL_REVIEW); the commercial recommendation is blocked, outcome flagged, a legal queue event appears. Then show the approval record gating activation (maker-checker): the pack stays `IN_LEGAL_REVIEW` because no approver is named (DEC-11). Reason codes cite policy IDs and statuses only, never "permitted by law X".
- Step 10: minor (age 16) protected journey; age unknown also treated as protected for commercial use.
- Close: state that applicability of every law is for the bank's legal/compliance teams [LEGAL-CONFIRM].

## 7A. Two-track demo plan (2026-11-10; DEC-64; detail in file 21 §11)

Two standalone demos from **one build**, both `mode=STANDALONE`, `demo=true`, synthetic only, `ILLUSTRATIVE` bundles (DEC-44), watermark on every output, no hosted LLM. Sections 5 and 7 above become the `A-US` part of the Track A demo.

| Demo | Scenes (summary) |
|---|---|
| **Track A "Conventional"** (~12 min) | Framing (two tracks, one core; credit stays with the credit platform) -> `A-US`: Alex, NYC, 24 (sections 5 and 7) -> `A-UK`: London customer, en-GB/GBP, GB packs PENDING_LEGAL_REVIEW with illustrative assumptions (file 19, written 2026-11-10) -> cell isolation (London token unknown in `A-US`; NYC-resident, London-booked customer resolves to UNKNOWN, commercial denied) -> aggregate-only portfolio across cells -> shared controls (minor protected, `US-CA` out of scope, consent withdrawal, pending-review fail-safe) |
| **Track B "Islamic"** (~10 min, separate deployment and URL) | Capabilities (`institution_type: ISLAMIC`, overlay loaded, ar + en, AED, no LLM) -> synthetic UAE customer, Arabic RTL and back -> suggestions only from the synthetic Islamic catalogue (Mudarabah and Takaful to human review; Murabaha/Ijarah only as `CREDIT_REFERRAL`) -> terminology check and template fallback -> prohibited category shown neutrally, never used for offers, no religion inference -> Sharia approval block (`SHARIA_REVIEW_PENDING`; watermark "illustrative - not a Sharia ruling") -> isolation (Track A tokens/models unknown; `GB` is `UNSUPPORTED_JURISDICTION`) -> minor protected (AE threshold [ASSUME 21]), "approve my financing" refused with handoff |

Success criteria in section 4 apply per track; BQ-11 KPI 6 (plugin parity) is shown once, on Track A. [REC]

## Change log (NYC scope update)
- Goal and success criteria changed from US+UK to NYC-only; MoSCoW rows updated; policy layer, packs, applicability assessment, simulator and contracts added.
- Deferred list extended (other states, UK, packs approval); added Won't row for 50-state rulebooks.
- Added section 7 (MVP slice, deferred items, updated demo narrative with jurisdiction decisions and legal-review-pending fail-safe).

## Change log (validation pass 2026-11-09)
- D-01: segmentation Must row, Won't row, deferred item 1, success criterion and demo step 10 use the two-tier minors block list (file 14 §13.3).
- D-02: Won't row states affordability is assessed only by the credit platform (suppression flag at most); gate row names suppression incl. "pause offers".
- D-07: policy simulator, golden cases and maker-checker moved Should -> Must (matches file 10 E15/E17 and the legal-review-pending demo).
- D-08: split into Must model registry with approval status (story S7.3a) and Should drift view (story S7.3b).
- D-09 / G-9: retention row adds outcome capture (stories S8.6, S19.5); demo step 7 records an outcome.
- G-1 (D-29): Should human review queue (story S8.5). G-2 (D-30): Must role-to-scope matrix. G-3, G-4 (D-31, D-32): Should backup/restore and DSR erasure method. G-6 (D-34): Should data-quality rules.
- D-41: [LEGAL-CONFIRM] added to the legend; §6 dependencies cite DEC IDs (file 12).
- Sweep 09:15 ("market" gate -> jurisdiction) corrected; 09:46 "/ SS1/23" dropped (UK future) and SR 26-2 labelled supervisory guidance; other "UK deferred" and "NYC-market" mentions kept (future expansion / defined concept).

## Change log (readiness fixes 2026-11-09)
- §1: churn row (rule baseline serves; ML SHADOW only, DEC-47); illustrative packs row (`ILLUSTRATIVE` demo pack state, DEC-44/DEC-45); model registry row (`RULESET`/`ML_MODEL` statuses, DEC-47).
- §5 steps 5-6: rule-baseline version shown; recommendations permitted only under the watermarked `ILLUSTRATIVE` bundle (DEC-44).
- §7: demo-posture paragraph resolving the §5 vs fail-safe conflict (BMAD readiness R-1); pending-review step now removes the illustrative assumption and shows the pack stuck in `IN_LEGAL_REVIEW` (DEC-11). Target market itself is open (DEC-50); NYC content unchanged.

## Change log (two-track scope 2026-11-10)
- Header two-track note; MoSCoW Must rows for deployment profiles/cells, Track A UK cell, Track B Sharia overlay and Islamic catalogue, Arabic/RTL, per-track models; portfolio Could row aggregate-only (DEC-62); Won't rows updated (UK no longer deferred; no cross-track/cross-cell data, no hosted LLM); deferred items 8 and 10 updated; new §7A two-track demo plan (DEC-64). Source: file 21, file 12 DEC-50..DEC-64.
- Files 19/20 recorded as written 2026-11-10; Must rows cite DEC-65, DEC-66, DEC-67 and the SDC mapping (file 21 §4.4).
