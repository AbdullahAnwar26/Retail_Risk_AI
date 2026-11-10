# 06 - Module Responsibility Matrix

Labels: [DOC p.N] [REC] [ASSUME] [UNKNOWN] [MANAGER] [LEGAL-CONFIRM]. NYC scope update: every module calls the Jurisdiction Policy Layer (file 14); control IDs referenced below are from file 14 section 5. Nine modules are [MANAGER]-defined. Rule of thumb [MANAGER]: deterministic code for financial calculations, validated ML for prediction, LLM only for orchestration and explanation; LLM never approves credit, rejects services, alters accounts or bypasses rules.

Legend: D = deterministic, ML = machine learning (baseline first), LLM = language model.

## 1. Summary matrix
| # | Module | Responsibility | Inputs | Outputs | D / ML / LLM | Depends on | Owner of data |
|---|---|---|---|---|---|---|---|
| 1 | Customer Intelligence | Build unified customer financial profile and health indicators | Customers, accounts, goals, consent, transaction features (from M2) | Customer financial features, profile summary, data-quality flags | D (aggregates, ratios); ML none in MVP; LLM none | M2, M8 (consent) | Customer features, profile snapshots |
| 2 | Transaction Intelligence | Normalise, dedupe, categorise, detect recurring income/bills, handle transfers/refunds/reversals/pending/FX | Raw transactions, accounts, merchant/category reference | Clean transaction ledger view, categories, recurring items, income/expense/savings aggregates | D rules first; ML baseline classifier only after rules; LLM none | Reference data, M8 | Normalised transactions, categories, aggregates (raw transactions owned by source/host) |
| 3 | Customer Segmentation | Assign multi-membership segments (Gen Z, blue-collar, others); route protected minors (16-17) | Features (M1), age, consent, optional credit segment/tags | Segment memberships with reason and confidence, protected flag | D rules (MVP); ML clustering optional later; LLM none | M1, M8 | Segment memberships |
| 4 | Financial Needs Prediction | Identify/prioritise needs (e.g. emergency savings, budgeting help; v1 taxonomy file 12 BQ-6; no credit-building need in v1, DEC-49) | Features, segments, goals | Need list with score, drivers, evidence | D rules baseline; ML (logistic/gradient boosting) only when synthetic validation shows value; LLM none | M1, M2, M3 | Predictions (needs) |
| 5 | Churn Prediction | Estimate disengagement risk and drivers | Time-windowed features, activity, balance trends | Churn risk score/band, top drivers, model version | D baseline (inactivity rules); ML baseline (logistic regression, then tree models); LLM none | M1, M2, M8 | Predictions (churn), model versions |
| 6 | Product Recommendation | Rank catalogue products for needs; apply eligibility, jurisdiction (PDP + `jurisdictions_offered`), minor-protection and suppression gates | Needs (M4), segments (M3), churn (M5), catalogue, eligibility rules, consent | Ranked recommendations with reason codes, gate results | D rules (eligibility, ranking constraints); ML optional ranker later; LLM none (wording only via M9) | M3-M5, catalogue, M8 | Recommendations, catalogue (reference owner is host/bank) |
| 7 | Retention Intelligence | Suggest retention/engagement actions and priority | Churn (M5), needs (M4), recs (M6), suppression flags, outcome reports (host or staff) | Action suggestions, priority queue, outcome records (outcome, reason, time, source; file 07 `retention_action`) | D rules (playbook mapping); ML uplift only later; LLM none | M4-M6, M8 | Retention actions, outcomes |
| 8 | Risk and Compliance | Owns the Jurisdiction Policy Layer (PDP host, pack verification, decision log); policy gates: consent, minors guard (two-tier list), jurisdiction resolution, fairness/proxy monitoring, explanation validation, audit | All module requests/results, jurisdiction policy packs + market reference data | Allow/deny/flag decisions, fairness reports, audit records | D only (policy as data); ML only for monitoring statistics | Policy config (compliance-approved) | Audit records, consent/preferences cache, fairness reports |
| 9 | Central AI Orchestrator | Route requests to tools in right order, compose structured results, explain in plain language, assistant Q&A | User/API requests, structured module outputs | Orchestrated response, explanations, assistant answers | LLM for intent routing and wording; all facts from tools; D validation of numbers | M1-M8, LLM runtime | Conversation/trace logs (no new facts) |

## 2. Module detail

### M1 Customer Intelligence
- Business purpose: single, explainable view of a customer's money behaviour.
- Responsibilities: assemble features (income level/stability, savings rate, spend mix, balances trend), financial health indicators, data-quality flags ("never fill a gap with a guess", DOC p.9) (rules: file 07 §4.1; below threshold -> `DATA_QUALITY_INSUFFICIENT`, no output; story S5.5).
- Interfaces: `POST /v1/customers/{customer_token}/insights:evaluate`, `GET /v1/customers/{customer_token}/insights` (file 15 §2); internal feature API. Outputs labelled not-for-credit-decisioning (DEC-42) because profile features (income stability, spend vs income) resemble credit affordability inputs.
- Data: customers, accounts, goals, features.
- Policy calls: pre-PEP `PROFILE_BUILD`; post-PEP strips excluded features. Controls: `CTL-CONSENT-PURPOSE`, `CTL-DATA-MINIMISE`, `CTL-SENSITIVE-EXCLUDE`, `CTL-MINOR-PROTECT` (`PROFILE_BUILD` is Tier 2: blocked for minors unless a signed `MINOR-APPROVAL-*` record allows a named non-commercial educational use, file 14 §13.3). Jurisdiction-aware: field allow-list and retention come from the resolved pack, not code.
- Testing: golden synthetic customers with hand-computed features; currency/rounding tests.
- Security: tokenised IDs, minimal fields, access logged.
- Modes: Standalone reads synthetic DB; Plugin receives data via adapter. Same core code.

### M2 Transaction Intelligence
- Purpose: trustworthy transaction understanding.
- Responsibilities: dedupe, pending vs posted, reversals/refunds linkage, internal transfer pairing, FX handling, categorisation (rules + merchant dictionary, ML later), recurring/salary detection, income smoothing method (12-month average pattern is used by credit, DOC p.22; reuse of method is [REC] but separately implemented).
- Interfaces: batch and per-customer analysis endpoints; category taxonomy reference.
- Policy calls: pre-PEP `TXN_ENRICHMENT` (for minors limited to age/routing checks, file 14 §13.3); post-PEP forbidden-inference check on categories. Controls: `CTL-CONSENT-PURPOSE`, `CTL-NO-INFERENCE`. Jurisdiction-aware: category/language maps are reference data; forbidden-inference list versioned in the pack.
- Testing: edge-case fixtures (duplicates, reversals, mixed currency), precision/recall on synthetic labels.
- Security: raw descriptions treated as sensitive; no sensitive-trait inference (e.g. health, religion from merchants) [MANAGER].

### M3 Customer Segmentation
- Purpose: multi-membership segments to personalise insight (not credit decisions).
- Responsibilities: rules for Gen Z (18-29, age-based) and blue-collar (definition [UNKNOWN]); teens 16-17 get `protected=true` and are excluded from commercial profiling; memberships carry explanation. Optional read of credit S0-S5 and tags as attributes [UNKNOWN interface].
- Policy calls: pre-PEP `SEGMENTATION_INSIGHT` or `SEGMENTATION_COMMERCIAL`; post-PEP drops commercial memberships for blocked customers, proxy check. Controls: `CTL-MINOR-PROTECT`, `CTL-PROXY-REVIEW`, `CTL-SENSITIVE-EXCLUDE`. Minors' protected journey stays separate; age unknown is treated as protected for commercial use. Age bands are config, not jurisdiction literals.
- Testing: overlap cases, age boundary 17/18, teen-exclusion tests.
- Security/fairness: segments must not become proxies; fairness monitoring via M8.

### M4 Financial Needs Prediction
- Purpose: surface needs before the customer asks.
- Responsibilities: need taxonomy (e.g. emergency fund, debt-pressure help, savings goal, budgeting support); rule scores with evidence; baseline ML later; goal tracking.
- Outputs: need, score, drivers, evidence references, model/rule version.
- Testing: label-by-construction on synthetic data with caution (leakage); time-split evaluation.
- Policy calls: pre-PEP `NEEDS_PREDICTION`; post-PEP tags needs insight-use vs commercial-use; minors blocked unless approval record (Tier 2, file 14 §13.3). Controls: `CTL-MINOR-PROTECT`, `CTL-MODEL-APPROVED-ONLY`, `CTL-EXPLANATION`.
- Note: credit-building "need" is not in the v1 taxonomy (DEC-49); no credit offer logic [REC]. Needs and churn are served by the rule baseline in v1; ML runs only in SHADOW until DEC-19 is resolved (DEC-47).

### M5 Churn Prediction
- Purpose: estimate likelihood of disengagement.
- Responsibilities: churn definition [UNKNOWN]; feature windows; leakage-safe training; calibrated score and bands; drivers (explainability); monitoring (drift) and model versioning.
- Policy calls: pre-PEP `CHURN_PREDICTION`; score withheld if blocked. Controls: `CTL-MINOR-PROTECT`, `CTL-MODEL-APPROVED-ONLY`, `CTL-EXPLANATION`.
- Testing: time-based split, calibration, subgroup metrics; baseline must be beaten before ML adopted.

### M6 Product Recommendation
- Purpose: suggest relevant, eligible products with reasons.
- Responsibilities: candidate generation from needs; eligibility and `jurisdictions_offered` filter from catalogue (synthetic; real terms by bank); minor-protection gate (Tier 1: always blocked); suppression ("pause offers" from credit, DOC printed 21); consent/marketing preference; ranking; reason codes; no invented terms [MANAGER].
- Outputs: recommendation with product id, reason code(s), gate trace, expiry; credit products appear only as "refer to credit platform for assessment" [REC].
- Policy calls: pre-PEP `PRODUCT_RECOMMENDATION`, `MARKETING_ELIGIBILITY`, `CREDIT_PRODUCT_REFERRAL`; post-PEP gates every candidate (suppress, channel filter, human-review flag). Controls: `CTL-MARKETING-SUPPRESS`, `CTL-CHANNEL-ALLOW`, `CTL-CREDIT-REFERRAL-ONLY` (always on; AI never approves/rejects credit), `CTL-ADVERSE-ACTION-HANDOFF`, `CTL-HUMAN-REVIEW`, `CTL-MINOR-PROTECT`, `CTL-VULNERABILITY-ROUTE`. Jurisdiction-aware: allowed channels and wording come from the resolved pack; catalogue is market reference data. No affordability assessment: affordability is the credit platform's and reaches us, if at all, as a suppression flag (file 15 §11 CI-2).
- Human review [REC] (gap G-1): candidates flagged `requires_human_review` are withheld until a reviewer approves-to-show or rejects with a mandatory reason (standalone dashboard queue; plugin outcome via `POST /v1/recommendations/{rec_id}/outcomes`); human-review outcome recorded per file 08 CMP-14 (story S8.5; DEC-37).
- Testing: gate tests (every blocked case), catalogue-grounding tests, jurisdiction isolation (unsupported or unknown jurisdiction gets restrictive default, no NYC rules borrowed).

### M7 Retention Intelligence
- Purpose: turn churn/needs into helpful actions.
- Responsibilities: playbooks (fixed catalogue of actions, e.g. check-in, relevant savings feature), priority queue for RM/staff, outcome capture (NOT_ACTIONED / CONTACTED / RETAINED / CLOSED / UNKNOWN with reason, time and source HOST or STAFF; values per DEC-43; staff entry in standalone, `POST /v1/retention-actions/{action_id}/outcomes` in plugin; stories S8.6, S19.5); respect contact limits and vulnerability flags (DOC p.41, 54); no automatic customer contact.
- Policy calls: pre-PEP `RETENTION_SUGGESTION`; post-PEP applies contact limits and quiet hours. Controls: `CTL-CONTACT-LIMIT`, `CTL-MARKETING-SUPPRESS`, `CTL-VULNERABILITY-ROUTE`, `CTL-MINOR-PROTECT`, `CTL-HUMAN-REVIEW`, `CTL-EXPLANATION`.
- Testing: action rules traceability; suppression honoured; outcome recording audited and never triggers contact.

### M8 Risk and Compliance
- Purpose: enforce rules before and after intelligence runs.
- Responsibilities: **owns policy enforcement** - hosts the PDP (`core/policy/`), loads only signed packs in the runtime bundle (`ACTIVE`, or `ILLUSTRATIVE` in a standalone demo only; file 14 §8.4, DEC-44, DEC-45), resolves customer jurisdiction context (`US` > `US-NY` > `US-NY-NYC` plus L0 baseline), writes the decision log with `decision_id`; market reference-data selection; consent/purpose checks; minors protection (two-tier block list, file 14 §13.3); policy-as-data (compliance-approved, versioned, maker-checker [DOC p.9 pattern]); fairness and proxy monitoring (method per file 08 CMP-09 and DEC-21: defined metrics, minimum group size, proxies only from a restricted store, never inferred); explanation/number validation (DOC p.40); audit record writer; data minimisation and retention hooks.
- [REC] Validation-gap items owned by M8: DSR erasure keeps the append-only, hash-chained audit and decision logs intact by storing only `subject_ref` tokens and destroying the token mapping (file 07 §8; DEC-35; story S12.6); backups of decision log and signed packs with a restore test that verifies the hash chain (file 08 SEC-20; DEC-36; story S12.7); role-to-scope/screen matrix enforced for audit, simulator, approval and reveal functions (file 08 SEC-19; DEC-38).
- Outputs: gate decisions with reason, audit entries, periodic reports.
- Legal content: to be confirmed by legal/compliance [LEGAL-CONFIRM]; module ships with placeholders/config, not invented rules. Packs stay `PENDING_LEGAL_REVIEW` (restrictive default, `CTL-DEFAULT-DENY`) until the legal approver sets applicability; developers cannot mark a policy NOT_APPLICABLE (file 14 section 9).
- Fail-safe: unknown or out-of-scope jurisdiction, unknown age, unknown consent, no matching policy, PDP error -> DENY for commercial activities (file 14 section 10).
- Controls: all, esp. `CTL-DEFAULT-DENY`, `CTL-AUDIT-LOG`, `CTL-FAIRNESS-MONITOR`, `CTL-RETENTION`, `CTL-DSR-SUPPORT`.

### M9 Central AI Orchestrator
- Purpose: single entry for assistant and composite workflows.
- Responsibilities: tool calling to M1-M8 via typed schemas; structured outputs; explanation text built from module results; numeric consistency check (discard and fall back to template if mismatch, DOC p.40 pattern); refusal behaviour for out-of-scope actions (credit decisions, account changes); trace logging.
- Constraints: no invented products, terms, eligibility or customer data; LLM unavailable -> templates (DOC printed 25).
- Policy calls: pre-PEP `EXPLANATION_GENERATION`, `ASSISTANT_QA` (LLM allowed? languages?); post-PEP output validator, AI-disclosure, refusal of credit-decision/adverse-action intents. Controls: `CTL-LLM-GUARDRAIL`, `CTL-EXPLANATION`, `CTL-AI-DISCLOSURE`, `CTL-NO-INFERENCE`, `CTL-MINOR-PROTECT` (`EXPLANATION_GENERATION`, `ASSISTANT_QA` are Tier 2 for minors). The LLM is governed under the bank's own risk practices; a separate GenAI governance track is a recommended control and bank policy decision (DEC-19), not a regulatory requirement. May query policy decisions (file 15 section 3.4) but never states "permitted by law X".
- Testing: tool-call contract tests, hallucination/number-mismatch tests, prompt-injection tests.

## 3. Mode behaviour (all modules)
| Aspect | Standalone | Plugin |
|---|---|---|
| Core logic | Shared package, identical | Shared package, identical |
| Data source | Synthetic generator + local DB | Host adapter (API/batch/event) |
| Auth | Local demo auth | Host identity via adapter [UNKNOWN] |
| Catalogue/eligibility | Synthetic NYC-market catalogue files (other markets later) | Host service via adapter |
| Policy packs | Same signed packs and PDP | Same packs; host may tighten, never loosen (file 14 section 11) |
| Output | Dashboard + API | Versioned API/events only |
| Audit | Local store | Local store plus export to host sink [UNKNOWN] |
| Consent | Synthetic consent records | Host consent service |

## 4. Dependency graph (order of build)
M2 -> M1 -> M3 -> M4/M5 -> M6 -> M7; M8 gates all; M9 wraps all.

## 5. Jurisdiction Policy Layer: rules for all modules
- A module that skips its pre-processing PEP cannot return output: results require a `decision_id` (file 14 section 6).
- Modules never contain jurisdiction literals; they pass a customer jurisdiction context and activity to the PDP owned by M8.
- Obligations a PEP does not understand lead to DENY.
- Applicability of any law is not assumed; packs cite file 13 row IDs and stay `PENDING_LEGAL_REVIEW` until approved [LEGAL-CONFIRM].
- Minors: every module applies the same two-tier block list (file 14 §13.3). Tier 1 (`SEGMENTATION_COMMERCIAL`, `CHURN_PREDICTION`, `PRODUCT_RECOMMENDATION`, `MARKETING_ELIGIBILITY`, `RETENTION_SUGGESTION`, `CREDIT_PRODUCT_REFERRAL`, `MODEL_TRAINING`) always BLOCK and is locked against packs and host overlays; Tier 2 (`PROFILE_BUILD`, `NEEDS_PREDICTION`, `SEGMENTATION_INSIGHT`, `EXPLANATION_GENERATION`, `ASSISTANT_QA`) BLOCK unless a signed `MINOR-APPROVAL-*` record allows a named non-commercial educational use; `TXN_ENRICHMENT` limited to age/routing checks.
- Outputs are retail insight and marketing suggestions only; responses and events carry `not_for_credit_decisioning` (file 15 §1 IC7; DEC-42) [LEGAL-CONFIRM].

## Change log (NYC scope update)
- Added Jurisdiction Policy Layer ownership to M8 and policy calls, control IDs (file 14 section 5) and jurisdiction-aware behaviour to M1-M9.
- Replaced US/UK catalogue and isolation wording with NYC market plus restrictive default for unsupported jurisdictions.
- Added section 5 (policy layer rules).

## Change log (validation pass 2026-11-09)
- D-01: M1 adds `CTL-MINOR-PROTECT`; M2, M4, M6, M9 and section 5 apply the two-tier minors block list (file 14 §13.3); M8 "teen protection" -> minors protection.
- D-02: M6 states no affordability assessment (credit-platform suppression flag only, CI-2).
- D-09 / G-9: M7 inputs/outputs and responsibilities add retention outcome capture (fields, values per DEC-43, endpoint, stories S8.6, S19.5).
- D-11: M1 and section 5 add the `not_for_credit_decisioning` label (DEC-42).
- D-18: M7 controls add `CTL-HUMAN-REVIEW`, `CTL-EXPLANATION`; M9 adds `CTL-MINOR-PROTECT`.
- D-22: M1 interface now `POST /v1/customers/{customer_token}/insights:evaluate` / `GET .../insights` (file 15 §2).
- D-29 / G-1: M6 human review bullet (story S8.5, CMP-14, DEC-37). D-33 / G-5: M8 fairness method reference (DEC-21). D-34 / G-6: M1 data-quality rules reference (file 07 §4.1). G-2, G-3, G-4: M8 [REC] bullet (role matrix, backup/restore, DSR erasure vs hash-chained logs).
- File 13 re-verification: M9 notes the separate GenAI governance track is a recommended control (DEC-19).
- Sweep 06:15 ("market" gate -> jurisdiction), 06:17 (policy config per market -> jurisdiction policy packs + market reference data), 06:63 (market filter -> `jurisdictions_offered` filter) corrected; 06:65/76/95/112 kept (defined concept / future expansion).

## Change log (readiness fixes 2026-11-09)
- M4 row and note: credit-building need removed from v1 (DEC-49); rule baseline serves, ML SHADOW only (DEC-47). M8: loads runtime-bundle packs only (`ACTIVE`, or `ILLUSTRATIVE` in a standalone demo; DEC-44, DEC-45).
