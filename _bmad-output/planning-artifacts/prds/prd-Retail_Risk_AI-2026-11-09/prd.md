---
title: Retail Banking AI Intelligence Subsystem (two tracks: Conventional NYC + London; Islamic bank)
status: final
created: 2026-11-09
updated: 2026-11-10
workflow: bmad-prd (v6.12.4), create intent, fast path, facilitated from planning pack
inputs: [briefs/brief-Retail_Risk_AI-2026-11-09/brief.md, planning/01-18, planning/21]
---

# PRD: Retail Banking AI Intelligence Subsystem
*Working title: confirm at GATE-1.*

> **Scope (DEC-50, DECIDED by the developer 2026-11-10; planning/21, planning/12 §H.7).** Two separate tracks on one shared core and one contract. **Track A "Conventional"**: US / US-NY / US-NY-NYC and UK (`GB`) / England (`GB-ENG`) / London (`GB-ENG-LDN`), one deployment profile with two data cells (`A-US`, `A-UK`; DEC-52). **Track B "Islamic"**: an Islamic bank, assumed UAE onshore [ASSUMPTION], a separate bank-hosted deployment that never shares data or models with Track A (DEC-53), with a tighten-only Sharia overlay (DEC-54, DEC-55). DEC-13 (UK deferred) is superseded. UK legal content (planning/19) and UAE + Sharia content (planning/20) were written on 2026-11-10; all GB and AE packs stay PENDING_LEGAL_REVIEW = BLOCK. The NYC content below stays valid for the `A-US` cell. FR-69..FR-82 add the two-track requirements.

## 0. Document Purpose

This PRD is for the project manager (approver), the developer (subsystem owner), legal/compliance, model-risk, and the BMAD downstream workflows: `bmad-ux`, `bmad-architecture` and `bmad-create-epics-and-stories`. It re-homes the approved-with-conditions planning pack (`planning/00-18`) into BMAD format. It does not replace the pack: each section cites its source as "Source: planning/NN §x", and the pack stays the authority for detail.

How to read it:
- Features are grouped, with FRs numbered globally (FR-1..FR-82; FR-69..FR-82 added for the two-track scope, §4.13). NFRs are in §10.
- Decisions use the file 12 IDs (DEC-xx, BQ-x) and the status from file 12 §H: **DECIDED**, **DECIDED-DEFAULT** (manager ratifies at the gate shown) or **BLOCKED-EXTERNAL** (build proceeds on the stated safe assumption). Since 2026-11-10 (§H.7) the GATE-1, GATE-2 and GATE-3 DECIDED-DEFAULT batches are **DECIDED (developer-delegated)**, manager countersignature recommended; labels below that still say DECIDED-DEFAULT for those items are read through §H.7.
- `[LEGAL-CONFIRM]` marks points the bank's legal/compliance teams must confirm. Nothing here is legal advice, and no law is assumed to apply.
- `[ASSUMPTION]` marks values set without confirmation. They are indexed in §9.
- FR IDs here are local to this subsystem. The platform's own prd.md IDs (platform FR-39, FR-50/51, FR-79..86) are cited with the "platform" prefix. That document was never provided (planning/12 §B MS1; gap accepted, 12 §H.3).

## 1. Vision

The subsystem turns a retail customer's transactions into an explainable picture of their finances, their likely needs and their disengagement risk. It then suggests catalogue products and retention actions that staff can explain and act on. It is advisory only. It never approves or declines credit, never changes accounts and never contacts customers. *Source: planning/02 §1, §3; planning/05 §1 PR-3.*

There is one shared intelligence core. It runs as a standalone demo with its own API, dashboard and synthetic data, and as a plugin behind versioned REST contracts. The host never imports our code, so the host's unknown stack does not matter. *Source: planning/04 §1; planning/03 §6.*

Jurisdiction rules are data. Versioned, signed, legal-approved policy packs for US federal, New York State and New York City, and for UK, England and London (Track A), and for the UAE (Track B), are evaluated by one PDP. An Islamic institution adds a tighten-only Sharia overlay. Uncertainty always resolves to the restrictive outcome. *Source: planning/14 §1; planning/02 §8; planning/21 §1-§4.*

## 2. Target User

### 2.1 Jobs To Be Done
- **Relationship staff:** "Show me why this customer may leave and what I can safely offer, with reasons I can repeat." *Source: planning/10 S8.3, S10.1.*
- **Retention reviewer:** "Let me clear or reject flagged suggestions before anyone sees them, within an SLA." *Source: planning/10 S8.5; DEC-37.*
- **Compliance reviewer / auditor:** "Let me replay any recommendation: its inputs, versions, gates and the policy decision." *Source: planning/10 S10.2, S14.6.*
- **Policy author / approver / release (ops):** "Let me draft, simulate, approve and activate jurisdiction packs, with maker-checker and no self-approval." *Source: planning/14 §9.*
- **Manager:** "Show me the whole flow on synthetic NYC and London customers (Track A) and on synthetic Islamic-bank customers (Track B), including what is blocked and why." *Source: planning/09 §5, §7; planning/21 §11.*
- **Sharia approver (Track B):** "Let me approve or reject the overlay pack, catalogue items and templates separately from legal approval, and see that nothing customer-facing runs without my approval." *Source: planning/21 §4.3; DEC-55.*
- **Host integrator:** "Let me call a versioned contract and get identical results to the standalone." *Source: planning/15 §1-2.*

### 2.2 Non-Users (v1)
- Retail customers: there is no customer-facing UI in the MVP. Bank systems own all customer communication. *Source: planning/02 §3.*
- Credit-platform decision makers, who must not consume our outputs for credit decisions (DEC-42).
- Minors (16-17), except as protected synthetic test records (DEC-09).

### 2.3 Key User Journeys
Staff personas in the pack are roles, not named people. The named customer persona "Alex" comes from planning/09 §5. Downstream UX keeps these UJ IDs (see `ux-designs/.../EXPERIENCE.md`).

- **UJ-1. The manager walks Alex end to end.** Alex is synthetic: NYC, 24, a shift worker who declared an occupation with consent [ASSUMPTION: needed for blue-collar membership under BQ-2]. The manager opens the standalone dashboard (local demo auth) and picks Alex.
  - The dashboard shows the resolved jurisdiction `US-NY-NYC`, the runtime-bundle pack versions (pack state `ILLUSTRATIVE` in the demo), and normalised transactions with the duplicate and reversal handled.
  - It then shows segments (Gen Z + blue-collar, each with reasons), the need `NEED_EMERGENCY_BUFFER` with 6-month evidence, and the churn band with drivers and model version.
  - Climax: two recommendations from the SYNTHETIC catalogue, each with reason codes, gate trace, `decision_id` and controls applied. Credit appears at most as a referral. They are permitted only because the demo deployment runs the illustrative packs in pack state `ILLUSTRATIVE` (DEC-44), so each carries the watermark "illustrative - not legal advice - not for real customers".
  - Resolution: a human check-in retention action is recorded as CONTACTED.
  - *Source: planning/09 §5 steps 2-7, §7.*
- **UJ-2. The fail-safe is visible.**
  - The manager switches the selector to an out-of-scope (`US-CA`) customer and to one with a missing jurisdiction. Both get the restrictive default; NYC rules are not borrowed.
  - In the simulator the illustrative assumption is removed from the relevant policy, so it is plain PENDING_LEGAL_REVIEW again. The commercial recommendation is blocked and flagged, and a legal-queue event appears.
  - *Source: planning/09 §7; DEC-15, DEC-16 (DECIDED).*
- **UJ-3. A minor is protected.** Opening a synthetic 16-year-old shows "protected - no commercial profiling". There are no Tier-1 outputs, and no Tier-2 outputs unless a signed `MINOR-APPROVAL-*` record exists. An age-unknown customer is treated as protected for commercial activities. *Source: planning/09 §5 step 10; planning/14 §13.3.*
- **UJ-4. The assistant stays grounded.** Staff ask "why this product?" and get a template answer built from tool results. They ask "approve a loan" and get a refusal plus a handoff message. *Source: planning/09 §5 step 8; planning/10 S9.3.*
- **UJ-5. The retention reviewer clears a flagged item.** A suggestion flagged `requires_human_review` is QUEUED with an SLA indicator (2 business days [ASSUMPTION], DEC-37). The reviewer moves it to IN_REVIEW, then CLEARED_TO_SHOW or REJECTED with a mandatory reason. Expired items are never shown. *Source: planning/07 §2.1 `review_item`; planning/10 S8.5.*
- **UJ-6. A policy goes through maker-checker.** The policy author drafts a pack change from a file 13 row, attaches the simulator diff and submits it (pack state `IN_LEGAL_REVIEW`). The approver, who cannot be the author, approves. The release role activates it after recorded sign-off. **In the MVP no approver is named (DEC-11), so this journey ends with the pack in `IN_LEGAL_REVIEW`** (pack lifecycle, DEC-45). *Source: planning/14 §9; planning/12 §H.1.*
- **UJ-7. The host gets the same answer.** A mock host calls `POST /v1/customers/:token/evaluate` with jurisdiction context and consent purpose. It receives results identical to the standalone path, each with a `decision_id` and `not_for_credit_decisioning: true`. *Source: planning/15 §2, §13; planning/04 §7.*
- **UJ-8. Consent is withdrawn.** When marketing consent is withdrawn, recommendations disappear, affected results are recomputed and the withdrawal is audited. *Source: planning/09 §5 step 11; planning/08 PRV-03.*
- **UJ-9. Track A demo spans two cells.** UJ-1 runs in the `A-US` cell (Alex). The manager then opens a synthetic London customer in the `A-UK` cell (en-GB, GBP, GB packs PENDING_LEGAL_REVIEW with illustrative assumptions, watermarked). The London token is not found in `A-US`; a NYC-resident, London-booked customer resolves to UNKNOWN and is denied commercial activity. The portfolio view shows aggregate-only counts across both cells (no tokens, small cells suppressed). *Source: planning/21 §11; DEC-52, DEC-62, DEC-64.*
- **UJ-10. Track B demo (separate deployment).** Capabilities show `institution_type: ISLAMIC` and `PACK-INST-ISLAMIC`. A synthetic UAE customer goes through the same core flow with the dashboard in Arabic (right-to-left) and English. Suggestions come only from the synthetic Islamic catalogue; Mudarabah and Takaful items go to human review; Murabaha, Tawarruq, Ijarah and the Islamic card appear only as `CREDIT_REFERRAL`. A template using "loan"/"interest" is rejected and the fallback is shown. A catalogue item moved to `IN_SHARIA_REVIEW` turns the suggestion into BLOCK (`SHARIA_REVIEW_PENDING`); outputs also carry "illustrative - not a Sharia ruling". A `GB` jurisdiction presented to Track B is UNSUPPORTED. *Source: planning/21 §11; DEC-53..DEC-56, DEC-59, DEC-64.*

## 3. Glossary
- **Shared core** — The single intelligence package used identically by standalone and plugin modes. Contains no I/O, no mode branching and no jurisdiction literals outside `core/policy`. *planning/04 §4.*
- **Standalone mode / Plugin mode** — Two adapter configurations of the same image, selected at start-up. *planning/04 §1.*
- **Jurisdiction** — A node in a hierarchy held as data: `*` > `US` > `US-NY` > `US-NY-NYC`; `*` > `GB` > `GB-ENG` > `GB-ENG-LDN`; `*` > `AE` > `AE-<emirate>` [ASSUMPTION]. *planning/14 §2; planning/21 §2.*
- **Track** — A deployment profile family: Track A Conventional (NYC + London), Track B Islamic (UAE onshore [ASSUMPTION]). Config, never a code branch. *planning/21; DEC-51.*
- **Deployment profile** — The signed, read-only configuration a deployment loads at bootstrap: track, `institution_type` (`CONVENTIONAL`/`ISLAMIC`), mode, demo, cells, packs, institution overlay, age-of-majority map, LLM hosting, registry scope, IdP realm. *planning/21 §3; DEC-51.*
- **Cell** — The unit of personal-data isolation: one database, subject-key store, logs and region. Track A has `A-US` and `A-UK`; Track B has `B-AE`. *planning/21 §2; DEC-52.*
- **Institution overlay (Sharia layer)** — `PACK-INST-ISLAMIC`, a restrictive-only policy layer between the jurisdiction packs and any host overlay, loaded only for `institution_type = ISLAMIC`. *planning/21 §4; DEC-54.*
- **Sharia approval state** — `NOT_REVIEWED`, `IN_SHARIA_REVIEW`, `SHARIA_APPROVED`, `SHARIA_REJECTED` on the overlay pack, Track B catalogue items, templates and reason-code texts; recorded by `sharia_approver`. *DEC-55.*
- **Jurisdiction context** — The coarse codes a customer is connected to: residence, booking entity, servicing location, channel, age status, consents, vulnerability flag and `as_of`. *planning/14 §3.4.*
- **Deployment scope** — The set of jurisdictions a cell serves (`in_scope` in the deployment profile): `A-US` = `[US, US-NY, US-NY-NYC]`, `A-UK` = `[GB, GB-ENG, GB-ENG-LDN]`, `B-AE` = `[AE, AE-<emirate>]`. *planning/07 §5; planning/21 §3.*
- **Policy pack** — A versioned, effective-dated, signed set of policy records for one jurisdiction level: `PACK-GLOBAL-BASELINE` (L0), `PACK-US`, `PACK-US-NY`, `PACK-US-NYC`; `PACK-GB`, `PACK-GB-ENG`, `PACK-GB-ENG-LDN`; `PACK-AE`, `PACK-AE-<emirate>`; plus the institution overlay `PACK-INST-ISLAMIC`. *planning/14 §8; planning/21 §3.*
- **Control** — An enforceable mechanism with a `CTL-*` ID from the control catalogue, independent of any law. *planning/14 §5.*
- **PDP / PEP** — Policy Decision Point (in `core/policy`, hosted by M8) and Policy Enforcement Point (one per module, called before and after processing). *planning/14 §6.*
- **Decision** — A PDP outcome (PERMIT, PERMIT_WITH_OBLIGATIONS, DENY) with a `decision_id`, reason codes, obligations and the packs applied. `INDETERMINATE` is enforced as DENY. *planning/15 §3.4.*
- **PENDING_LEGAL_REVIEW** — Status of any policy whose applicability legal has not decided. Enforced as BLOCK. *planning/14 §10.*
- **Pack state** — Pack-level lifecycle: `DRAFT`, `VALIDATED` (the "loaded but unapproved" state: simulator only), `IN_LEGAL_REVIEW`, `APPROVED`, `ACTIVE`, `ILLUSTRATIVE`, `RETIRED`. Only `ACTIVE` packs, or `ILLUSTRATIVE` packs in a standalone demo, are in the runtime bundle. *planning/14 §8.4; DEC-45.*
- **ILLUSTRATIVE (pack state)** — Demo-only activation of an all-illustrative pack. It loads only when `mode=STANDALONE`, `demo=true` and the synthetic-only guard is on, and plugin or production deployments refuse it at load. Policies with an explicit `illustrative_assumption` are evaluated on it, and every output is watermarked "illustrative - not legal advice - not for real customers". It is not approval. *planning/12 §H.6 DEC-44.*
- **SHADOW (model status)** — An ML model that runs on synthetic data with its outputs logged, never shown and never used. It is the only ML status reachable while DEC-19 is blocked. *DEC-47.*
- **Activity** — An item in the versioned taxonomy a policy scopes to (e.g. `PRODUCT_RECOMMENDATION`). There is no `CREDIT_DECISION` activity. *planning/14 §3.3.*
- **Two-tier minors block list** — Tier 1 is always BLOCK and locked. Tier 2 is BLOCK unless a signed `MINOR-APPROVAL-*` record allows it. `TXN_ENRICHMENT` is limited to age/routing checks. *planning/14 §13.3.*
- **Catalogue** — The versioned product list with `jurisdictions_offered`, one per cell: `CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH`, `CAT-B-AE-ISLAMIC-SYNTH`. SYNTHETIC in the MVP (BQ-9, DEC-56). *planning/07 §2.1; planning/21 §5.*
- **Recommendation** — A catalogue-grounded suggestion of kind PRODUCT_SUGGESTION, SERVICE_SUGGESTION, EDUCATIONAL_CONTENT or CREDIT_REFERRAL. *planning/15 §3.3.*
- **Credit referral** — "Refer to the credit platform for assessment", with no eligibility, limit, rate or likelihood. *planning/14 §13.1.*
- **Retention action / outcome** — A suggestion from the fixed playbook (BQ-6), and its reported outcome (BQ-7). *planning/12 §H.2.*
- **Review item** — A human-review record with states QUEUED, IN_REVIEW, CLEARED_TO_SHOW, REJECTED, ESCALATED, EXPIRED, CANCELLED. *planning/07 §2.1.*
- **Reason code** — A code from the fixed, versioned reason-code library. Owner COMP (DEC-40). *planning/08 CMP-11.*
- **Churn** — Working definition BQ-1: closure, salary diversion or dormancy within a 90-day window. *planning/12 §H.2.*
- **Gen Z segment** — Age 18-29 at the snapshot date (lower bound = the jurisdiction's age of majority, DEC-60), bounds in config. Separate from the credit GENZ tag (18-25). Display/insight only in the MVP (BQ-3).
- **Blue-collar segment** — From consented, self-declared occupation mapped to SOC 2018 groups only (BQ-2). Display/needs content only. Carries a proxy-risk flag.
- **VARIABLE_INCOME** — Behavioural segment: income coefficient of variation > 0.3, or irregular pay intervals over 6 months [ASSUMPTION] (BQ-2).
- **Synthetic** — Generated data with `is_synthetic=true`. The only data allowed until 12 §H.4 condition 1 is lifted.

## 4. Features

### 4.1 Synthetic dataset (per cell; NYC detail below, London and UAE in FR-74)
**Description:** A seeded, reproducible generator of NYC adult customers (and, per FR-74, London and UAE customers with one seed per cell), accounts and transactions. It also produces small labelled test sets: out-of-scope `US-CA`, missing or `UNKNOWN` jurisdiction, `US-NY` outside NYC, NYC resident booked elsewhere, minors 16-17 and age unknown. It includes edge cases, a SYNTHETIC catalogue and consent records. Realises UJ-1..UJ-3. *Source: planning/07 §7; planning/09 §1; planning/10 E4; DEC-16 (DECIDED).*

#### FR-1: Seeded customer and account generation
The tester can generate NYC-market adult customers and the jurisdiction/age test records from a seed. Realises UJ-1, UJ-2, UJ-3.
**Consequences (testable):**
- The same seed gives byte-identical output (same dataset hash).
- Every row has `is_synthetic=true`. The standalone mode refuses to ingest rows without it.
- The test jurisdictions include `US-NY-NYC`, `US-CA`, missing and `UNKNOWN` (DEC-16).
- Personas include a declared-occupation shift worker (Alex) and a `VARIABLE_INCOME` worker [ASSUMPTION: persona set per planning/07 §7 plus BQ-2].

#### FR-2: Transaction generation with edge cases
The tester can generate transactions with configurable rates of duplicates, pending-then-booked, refunds, reversals, internal transfers, multi-currency and irregular income, with ground-truth labels. *Source: planning/10 S4.2.*
**Consequences:** the rate of each edge case is within the configured tolerance, and every injected edge case has a ground-truth label.

#### FR-3: Synthetic catalogue, consent and preferences
The tester can load a SYNTHETIC NYC-market catalogue (each product carries `jurisdictions_offered` and a "DEMO - not real terms" label) plus synthetic consent and contact-preference records. *Source: planning/10 S4.3; BQ-9 (DECIDED).*
**Consequences:** no real product names or terms. Consent purpose codes are validated against config reference data (DEC-18 safe assumption).

#### FR-4: Behavioural scenarios and validation report
The data scientist can generate time-structured churn and need scenarios and a validation report. *Source: planning/10 S4.4; planning/07 §7.*
**Consequences:** the report contains distribution sanity per persona, edge-case coverage, a label-leakage audit, persona/jurisdiction balance and a reproducibility hash. It is reviewed at GATE-4.

### 4.2 Transaction intelligence (M2)
**Description:** Normalises raw transactions into a clean ledger view used by every other module. *Source: planning/06 M2; planning/10 E5; planning/07 §4.*

#### FR-5: Normalisation and de-duplication
The analyst sees duplicates and pending items handled idempotently. *Source: S5.1.*
**Consequences:** re-processing the same batch changes nothing. Exact duplicates on `dedupe_key` are skipped.

#### FR-6: Transfers, refunds, reversals and FX
Internal transfers are paired, and refunds and reversals are linked. Currencies are never mixed in sums. *Source: S5.2.*
**Consequences:** the fixture tests pass. A sum across currencies without conversion is impossible by type.

#### FR-7: Categorisation with confidence and rule trace
Transactions get a category, confidence, method (RULE/ML/HOST) and rule ID. *Source: S5.3; planning/08 PRV-06.*
**Consequences:** the taxonomy has no forbidden-inference labels (`EXCL-FORBIDDEN-INFERENCE`). Precision/recall are reported on synthetic labels.

#### FR-8: Income, expense, savings and recurring detection
The system detects salary, bills, recurrence and savings rate with a documented calculation. *Source: S5.4.*
**Consequences:** the golden test cases pass. All money is held in minor units with currency.

#### FR-9: Data-quality rules
Each module refuses to produce output on thin or broken data. *Source: S5.5; planning/07 §4.1 DQ-01..DQ-10; DEC-39 (DECIDED).*
**Consequences:** a breach of a blocking rule returns `DATA_QUALITY_INSUFFICIENT` and no output. Thresholds are config (e.g. churn needs at least 12 months of history, needs at least 6). Every result records `rule_version`.

### 4.3 Customer profile and segmentation (M1, M3)
**Description:** Builds the financial profile and the multi-membership segments, and routes protected customers. *Source: planning/06 M1, M3; planning/10 E6, E18.*

#### FR-10: Customer profile and features
Relationship staff see a profile with health indicators (income stability, spend mix, savings rate, balance trend), backed by a feature dictionary. Realises UJ-1. *Source: S6.1.*
**Consequences:** the profile is reproducible from the seed. Excluded attributes are removed before storage. The feature-list hash is recorded.

#### FR-11: Multi-membership segmentation
A customer can belong to several segments, each with reasons and validity dates. Realises UJ-1. *Source: S6.2; BQ-2, BQ-3 (DECIDED-DEFAULT, GATE-1).*
**Consequences:**
- Gen Z = 18-29 at the snapshot date, with bounds in config.
- Blue-collar applies only with a consented self-declared occupation mapped to SOC 2018 groups 37, 45, 47, 49, 51 or 53 [ASSUMPTION mapping].
- `VARIABLE_INCOME` follows the BQ-2 rule.
- Income band S0-S5, ZIP, merchant patterns, name, language, age (beyond Gen Z) and sex are never used for membership.
- Gen Z and blue-collar are never ranking or churn features until DEC-21 and DEC-23 are resolved.

**Notes:** file 10 S6.2 predates BQ-2 and omits `VARIABLE_INCOME`. This PRD follows 12 §H.2.

#### FR-12: Minors and unknown-age protected routing
Compliance can rely on customers aged 16-17, or of unknown age, being routed to the protected path. Realises UJ-3. *Source: S6.3, S18.1-S18.3; planning/14 §13.3; DEC-09 (DECIDED-DEFAULT); DEC-22 (BLOCKED-EXTERNAL).*
**Consequences:**
- Guard tests prove no Tier-1 output is ever produced: commercial segments, churn, recommendations, marketing eligibility, retention, credit referral or model training.
- No Tier-2 output (profile, needs, insight segments, explanations, assistant) is produced without a signed `MINOR-APPROVAL-*` record. A config flag alone is not enough.
- Unknown age is protected for commercial activities. Under-13 records are rejected or quarantined at ingest.
- The protected threshold is the jurisdiction's age of majority from the deployment profile (`US` 18, `GB` 18, `AE` 21 [ASSUMPTION, restrictive]; all [LEGAL-CONFIRM]; DEC-60). Tier 1 is never relaxed in any track.
- Every block is audited.

#### FR-13: Segment fairness view
Compliance can see output distributions by segment and synthetic group. *Source: S6.4; planning/08 CMP-09; DEC-21 (BLOCKED-EXTERNAL, safe assumption).*
**Consequences:**
- Metrics are selection rate and score distribution by group, computed only on synthetic group labels set by the generator and held in a restricted store that models cannot read.
- Minimum group size is config, value 100 per cell [ASSUMPTION; DEC-48 aligns DEC-21 with planning/08 CMP-09]. Smaller cells are shown as "insufficient", never back-filled and never counted as passing.
- Group attributes are never inferred.

### 4.4 Needs and churn predictions (M4, M5)
**Description:** Rule-based needs and churn risk, with ML only where it beats the baseline, all under model governance. *Source: planning/06 M4, M5; planning/10 E7.*

#### FR-14: Rule-based financial needs
Staff see needs from the BQ-6 taxonomy, each with evidence from the last 6 months and a rule version. Realises UJ-1. *Source: S7.1; BQ-6 (DECIDED-DEFAULT, GATE-1).*
**Consequences:**
- The needs are `NEED_EMERGENCY_BUFFER`, `NEED_BUDGETING`, `NEED_SAVINGS_GOAL`, `NEED_INCOME_SMOOTHING`, `NEED_BILL_MANAGEMENT` and `NEED_FINANCIAL_STRESS_SUPPORT`.
- The last one only signposts free guidance plus human review, never a credit product.
- No debt-to-income or affordability calculation exists.

**Notes:** "credit-building need" (planning/02 §2, planning/06 M4) is removed from v1 (DEC-49, DECIDED-DEFAULT, GATE-1). Financial education stays available through `RA_PRODUCT_EDUCATION` and `EDUCATIONAL_CONTENT`. Credit products appear only as `CREDIT_REFERRAL`.

#### FR-15: Churn baseline and transparent ML
The data scientist gets a rules baseline and a transparent ML model (logistic regression first), evaluated on a time split using the BQ-1 label. *Source: S7.2; BQ-1 (DECIDED-DEFAULT, GATE-1).*
**Consequences:**
- Features use only data up to the snapshot date. Labels use only the 90-day window after it.
- Leakage checks and calibration are run.
- ML becomes eligible for MRM review only if PR-AUC beats the baseline by at least 0.02 [ASSUMPTION].
- **In v1 the rule baseline serves churn (and needs).** The ML model runs only in SHADOW: synthetic data, outputs logged, never returned or shown, until DEC-19 is resolved and MRM approves (DEC-47).
- Bands come from calibrated probability, with cut-offs in config.
- Metrics are reported by subgroup.

#### FR-16: Model registry and approval status
Every prediction records the registry entry that produced it. Entries have `kind` `RULESET` (status DRAFT, ACTIVE, RETIRED; labelled "non-model" per planning/08 NYC-07) or `ML_MODEL` (status DRAFT, SHADOW, APPROVED, RETIRED). Only `RULESET` ACTIVE or `ML_MODEL` APPROVED serve (`CTL-MODEL-APPROVED-ONLY`). *Source: S7.3a; planning/08 CMP-06; DEC-19 (BLOCKED-EXTERNAL); DEC-47 (DECIDED-DEFAULT, MRM at GATE-7).*
**Consequences:**
- The PEP blocks every entry that is not cleared to serve.
- `ML_MODEL` → APPROVED needs an MRM approval reference and is refused while DEC-19 is unresolved (`MODEL_APPROVAL_UNAVAILABLE`).
- SHADOW outputs go to a separate shadow log. They never reach the API, customer views, events, M6, M7 or the assistant; only aggregate metrics are shown.
- If MRM classifies rulesets as models, config `ruleset_requires_mrm_approval` applies the same approval to them.
- Registry entries declare their feature list, which CI checks against the exclusion lists.

#### FR-17: Drift indicator
Each model has a simple drift indicator per batch. *Source: S7.3b; CMP-07.* **Priority: Should.**

#### FR-18: Prediction explanations
Each prediction returns its top drivers as reason codes, and these are stable between runs. *Source: S7.4.*

### 4.5 Product recommendation (M6)
**Description:** Catalogue-grounded, gated, ranked suggestions, each with reasons and a gate trace. *Source: planning/06 M6; planning/10 E8; planning/14 §6.*

#### FR-19: Catalogue-grounded candidates
Every recommendation refers to a product in the active catalogue version whose `jurisdictions_offered` includes the customer's resolved jurisdiction. Realises UJ-1. *Source: S8.1.*

#### FR-20: Eligibility and gating
Each candidate passes the gates: consent purpose, jurisdiction (PDP), minor protection, suppression (opt-out, credit-platform "pause offers" stub, vulnerability) and channel. *Source: S8.2; planning/08 CMP-05, CMP-15.*
**Stub data source (R-7):** the "pause offers" stub is a `suppressions[]` entry `{type: PAUSE_OFFERS, source: HOST, origin: CREDIT_PLATFORM}` pushed through `POST /v1/preferences:batchUpsert` (planning/15 §11 CI-2; contract `contracts/openapi/retail-intelligence-v1.yaml`). In the standalone demo the synthetic generator (FR-3) writes these rows; in plugin tests the host simulator pushes them (FR-57). The interface to the real credit platform stays [OPEN] (CI-2). The stub only suppresses through `CTL-MARKETING-SUPPRESS` and is never a feature.
**Consequences:**
- Each gate has a failing-case test.
- The gate trace stores `decision_id` and control IDs.
- Suppressed rows are kept with a reason.
- Eligibility rules are configured as data. There is no affordability input or field.

#### FR-21: Credit only as referral
Credit products appear only as `CREDIT_REFERRAL`, never for under-18s, and never with eligibility, limit, rate or likelihood. *Source: planning/14 §13.1; CMP-03, CMP-05; NYC-04.*
**Consequences:** the forbidden-term linter fails the build on credit-decision terms in customer-outcome schemas.

#### FR-22: Ranking and reason codes
Ranked suggestions carry reason codes from the versioned library (DEC-40). Support-first actions rank before commercial offers when financial-stress signals exist. *Source: S8.3; CMP-16.*

### 4.6 Retention intelligence and human review (M7)
**Description:** Retention suggestions from a fixed playbook, a human review queue and outcome capture. *Source: planning/06 M7; planning/10 S8.4-S8.6, S19.5.*

#### FR-23: Retention suggestions and priority list
The retention officer gets suggestions from the fixed playbook (`RA_HUMAN_CHECKIN`, `RA_SAVINGS_FEATURE`, `RA_BUDGET_TOOL`, `RA_PRODUCT_EDUCATION`, `RA_SERVICE_RECOVERY`, `RA_NO_ACTION`) and a priority list. Realises UJ-1. *Source: S8.4; BQ-6.*
**Consequences:** no automatic contact. No price, fee or rate changes. `RA_HUMAN_CHECKIN` always needs human review. Contact limits and quiet hours are respected (CMP-16).
**Contact-limit and quiet-hour values (R-7):** the effective limit is the most restrictive of (a) `CTL-CONTACT-LIMIT` params in the runtime-bundle packs (`max_per_window`, `window`, `quiet_hours`; minimum of limits, union of quiet hours, planning/14 §4.2) and (b) the host's `contact_limits` and `quiet_hours` from preference ingest. No value is hard-coded in the core. Legal values come only from approved packs [LEGAL-CONFIRM]. Synthetic demo defaults, held in THRESHOLDS config and used only when neither source supplies a value: at most 1 suggested outreach per customer per 7 days, quiet hours 21:00-08:00 in the deployment reference time zone [ASSUMPTION; MGR to confirm at GATE-1; not a legal value]. The response shows the effective values in `contact_constraints`.

#### FR-24: Human review queue
Items flagged `requires_human_review` are held in a queue and shown only once CLEARED_TO_SHOW. Realises UJ-5. *Source: S8.5; DEC-37 (DECIDED-DEFAULT, GATE-8).*
**Consequences:**
- Role is `retention_reviewer`. SLA is 2 business days [ASSUMPTION], held in config.
- A reason is mandatory on REJECTED and ESCALATED. The reviewer cannot be the requester.
- Items pass to EXPIRED at the SLA and are never auto-released.
- Every transition is audited.
- Plugin mode returns the flag and accepts the outcome through the outcome endpoints.

#### FR-25: Retention outcome capture
Staff (standalone) or the host (plugin) can record an outcome per action: NOT_ACTIONED, CONTACTED, RETAINED, CLOSED or UNKNOWN, with a fixed reason, time and source (STAFF or HOST). *Source: S8.6, S19.5; BQ-7 (DECIDED-DEFAULT).*
**Consequences:**
- Outcomes are append-only (`retention_outcome`).
- RETAINED means no churn event within 90 days. UNKNOWN means nothing reported within 120 days.
- Outcomes are for reporting only and never trigger contact.

### 4.7 Jurisdiction policy layer (M8)
**Description:** Jurisdiction rules as data, evaluated by one PDP with fail-safe defaults. Identical in both modes. *Source: planning/14 §1-§11; planning/10 E14.*

#### FR-26: Jurisdiction hierarchy and context resolution
The system holds jurisdictions as data (`*`, `US`, `US-NY`, `US-NY-NYC`; `GB`, `GB-ENG`, `GB-ENG-LDN`; `AE`, `AE-<emirate>`) and resolves each customer's jurisdiction context within the serving cell (FR-70). *Source: S14.1; DEC-25 (BLOCKED-EXTERNAL).*
**Consequences:**
- Resolution uses residence and booking location. If either is missing or they conflict, the result is UNKNOWN (DEC-25 safe assumption).
- A CI lint forbids jurisdiction literals in `core/` outside `core/policy`.

#### FR-27: Policy data model and control catalogue
Policies reference controls from a versioned catalogue whose parameters are validated by schema. *Source: S14.2; planning/14 §3, §5.*

#### FR-28: PDP resolution and conflict rules
The PDP resolves applicable policies deterministically. "Most restrictive wins" is the conflict rule (DEC-26, DECIDED), and `INDETERMINATE` becomes DENY with a conflict record. *Source: S14.3.*

#### FR-29: Enforcement points in every module
Every module calls the PDP before and after processing. No output exists without a `decision_id`, and an unknown obligation means DENY. *Source: S14.4; planning/06 §5.*

#### FR-30: Fail-safe behaviours
Each row of planning/14 §10 has its restrictive behaviour and a test. *Source: S14.5; DEC-14, DEC-24 (DECIDED).*
**Consequences:**
- PENDING_LEGAL_REVIEW is BLOCK, including in the demo. The single exception (DEC-44, ratify GATE-1): inside an `ILLUSTRATIVE` bundle, a policy with an explicit `illustrative_assumption` is evaluated on that assumption, and every output is watermarked.
- An `ILLUSTRATIVE` pack in a plugin, pilot or production deployment, or with `demo=false`, is refused at load and the instance is not ready.
- No policy match means DENY; the L0 permit whitelist is empty in v1 (DEC-45).
- A pack past `review_due` is BLOCK, with no grace period. The last-good bundle cannot be used past its own `review_due`.
- UNKNOWN or UNSUPPORTED jurisdiction denies all commercial activities.
- Unknown consent means no processing for that purpose.
- An invalid pack signature means the system is not ready.
- A PDP exception means DENY.

#### FR-31: Decision log and replay
The auditor can replay any decision with the recorded pack versions and `as_of`, and get an identical outcome. *Source: S14.6; CMP-12.*
**Consequences:** the log holds policy IDs and statuses only. It never says "permitted by law X".

#### FR-32: Host overlay tightens only
A host overlay may add or tighten controls. Loosening parts are rejected with a `policy.overlay.rejected` event. *Source: S14.7; DEC-17 (DECIDED).* Enforced as a core invariant test. **Priority: Should.**

#### FR-33: Locked L0 baseline
`PACK-GLOBAL-BASELINE` holds the safeguards no pack or overlay can loosen: no credit decisions, LLM guardrails, no sensitive inference, and the minors Tier-1 block. *Source: S16.4; planning/14 §13.*
**Consequences (DEC-45):** L0 is restrictive-only; CI rejects any L0 policy that grants an activity. It is approved as `INTERNAL_POLICY` by the developer with a second-person review, signed with the demo-class key in standalone and the deployment key in pilot or plugin. COMP reviews its content at GATE-3A.

### 4.8 Policy pack authoring, approval and simulation
**Description:** Maker-checker authoring of signed packs, illustrative NYC packs and a simulator. *Source: planning/10 E13 (S13.2, S13.4), E15, E16, E17; planning/14 §8, §9, §14.*

#### FR-34: Pack format and signed build
Policy authors write packs as versioned, effective-dated YAML. Each policy cites `matrix_ref`, the source citation and URL. *Source: S15.1.*
**Consequences:** the manifest is signed, and an invalid signature is refused at load.

#### FR-35: Maker-checker approval workflow
Policies move DRAFT, then LEGAL_REVIEW, then APPROVED, then RETIRED, and the checker must differ from the maker. Packs follow the pack lifecycle `DRAFT` → `VALIDATED` → `IN_LEGAL_REVIEW` → `APPROVED` → `ACTIVE` → `RETIRED`, with the demo branch `VALIDATED` → `ILLUSTRATIVE`. `VALIDATED` is the "loaded but unapproved" state: it is used by the simulator only, never at runtime. *Source: S15.2; planning/14 §8.4; DEC-12 (DECIDED-DEFAULT, GATE-3); DEC-45 (DECIDED).*
**Consequences:**
- Roles `policy_author` and `policy_approver` belong to compliance.
- Only `policy_approver` sets `applicability_status`. Developers cannot set NOT_APPLICABLE.

#### FR-36: Activation control
Activation needs `policy_release` (ops) plus recorded manager/legal sign-off. *Source: S15.3; DEC-11 (BLOCKED-EXTERNAL).*
**Consequences:**
- Activating an unapproved pack to `ACTIVE` is refused. While no approver is named, no pack can be approved or made `ACTIVE` (hard stop at GATE-4A and GATE-8A).
- `policy_release` may put a `VALIDATED`, all-illustrative pack into `ILLUSTRATIVE` only in a standalone demo deployment, and the activator must differ from the maker (DEC-44). This is not approval.
- All of this is reached through `POST /v1/policy/packs/:id/:version/activations` (planning/15 §2.1, DEC-46).

#### FR-37: Periodic review and expiry
Each policy has a `review_due` date. Once it passes, the policy is treated as PENDING_LEGAL_REVIEW (DEC-14). *Source: S13.4, S15.4.* **Priority: Should.**

#### FR-38: Illustrative federal, NY State and NYC packs
Draft packs map candidate file 13 rows to controls, and every policy stays PENDING_LEGAL_REVIEW until approved. *Source: S16.1-S16.3; planning/13 §3.*
**Consequences:**
- Every policy has a `matrix_ref`. No legal text is invented.
- Packs are built only for rows decided at GATE-2A. Undecided rows stay as placeholders.
- For the demo, a policy may carry an `illustrative_assumption`, set by the maker and checked by a second user. It never changes `applicability_status` (DEC-44).
- DEC-50 is resolved (two tracks). NYC pack content proceeds from file 13; UK and AE pack content follow FR-77 and FR-78, gated on planning/19 and planning/20 (written 2026-11-10) and a named approver.

#### FR-39: Exclusion and proxy lists as pack data
`EXCL-SENSITIVE`, `EXCL-FORBIDDEN-INFERENCE`, `EXCL-PROXY` and `EXCL-PROTECTION-ONLY` are versioned pack data. *Source: S16.5; planning/14 §13.2; DEC-23 (BLOCKED-EXTERNAL).*
**Consequences:** the starter list covers every protected attribute named in file 13 rows plus known proxies (ZIP/fine location, name-derived traits, language preference, religion- or health-linked merchants). All are excluded and marked PENDING_LEGAL_REVIEW.

#### FR-40: Policy simulator
Policy authors can run golden cases and a synthetic population against a candidate pack and see a diff against the current runtime bundle. Realises UJ-2, UJ-6. *Source: S17.1.*
**Consequences:** the report is a required attachment for LEGAL_REVIEW. The simulator runs through `POST /v1/policy/simulations` and `GET /v1/policy/simulations/:id`, on synthetic data only (planning/15 §2.1, DEC-46).

#### FR-41: Golden cases and invariant tests
There are golden cases per jurisdiction (NYC resident, unsupported state, unknown jurisdiction, cross-jurisdiction) and property tests: monotonicity, determinism, fail-safe, locked baseline, no credit permission. *Source: S17.2, S17.3; planning/14 §14.*
**Consequences:** a failing case blocks the pack build.

#### FR-42: Add-a-jurisdiction rehearsal
A dummy jurisdiction pack is added with zero core diffs and unchanged contract tests. *Source: S15.5; planning/14 §12.* **Priority: Should.**

#### FR-43: Applicability decision records
Each file 13 row can be marked APPLICABLE, NOT_APPLICABLE or PENDING_LEGAL_REVIEW, as signed data, by the legal approver only. *Source: S13.2; DEC-10 (BLOCKED-EXTERNAL).*
**Consequences:** while DEC-10 is unresolved, every row stays PENDING_LEGAL_REVIEW.

### 4.9 AI orchestration (M9)
**Description:** Tool-grounded explanations and a read-only staff assistant. Template-only in the MVP. *Source: planning/06 M9; planning/10 E9; DEC-06 (DECIDED).*

#### FR-44: Typed, read-only tool layer
The orchestrator calls only typed tools from a fixed allow-list that wrap module outputs. No tool writes, sends, changes accounts or decides credit. *Source: S9.1; planning/08 SEC-13.*

#### FR-45: Grounded template explanations
Staff get plain-language explanations built from module outputs and reason codes, using templates in the MVP. Realises UJ-4. *Source: S9.2; planning/08 SEC-14.*
**Consequences:** every number, product code and fact matches the tool results. Any future LLM text that fails validation falls back to the template.

#### FR-46: Refusals and safety
The assistant refuses credit approval or decline, account changes, unknown products and legal-conclusion statements ("permitted by law X"). Realises UJ-4. *Source: S9.3; CMP-03.*
**Consequences:** credit intents return `422 OUT_OF_SCOPE_REQUEST` with the handoff template. The red-team and prompt-injection sets pass.

#### FR-47: LLM port, disabled
An `LLMPort` exists with the template adapter active. Hosted or local LLM adapters stay disabled until a separate MGR/SEC decision before PH-9. *Source: S9.4; DEC-06; DEC-58.* **Priority: Should.**
**Consequences:** the deployment profile restricts `llm.hosting`: Track B allows only `NONE` or `BANK_ON_PREM` (never an external API); Track A allows only in-region hosting per cell. The bootstrap refuses any other value (DEC-58).

### 4.10 Standalone dashboard and demo
**Description:** The staff-facing web dashboard. It is just another API client. *Source: planning/10 E10; planning/09 §5, §7; UX contract in `ux-designs/ux-Retail_Risk_AI-2026-11-09/`.*

#### FR-48: Customer view
Staff see transactions, profile, segments, needs, churn, recommendations and retention for one customer. Each output shows its `decision_id`, jurisdiction context and versions. Realises UJ-1. *Source: S10.1.*

#### FR-49: Audit and controls view
The compliance reviewer can open the audit trail, gate results and consent state, and replay a recommendation. *Source: S10.2.*
**Consequences:** screen access follows the role-to-scope matrix (FR-61).

#### FR-50: Assistant panel
A read-only chat that uses only the M9 tools. Realises UJ-4. *Source: S10.3.* **Priority: Should.**

#### FR-51: Portfolio view
Aggregate segment and churn distributions only, through `GET /v1/portfolio/summary` (`ri.portfolio.read`; FR-80): no tokens or PII, cells below the configured minimum suppressed, minors excluded from commercial aggregates; aggregation across the Track A cells allowed, never across tracks. Aggregate disclosure control per planning/21 §10A (AG-1..AG-10): one fixed dimension per call, cell-root jurisdiction filter only, published snapshot dates only, minimum 20 as a floor config can only raise, rounded counts, complementary suppression, suppression inside each cell before combining, per-user quota and query audit. *Source: S10.4; DEC-62 (tightened, 12 §H.7.3).* **Priority: Could.**

#### FR-52: Scripted demo pack
Seeded personas and a script that matches planning/09 §5 and §7 and runs from a clean start. *Source: S10.5.*

#### FR-53: Policy views
A pack viewer (with the PENDING_LEGAL_REVIEW state, `pack_state` and, in Track B, the Sharia approval state shown openly, DEC-15, DEC-55), a decision-log view, a test-jurisdiction selector limited to the serving cell's in-scope codes plus out-of-scope, cross-cell or cross-track, and unknown cases (DEC-16 as extended in 12 §H.7), and a simulator page. Realises UJ-2. *Source: planning/09 §1, §7; planning/04 §5.1.*
**Consequences:** while the bundle is `ILLUSTRATIVE`, every screen shows a persistent watermark banner reading "illustrative - not legal advice - not for real customers" (DEC-44).

### 4.11 Plugin integration (contract v1)
**Description:** Versioned REST contracts, identity and consent ports, and parity with the standalone. *Source: planning/15; planning/10 E11, E19.*

#### FR-54: Contract v1 publication and capabilities
Host developers get OpenAPI 3.1 and JSON Schema contracts, sample payloads, a changelog and `GET /v1/capabilities` (activities, jurisdictions in scope, runtime-bundle packs with `pack_state`, the `demo` flag, serving registry entries, and the read-only `deployment_profile`: profile ID and version, track, `institution_type`, `cell_id`, currencies, locales, catalogue, institution overlay; DEC-61). *Source: S11.1, S19.1; planning/15 §2.*
**Consequences:** no credit-decision type exists in the schema. Contract v1 includes the governance, simulator, decision-log and model-registry endpoints of planning/15 §2.1 (DEC-46).

#### FR-55: Canonical envelope and error model
Every customer-level request carries jurisdiction context and consent purpose. Every response carries `decision_id`, versions, `permitted_use`, `not_for_credit_decisioning: true`, `illustrative` and `watermark` (DEC-44). *Source: S19.2; planning/15 §3, §4; DEC-42 (DECIDED).*
**Consequences:** the error model and idempotency keys exist, and the contract tests are green. **Priority: Must** (R-7: the label is a DECIDED guardrail, DEC-42, and is validated by the primary metric SM-5; the standalone dashboard uses the same envelope).

#### FR-56: Identity and consent through ports
Host identity (OIDC, claims `ri_deployment` and `ri_jurisdictions`) and host consent are honoured via ports. *Source: S11.3; SEC-03; DEC-30 (BLOCKED-EXTERNAL, safe assumption: generic OIDC adapter plus mock IdP, claim mapping and mTLS as config).*
**Consequences:** a claim mismatch returns 403 `JURISDICTION_NOT_PERMITTED`. Denied access returns no data.

#### FR-57: Host simulator and parity
A mock host drives the core through contract v1 and gets results identical to standalone. Realises UJ-7. *Source: S11.2 = S19.3 (merged); planning/04 §7; planning/15 §13.* **Priority: Should** (R-7 default, consistent with file 09 and the secondary metric SM-6; GATE-1 may promote it to Must for GATE-10 if the manager ratifies plugin parity as a pass/fail KPI, BQ-11 #6).

#### FR-58: Outcome endpoints
`POST /v1/recommendations/:rec_id/outcomes` and `POST /v1/retention-actions/:action_id/outcomes` accept feedback, human-review results and retention outcomes. *Source: S19.5; planning/15 §2.*

#### FR-59: Outbound events via outbox
Customer-outcome and policy events (CloudEvents 1.0) are written to a transactional outbox behind a transport-neutral port. There is no broker in the MVP. *Source: S19.4; DEC-33 (BLOCKED-EXTERNAL safe assumption).* **Priority: Could.**

#### FR-60: Boundary data handling
Only tokenised IDs and the minimum fields per activity cross the boundary. ZIP code and fine location never cross in v1; only the resolved jurisdiction code does. *Source: S19.4; planning/15 §8; DEC-34 (DECIDED).*

### 4.12 Security, privacy and operations
**Description:** Baseline controls needed for the demo and for a later pilot. *Source: planning/08 §2; planning/10 E12.*

#### FR-61: Role-to-scope and screen matrix
Deny-by-default roles x scopes x screens, with separation of duties, kept with the OpenAPI spec and covered by one CI negative test per cell. *Source: S12.1; planning/15 §5.1; SEC-19; DEC-38 (DECIDED-DEFAULT, draft GATE-3, final GATE-12).*

#### FR-62: Hash-chained audit
Every read of customer-level data, every evaluation, config change, pack activation, policy decision, model status change and LLM interaction is written to an append-only, hash-chained audit record. *Source: SEC-07; planning/07 §2.1.*

#### FR-63: Consent purpose enforcement and withdrawal
Consent is checked per source and purpose. Withdrawal stops use, triggers recompute and is audited. Realises UJ-8. *Source: PRV-03, PRV-04; DEC-18 (BLOCKED-EXTERNAL).*
**Consequences:** MARKETING, affiliate and channel purposes are denied unless an explicit synthetic consent record exists.

#### FR-64: Retention lifecycle hooks
Retention windows per entity come from `CTL-RETENTION`, with no values hard-coded. Purge runs in report-only mode until legal values exist. *Source: S12.5; PRV-07; DEC-20 (BLOCKED-EXTERNAL).* **Priority: Should.**

#### FR-65: DSR erasure with hash-chained logs
Deletion and objection requests delete or anonymise operational rows and destroy the per-subject key (crypto-shredding). The hash chain still verifies afterwards. *Source: S12.6; planning/07 §8.2; DEC-35 (DECIDED; LEG/PRIV confirm before real data, GATE-12).* **Priority: Should.**

#### FR-66: Backup and restore
Encrypted backups of the database, decision log and signed packs. A restore test re-verifies hash chains and pack signatures. *Source: S12.7; SEC-20; DEC-36 (DECIDED-DEFAULT: demo none; pilot RPO 24 h, RTO 8 h [ASSUMPTION]).* **Priority: Should.**

#### FR-67: Reason-code library and runbooks
A versioned reason-code library (DEV drafts, COMP approves) and operator runbooks (pack activation and rollback, signing-key rotation, fail-closed recovery). *Source: S12.8; DEC-40 (DECIDED-DEFAULT).* **Priority: Should.**

#### FR-68: Technical evaluation report
A report generated from the seed with the BQ-11 KPIs, risks and synthetic-data limits. *Source: S12.4.*

### 4.13 Two-track deployment (Conventional NYC + London; Islamic bank)
**Description:** One core and one contract serve two tracks configured by signed deployment profiles. Track A has two data cells; Track B is a separate, bank-hosted deployment with a tighten-only Sharia overlay. *Source: planning/21; planning/12 §H.7 DEC-50..DEC-64.*

#### FR-69: Deployment profile and bootstrap refusals
Operators configure each deployment with one signed, versioned deployment profile (planning/21 §3). *Source: DEC-51, DEC-61.*
**Consequences:**
- The profile is read-only at runtime, its hash is written to every decision record, and no request field, header, environment variable, feature flag or config file can set or override track, institution type, cells, overlay, LLM hosting, `demo` or `synthetic_only`.
- Profile integrity (planning/21 §3.1): detached signature (Ed25519; ECDSA P-256 if the bank requires FIPS modules [ASSUMPTION]) checked against track-bound keys held in the track's KMS/HSM; maker-checker signing; the bootstrap also refuses a missing, unreadable, altered, incompatible or rolled-back profile and a key bound to another track.
- The bootstrap refuses to start (not ready, fail-closed) on: an invalid profile signature; `institution_type: ISLAMIC` without an institution overlay pack; any pack, catalogue, model or template whose `track` tag differs from the profile; a cell region not allowed by `CTL-DATA-RESIDENCY`; a disallowed `llm.hosting` value; `demo: true` with `mode: PLUGIN`.
- A CI lint forbids track and market literals (`ISLAMIC`, `CONVENTIONAL`, `GB`, `AE`, `US`, track IDs) in `core/` outside the policy and profile loader.

#### FR-70: Track A data cells and cell routing
Track A runs as one profile with two data cells, `A-US` and `A-UK`, each with its own database, subject-key store, audit and decision logs, object store, secrets and region. *Source: DEC-52; DEC-24, DEC-25.*
**Consequences:**
- Personal data never crosses cells and no cross-cell join exists. A token from one cell is not found in the other.
- A request is served only by the cell whose jurisdiction roots contain the resolved residence and booking jurisdictions. Residence in one cell and booking in the other resolves to UNKNOWN, which denies every commercial activity.
- Only aggregate, non-personal metrics may be combined across the two cells (FR-51, FR-80), and only from each cell's own suppressed output. UK GDPR transfer questions are [LEGAL-CONFIRM] (planning/19, written 2026-11-10).
- Isolation checklist (planning/21 §2.1, §2.2): one instance and `deployment_id` per cell, so a session reaches one cell; token-free telemetry; per-cell audit, replay, outbox, backups, training inputs and support access; customer tokens not reversible by the subsystem and keyed per cell and track, with no cross-context linkage.

#### FR-71: Track B isolation
Track B is a separate, bank-hosted (on-premises) deployment with its own database, keys (deployment and demo-class), IdP realm, model registry, catalogue, packs, templates, logs and backups. *Source: DEC-53.*
**Consequences:** isolation tests prove that Track B never receives Track A data, models, features, training sets, keys or tokens (synthetic ones included), and that a Track A token, key or model reference is rejected by Track B and vice versa. Only release images, the contract, the control catalogue, the DSL and the schemas are shared.

#### FR-72: Institution-type overlay and Sharia controls
When `institution_type = ISLAMIC`, the PDP adds `PACK-INST-ISLAMIC` after the jurisdiction packs and before any host overlay. *Source: DEC-54; DEC-26.*
**Consequences:**
- The overlay is restrictive-only: CI rejects any overlay policy that grants an activity, widens an allow-list or raises an upper bound. The monotonicity property test covers it. It cannot touch locked L0 controls or the minors list.
- Four market-neutral controls are added to the catalogue: `CTL-CATALOGUE-ELIGIBILITY` (only items carrying the required attributes, e.g. `sharia_status = SHARIA_APPROVED`, are candidates), `CTL-TERMINOLOGY` (approved terms only, e.g. "finance"/"profit" not "loan"/"interest"; a violation blocks the text and falls back to the template), `CTL-PROHIBITED-CATEGORY` (never suggested, never in offers, never a ranking or targeting feature, never used to infer religion; budgeting labels stay neutral) and `CTL-SHARIA-APPROVAL`.
- `EXCL-FORBIDDEN-INFERENCE` includes "religion or religious observance" in every track. Term and category lists are SYNTHETIC placeholders [SHARIA-CONFIRM].
- The overlay implements the Sharia design constraints SDC-01..SDC-18 of planning/20 as mapped in planning/21 §4.4 (e.g. `contract_type` enum with no conventional types, Sharia basis/ISSC/certificate/KFS refs on every suggestion (contract 1.2.0 `sharia_disclosure`: required on product suggestions and credit referrals in ISLAMIC deployments, never in CONVENTIONAL ones; approved shape only with `SHARIA_APPROVED` and approval references; demo shape only with the illustrative attribute and watermark; incomplete disclosure = suppressed), Sharia questions handed to humans, re-review on Sharia change, Islamic-window bootstrap guard).

#### FR-73: Sharia approval state and `sharia_approver`
The Sharia approver records `NOT_REVIEWED` → `IN_SHARIA_REVIEW` → `SHARIA_APPROVED` / `SHARIA_REJECTED` on the overlay pack, Track B catalogue items, templates and reason-code texts (en + ar). In an ISLAMIC deployment the policy author's pack submit moves these artefacts to `IN_SHARIA_REVIEW` in the same audited step (contract 1.1.1); the Sharia approver then records the outcome. *Source: DEC-55, DEC-63.*
**Consequences:**
- Separation of duties (planning/20 SDC-17, strictest form per role matrix SOD-9): `sharia_approver` holds no `policy_author`, `policy_approver` or `policy_release` scope; per user, the Sharia approver differs from the maker and from the legal approver of the same pack version, and the activator differs from both; the vendor/developer never holds it; the Sharia Compliance Function's review is never automated.
- A Track B pack is `APPROVED` only with both legal and Sharia approval. Any customer-facing artefact without `SHARIA_APPROVED` gives BLOCK with reason `SHARIA_REVIEW_PENDING`.
- Demo only: inside an `ILLUSTRATIVE` bundle, `illustrative_sharia_assumption: ASSUME_COMPLIANT` (activator-confirmed; a separate attribute, never a value of the Sharia status) is honoured, and outputs add the watermark "illustrative - not a Sharia ruling".
- Interim approver roles: Head of Compliance plus the Sharia Compliance function / ISSC (roles, not named persons). Nothing reaches `APPROVED` or `ACTIVE` until named persons are recorded.

#### FR-74: Per-track synthetic catalogues and generator seeds
The tester can generate each cell's data from its own seed with one generator codebase, and load its SYNTHETIC catalogue: `CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH`, `CAT-B-AE-ISLAMIC-SYNTH` (planning/21 §5). *Source: DEC-56; BQ-9.*
**Consequences:**
- No real product names, rates, profit rates, fees or terms. Outputs are never mixed across cells; no FX conversion between cells.
- Islamic financing items (Murabaha and Tawarruq personal finance, Ijarah home finance, Islamic card) are reachable only as `CREDIT_REFERRAL`. Mudarabah investment deposits and Family Takaful always require human review. Late-payment charity is a descriptive, data-only attribute that is never calculated, charged, waived or used as a feature.
- Each cell's personas include the shared control cases (minor, age unknown, consent withdrawn, unknown, out-of-scope, cross-cell or cross-track, pending-review block, credit-request refusal). The Track B merchant taxonomy includes the prohibited categories so the exclusion path is tested.

#### FR-75: Per-track model registry scope
Registry entries carry `track` and `cell_scope`; models are trained, evaluated and validated separately per track, and nothing crosses tracks. *Source: DEC-57; DEC-47, DEC-48.*
**Consequences:** a model or ruleset whose `track` differs from the profile is refused at load. In the synthetic MVP one Track A rule baseline (rules only, no data) serves both cells; ML is trained and evaluated per cell, synthetic data included (planning/21 §6). Pooled US+UK training on real data needs a transfer assessment [LEGAL-CONFIRM]. Fairness reports are per cell (minimum group 100).

#### FR-76: Languages, locales and currency per track
Each cell declares its locales and currency: `A-US` en-US/USD, `A-UK` en-GB/GBP, `B-AE` ar-AE + en-AE/AED. *Source: DEC-59; DEC-41.*
**Consequences:** Track B explanation templates, reason-code texts and assistant templates exist in Arabic and English, each with translation review and Sharia terminology review (`CTL-TERMINOLOGY`); the number check applies to Arabic output. The dashboard supports right-to-left layout and WCAG 2.2 AA in both directions.

#### FR-77: Illustrative UK packs (GB, GB-ENG, GB-ENG-LDN)
Draft packs `PACK-GB`, `PACK-GB-ENG` and `PACK-GB-ENG-LDN` map candidate rows of planning/19 (written 2026-11-10) to controls; every policy stays PENDING_LEGAL_REVIEW. *Source: DEC-50; DEC-11, DEC-63.*
**Consequences:** each policy cites a planning/19 row; no legal text is invented. Content waits for planning/19 and a named legal approver; only `ILLUSTRATIVE` demo use is possible before that.

#### FR-78: Illustrative AE packs and Sharia overlay content
Draft packs `PACK-AE` and `PACK-AE-<emirate>`, plus the content of `PACK-INST-ISLAMIC` (term list, prohibited categories, catalogue eligibility attributes), map candidate rows of planning/20 (written 2026-11-10) to controls. *Source: DEC-50, DEC-54, DEC-55.*
**Consequences:** every jurisdiction policy is PENDING_LEGAL_REVIEW and every overlay artefact is not `SHARIA_APPROVED` until the named legal and Sharia approvers sign. Emirate and onshore status are [ASSUMPTION] until the bank confirms them.

#### FR-79: Per-track demo scripts
Two scripted standalone demos from one build: Track A (UJ-1..UJ-3, UJ-9) and Track B (UJ-10), both synthetic, `ILLUSTRATIVE`, watermarked and without an LLM. *Source: DEC-64; planning/21 §11.*
**Consequences:** each runs from a clean start with its own profile, seed and catalogue; the Track B demo runs on a separate deployment and URL.

#### FR-80: Dashboard read endpoints (customer list, stored transactions, portfolio)
Three additive contract operations close the GATE-3 gaps. *Source: DEC-62.*
**Consequences:**
- Customer directory `GET /v1/customer-directory` (contract 1.2.0, DEC-68; replaces `GET /v1/customers`, deprecated and not implemented): staff-only scope `ri.customers.list` (analyst, relationship staff); a required access purpose from a closed list registered per role and client (`CUSTOMER_SERVICING`, `INSIGHT_REVIEW`); an exact token lookup or a narrowing filter (segment or data-quality status); page <= 25, query total <= 100 and per-user daily quotas [ASSUMPTION defaults]; row filter by jurisdiction claim, per cell; customer token, resolved jurisdiction code, age-status class, data-quality flag and last-updated only; no name, contact, address, ZIP or account number; minors listed only as protected. Every call, allowed or denied, is audited (account, purpose, scope, filters, result count, outcome). Denials: `ACCESS_PURPOSE_INVALID`, `ACCESS_PURPOSE_NOT_REGISTERED`, `NARROWING_FILTER_REQUIRED`, `ACCESS_QUOTA_EXCEEDED`, `FORBIDDEN_SCOPE`. Auditors read the access records and hold no directory scope.
- `GET /v1/customers/{customer_token}/transactions` (new purpose-bound scope `ri.transactions.read`, relationship staff and analyst only, consent purpose `INSIGHT`): enriched view only (date, amount, currency, kind, category code, flags); no raw description, counterparty or merchant name; sensitive or prohibited categories as neutral codes; minors blocked except age/routing.
- `GET /v1/portfolio/summary` (new scope `ri.portfolio.read`, viewer and analyst): aggregate-only, as FR-51.
- Every operation carries `not_for_credit_decisioning: true` and is covered by role-matrix negative tests (FR-61).

#### FR-81: Track A human review on possible significant decisions
Pending planning/19 UQ4 (rows U02, U07: UK GDPR Arts 22A-22D [LEGAL-CONFIRM]), Track A (both cells) keeps meaningful human involvement. *Source: DEC-66; planning/21 §11A.*
**Consequences:**
- Offer suppression applies immediately (fail-safe) and creates a `review_item` for staff confirmation; the customer is never told a product is unavailable because of us. That review ends only as `REJECTED` (suppression confirmed) or `ESCALATED`; `CLEARED_TO_SHOW` is refused, so a reviewer can never release a suppressed offer (planning/21 §11A).
- Every retention action except `RA_NO_ACTION` needs `CTL-HUMAN-REVIEW` before it is shown or acted on.
- Vulnerability routing always ends with a person (`CTL-VULNERABILITY-ROUTE`).
- Encoded as `CTL-HUMAN-REVIEW` policies in `PACK-GB`, mirrored in `PACK-US` as internal policy (data, not code); they never carry `illustrative_assumption: ASSUME_NOT_APPLICABLE`, so the demo keeps the review step.

#### FR-82: Track B telemarketing contact caps
The AE pack carries `CTL-CONTACT-LIMIT` telemarketing parameters (planning/20 row E08 [LEGAL-CONFIRM]). *Source: DEC-67.*
**Consequences:**
- Consent dimensions purpose x channel x product type x contact method x language; frequency caps; DNCR check recorded as a host obligation.
- Until planning/20 IQ4 is answered, in-app banners, push and digital messages built from our outputs count as telemarketing (`channels_counted[]` includes them).
- Values are pack data, PENDING_LEGAL_REVIEW = BLOCK outside `ILLUSTRATIVE` demo bundles.

## 5. Non-Goals (Explicit)
- No credit approval, decline, scoring, limits, pricing, affordability or adverse action of any kind. *Source: planning/02 §3.*
- No customer messaging, account changes or product blocking.
- No real data, real bank connections or production deployment.
- No inference of sensitive characteristics. No personalised pricing (NYC-09).
- No fraud, AML or KYC. AML "restricted" flags are used only to suppress outreach (NYC-10).
- No 50-state rulebooks; no packs outside the two tracks (other US states, other UK nations, EU, Qatar, other GCC states, UAE free zones). UK (London) is in scope in Track A; DEC-13 is superseded (DEC-50).
- No Sharia rulings: the subsystem holds the bank's Sharia decisions as data and never decides whether a product or term is compliant.
- No hosted LLM, fine-tuning, deep learning or uplift modelling in the MVP.

## 6. MVP Scope

### 6.1 In Scope
- **Must:** FR-1..FR-16, FR-18..FR-23, FR-26..FR-31, FR-33..FR-36, FR-38..FR-41, FR-43..FR-46, FR-48, FR-49, FR-52, FR-53, FR-55, FR-61..FR-63, FR-68, FR-69..FR-76, FR-79, FR-80 (customer list and stored transactions), FR-81, FR-82.
- **Should:** FR-17, FR-24, FR-25, FR-32, FR-37, FR-42, FR-47, FR-50, FR-54, FR-56..FR-58, FR-60, FR-64..FR-67, FR-77, FR-78 (content gated on planning/19, planning/20 and named approvers).
- **Could:** FR-51 (and the portfolio part of FR-80), FR-59.

*Source: planning/09 §1 MoSCoW, mapped by facilitator. Contract v1 and the reference adapter are "Should" in file 09 but are needed for the plugin-parity KPI (BQ-11 #6).*

### 6.2 Out of Scope for MVP
- Minors' Tier-2 educational insight beyond the protected placeholder (DEC-22).
- Live credit-platform feed: contract fields exist, filled with synthetic stubs only (DEC-03, DECIDED-DEFAULT, GATE-3).
- Feature store, microservices, event broker, host SSO.
- Full DSR support, supranational jurisdictions, pooled cross-cell or cross-track models. Multi-deployment residency is now in scope as Track A cells and the separate Track B (DEC-52, DEC-53). *Source: planning/09 §2; planning/21.*

`[NOTE FOR PM]`: plugin parity (FR-57) stays Should by default (secondary metric SM-6); it is a BQ-11 KPI, so GATE-1 confirms whether it becomes Must for GATE-10. FR-55 is Must (R-7).

## 7. Success Metrics

**Primary** (BQ-11 KPIs, DECIDED-DEFAULT, ratify GATE-1, check GATE-10; pipeline validation on synthetic data only)
- **SM-1:** End-to-end flow for 100% of scripted personas. Validates FR-1, FR-48, FR-52.
- **SM-2:** 100% pass on control golden cases, in every cell: minor block, consent withdrawal, unknown/out-of-scope/cross-cell/cross-track jurisdiction, PENDING_LEGAL_REVIEW block, credit-request refusal; in Track B also `SHARIA_REVIEW_PENDING`, terminology fallback and prohibited-category exclusion. Validates FR-12, FR-30, FR-46, FR-63, FR-70..FR-73.
- **SM-3:** Churn baseline vs ML reported (PR-AUC, ROC-AUC, Brier, time split). ML is eligible for MRM review only at +0.02 PR-AUC [ASSUMPTION]. It stays in SHADOW until DEC-19 is resolved (DEC-47). Validates FR-15.
- **SM-4:** 100% of recommendations are catalogue items with reason codes and gate trace; 0 credit-decision outputs. Validates FR-19..FR-22.
- **SM-5:** 100% of outputs carry their versions and `not_for_credit_decisioning`. Validates FR-55, FR-31.

**Secondary**
- **SM-6:** Plugin parity on the golden set. Validates FR-57.
- **SM-7:** p95 under 1 s for evaluate calls in the demo environment. Validates NFR-1.
- **SM-8:** The same seed gives identical data, metrics and subgroup report. Validates FR-1, FR-4, FR-13.

**Counter-metrics (do not optimise)**
- **SM-C1:** Recommendation surfacing or acceptance rate. Do not raise it by loosening gates. Counterbalances SM-4.
- **SM-C2:** Churn recall at the cost of subgroup disparity or label leakage. Counterbalances SM-3.
- **SM-C3:** DENY or BLOCK rate. Do not reduce it by relaxing fail-safes; only legal approval changes outcomes. Counterbalances SM-1.
- **SM-C4:** Assistant answer rate. Refusals are correct behaviour. Counterbalances UJ-4.

## 8. Open Questions
Status follows file 12 §H (§H.7 wins where it differs). Items 1-12 are BLOCKED-EXTERNAL; the build proceeds on the stated safe assumption.

1. **OQ-1 / DEC-10 (N1):** bank charter, regulator, Part 500 status, products and channels; now per track (Track A: US charter and UK authorisation; Track B: UAE licence, onshore status, emirate, Sharia governance). Resolve by GATE-2A.
2. **OQ-2 / DEC-11 (N2):** named policy approver and backup. Interim role-based approvers decided (DEC-63: Track A Head of Compliance; Track B Head of Compliance + Sharia Compliance / ISSC); named persons still BLOCKED-EXTERNAL. Resolve by GATE-3A; hard stop for GATE-4A and GATE-8A.
3. **OQ-3 / DEC-07:** SOC 2 / ISO 27001 audit scope. Build as if in scope. Resolve by GATE-12.
4. **OQ-4 / DEC-18:** consent purpose taxonomy and grantor model [LEGAL-CONFIRM]. Resolve by GATE-4A.
5. **OQ-5 / DEC-19:** model-risk scope and tier, and the LLM governance framework. Resolve by GATE-7.
6. **OQ-6 / DEC-20:** retention periods per entity [LEGAL-CONFIRM]. Resolve by GATE-12.
7. **OQ-7 / DEC-21:** fairness proxies, thresholds and escalation [LEGAL-CONFIRM]. Resolve by GATE-6.
8. **OQ-8 / DEC-22:** NY Child Data Protection Act position, consent for 16-17, training on minors' data [LEGAL-CONFIRM]. Resolve by GATE-6.
9. **OQ-9 / DEC-23:** exclusion, forbidden-inference and proxy list contents [LEGAL-CONFIRM]. Resolve by GATE-4A.
10. **OQ-10 / DEC-25:** attributes that connect a customer to a jurisdiction [LEGAL-CONFIRM]. Resolve by GATE-4A.
11. **OQ-11 / DEC-30:** host IdP, claims and mTLS. Resolve by GATE-11.
12. **OQ-12 / DEC-33:** event transport, and whether the host wants MVP events. Resolve by GATE-11.
13. **OQ-13 / MS1-MS4:** platform prd.md (platform FR-39, FR-50/51, FR-79..86), the presentation, retail research, and a clean PDF. Gap accepted; re-request.
14. **OQ-14 (RESOLVED, DEC-48):** the minimum fairness group size is 100 per cell, as in planning/08 CMP-09. MRM sets the final value under DEC-21 at GATE-6.
15. **OQ-15 (RESOLVED, DEC-49, DECIDED-DEFAULT GATE-1):** "credit-building need" is removed from the v1 taxonomy.
16. **OQ-16:** legal questions in file 13 §6 (Q1-Q24), routed per planning/12 §G.2. Wording of the host terms that prohibit credit use (DEC-42) [LEGAL-CONFIRM] by GATE-11.
17. **OQ-17 (RESOLVED 2026-11-10, 12 §H.7.2):** the GATE-1 batch (DEC-01, 02, 04, 09, 43, 44, 49; BQ-1..BQ-7, BQ-11), GATE-2 batch (DEC-05, 08) and GATE-3 batch (DEC-03, 12, 31, 38 draft) are DECIDED (developer-delegated), manager countersignature recommended. DEC-13 and DEC-28 are SUPERSEDED. Still DECIDED-DEFAULT: DEC-47 (GATE-7, MRM), DEC-37 and DEC-40 (GATE-8), DEC-27 (GATE-11), DEC-36 (GATE-12). Starting application code still needs the developer's explicit implementation instruction.

18. **OQ-18 (RESOLVED by DEC-44, DECIDED-DEFAULT, ratify GATE-1):** the demo runs the illustrative packs in pack state `ILLUSTRATIVE`, in a standalone deployment with `demo=true` on synthetic data only. A policy with an explicit `illustrative_assumption` is evaluated on it, and every output is watermarked "illustrative - not legal advice - not for real customers". Plugin and production deployments refuse the pack at load. If DEC-44 is not ratified, a config switch turns it off and the demo shows blocked outcomes only.
19. **OQ-19 (RESOLVED by DEC-45, DECIDED):** `VALIDATED` is the "loaded but unapproved" state, used by the simulator only. The runtime bundle is L0 plus `ACTIVE` packs (or `ILLUSTRATIVE` packs in the demo). L0 is restrictive-only: the developer approves it as internal policy with a second-person review, standalone signs it with the demo-class key, and COMP reviews it at GATE-3A.
20. **OQ-20 (RESOLVED by DEC-47, DECIDED-DEFAULT, MRM at GATE-7):** the rule baseline is a registry `RULESET`, labelled "non-model", and serves churn and needs. ML stays in SHADOW (synthetic data, logged, never shown). APPROVED is unavailable until DEC-19 is resolved.
21. **OQ-21 / DEC-50 (RESOLVED 2026-11-10, DECIDED by the developer):** two tracks (Track A NYC + London; Track B Islamic bank, UAE onshore [ASSUMPTION]); see the note at the top and §4.13. Remaining per-track content gates: planning/19 (UK) and planning/20 (UAE + Sharia), both written 2026-11-10, plus named legal and Sharia approvers.
22. **OQ-22 [SHARIA-CONFIRM]:** Sharia term list, prohibited categories, product structures and the ISSC approval process (planning/20, written 2026-11-10). Track B non-demo bundles wait for them.
23. **OQ-23 [LEGAL-CONFIRM]:** whether the US and UK cells may ever share processing (international transfer mechanism); AE age of majority (21 assumed, restrictive).
24. **OQ-24 / DEC-65 (BLOCKED-EXTERNAL):** UK London operating entity type (UK-incorporated bank, UK branch of the NYC bank, or other) and the UK data controller (planning/19 UQ1, Q-U2). Until known, no entity type is assumed; GB packs stay PENDING_LEGAL_REVIEW = BLOCK; FR-77 content scope waits.
25. **OQ-25 [LEGAL-CONFIRM]:** planning/19 UQ4 (Art 22A significant decisions; FR-81 interim rule) and planning/20 IQ4 (in-app as telemarketing; FR-82 interim rule), IQ3 (any hosting beyond the bank's own UAE infrastructure; AD-27).

## 9. Assumptions Index
- §4.1 FR-1: the persona set includes a declared-occupation shift worker (Alex) and a `VARIABLE_INCOME` persona.
- §4.3 FR-11: SOC 2018 mapping (groups 37, 45, 47, 49, 51, 53); `VARIABLE_INCOME` rule CV > 0.3 or irregular intervals over 6 months (BQ-2).
- §4.3 FR-13: minimum group size 100 per cell (DEC-48; final value MRM under DEC-21).
- §4.4 FR-15: ML adoption margin +0.02 PR-AUC (BQ-11).
- §4.6 FR-24: review SLA of 2 business days (DEC-37).
- §4.12 FR-66: pilot RPO 24 h, RTO 8 h, quarterly restore test (DEC-36).
- §4.6 FR-23: synthetic demo contact defaults, 1 suggested outreach per 7 days and quiet hours 21:00-08:00, used only when no pack or host value exists (R-7).
- §10 NFR-12: contract deprecation window of 6 months; rate limit 50 req/s per client, burst 100 (DEC-31).
- §4.13: Track B assumed UAE onshore, emirate unknown; `GB-ENG-LDN` is a deployment-defined local code; AE age of majority 21 (restrictive); portfolio small-cell minimum 20; Qard-based current account structure (DEC-50, DEC-56, DEC-60, DEC-62).
- planning/07 §4.1: DQ thresholds are [ASSUME] until calibrated (DEC-39).

## 10. Cross-Cutting NFRs
*Source: planning/05 §9; planning/08 §2.*
- **NFR-1 Latency:** evaluate API p95 under 1 s in the demo (12 months of transactions). [REC]
- **NFR-2 PDP latency:** in-process with a cached bundle, p95 under 5 ms per decision [ASSUMPTION].
- **NFR-3 Batch:** 50k synthetic customers per deployment within 30 min on a laptop. [REC]
- **NFR-4 LLM-down availability:** full function through templates.
- **NFR-5 Policy-store availability:** not ready at start-up without a verified bundle, meaning the DB is up, L0 is verified and every configured pack is verified and loadable (DEC-45). An `ILLUSTRATIVE` pack outside a standalone demo makes the instance not ready (DEC-44). At runtime the last verified bundle is kept until max-age, then `CTL-DEFAULT-DENY` applies. The bundle is never used past `review_due` (DEC-14).
- **NFR-6 Replay determinism:** a stored request with stored versions (including packs and `as_of`) reproduces identical output and policy outcome.
- **NFR-7 Observability:** PII-free structured logs, OpenTelemetry, and the alerts listed in planning/08 SEC-21.
- **NFR-8 Accessibility:** the dashboard meets WCAG 2.2 AA (DEC-41, DECIDED).
- **NFR-9 Offline-capable:** no mandatory external calls; runs inside the bank boundary (SEC-12).
- **NFR-10 Security baseline:** SEC-01..SEC-21 (OIDC everywhere except `/health`, TLS 1.2+, encryption at rest, signed images and packs, SBOM, secrets never in the repo, rate and size limits).
- **NFR-11 Privacy baseline:** PRV-01..PRV-13 (tokens only, minimisation, no sensitive inference, PIA before any non-synthetic use).
- **NFR-12 Contract stability:** SemVer; `/v1` major in the URL; each version supported at least 6 months after its successor [ASSUMPTION]; default rate limit 50 req/s, burst 100 [ASSUMPTION] (DEC-31, DECIDED-DEFAULT, GATE-3).
- **NFR-13 Reproducibility:** same seed, same data, metrics and reports.
- **NFR-14 Isolation:** automated tests prove no personal data, token, key, model or training set crosses a cell (Track A) or a track (A/B), and that the profile, pack, catalogue and model `track` tags match at bootstrap (DEC-51..DEC-53, DEC-57).
- **NFR-15 Data location:** each cell runs in its declared region; Track B runs only on bank-hosted infrastructure, with no external network dependency (extends NFR-9; DEC-53, DEC-58).

## 11. Constraints and Guardrails
- **Purpose limitation:** every output carries `not_for_credit_decisioning` (DEC-42).
- **Minors:** two-tier block list locked in L0 (planning/14 §13.3).
- **Data:** synthetic only (12 §H.4 condition 1).
- **Packs:** PENDING_LEGAL_REVIEW, enforced as BLOCK (condition 2). The demo-only `ILLUSTRATIVE` state (DEC-44) is not approval and cannot load outside a synthetic standalone demo.
- **Models:** the rule baseline serves; ML is SHADOW only until DEC-19 is resolved (DEC-47).
- **Market:** two tracks (DEC-50): Track A NYC + London, Track B Islamic bank (UAE onshore [ASSUMPTION]). Track B never mixes data or models with Track A; personal data never crosses Track A cells. Every overlay tightens only.
- **Sharia:** the subsystem never makes a Sharia ruling; Sharia content is the bank's data, approved by `sharia_approver` [SHARIA-CONFIRM].
- **No-go triggers (12 §H.4):** any request to use real data, activate a pack, consume PD/grade, or enable a hosted LLM before its blocking item is resolved.

## 12. Compliance and Regulatory
Applicability is decided per row of the file 13 matrix (federal F01-F18, NY State S01-S17, NYC C01-C07), and for Track A UK and Track B UAE per rows of planning/19 and planning/20 (both written 2026-11-10), by the bank's legal approver; Sharia content by the bank's Sharia approver. The PRD asserts no law. Design controls derived from the matrix are planning/08 §2.4 NYC-01..NYC-11, each switched on by policy only after legal confirmation [LEGAL-CONFIRM]. Regulatory facts these decisions depend on are listed in planning/12 §G.1 (SR 26-2 is supervisory guidance; Reg B amendment litigation; limited reach of NYCHRL 8-107(5)(d); TCPA 91 FR 67214).

## 13. Stakeholders and Approvals
Gates and decision-makers follow planning/11 §3:
- GATE-1 (scope and definitions): Manager.
- GATE-2 (architecture): Manager and tech leads.
- GATE-2A, GATE-4A, GATE-8A: legal approver (DEC-11; interim roles DEC-63), plus the Sharia approver for Track B.
- GATE-1, GATE-2, GATE-3 batches: developer-delegated ratification on 2026-11-10 (12 §H.7.2), manager countersignature recommended. GATE-3 interim integration owner: the developer (DEC-63).
- GATE-6, GATE-8, GATE-12: legal/compliance (advisory).
- GATE-7: model risk.
- GATE-9: Manager and security.
- GATE-10: Manager.
- GATE-11: platform team.

## Change log
- 2026-11-09 (readiness fixes): market note at the top and OQ-21 (DEC-50). UJ-1, UJ-2 and UJ-6 updated for the `ILLUSTRATIVE` demo state and the pack lifecycle (DEC-44, DEC-45). Glossary adds pack state, ILLUSTRATIVE and SHADOW. FR-13 sets the group size to 100 (DEC-48). FR-14 removes the credit-building need (DEC-49). FR-15 and FR-16 cover the rule baseline, ML SHADOW and the RULESET/ML_MODEL registry (DEC-47). FR-30, FR-33, FR-35, FR-36, FR-38, FR-40, FR-53, FR-54 and FR-55 follow DEC-44..46. SM-3, OQ-14, OQ-15 and OQ-17..OQ-20 updated, NFR-5 updated, §11 constraints updated. No NYC content was rewritten.
- 2026-11-09 (readiness R-7, PH-3): FR-55 is Must (DEC-42; SM-5) and moved in §6.1; FR-57 priority stated (Should by default, GATE-1 may promote); FR-20 names the data source of the credit-platform "pause offers" stub (preference ingest, CI-2); FR-23 defines where contact-limit and quiet-hour values come from, with synthetic demo defaults as [ASSUMPTION] (Assumptions Index updated). Normative contract: `contracts/openapi/retail-intelligence-v1.yaml`.
- 2026-11-10 (two-track scope, planning/21, 12 §H.7): market note replaced (DEC-50 resolved: Track A NYC + London, Track B Islamic bank); Vision, JTBD (manager, new Sharia approver), UJ-9 (Track A two cells) and UJ-10 (Track B) added; glossary adds track, deployment profile, cell, institution overlay, Sharia approval state and per-cell catalogues; FR-1 section title, FR-12 (age threshold per jurisdiction, DEC-60), FR-26, FR-38, FR-47 (LLM hosting per track, DEC-58), FR-51, FR-53, FR-54 updated; new §4.13 FR-69..FR-80; §5, §6, SM-2, OQ-1, OQ-2, OQ-17, OQ-21 updated, OQ-22, OQ-23 added; NFR-14, NFR-15 added; §9, §11, §12, §13 updated. Minors, no-credit-decision and synthetic-only rules unchanged.
- 2026-11-10 (alignment with planning/19 and planning/20, written 2026-11-10): 19/20 cited as written; FR-72 SDC-01..SDC-18 pointer (planning/21 §4.4); FR-73 SoD per SDC-17; FR-81 (DEC-66) and FR-82 (DEC-67) added (Must); OQ-24 (DEC-65) and OQ-25 added.
- 2026-11-10 (readiness re-run fixes): scope note cites FR-69..FR-82; FR-73 defines the submit transition to IN_SHARIA_REVIEW and records the attribute-only illustrative Sharia assumption.
- 2026-11-10 (independent two-track validation, handoff/two-track-validation-report-2026-11-10.md): FR-51 aggregate disclosure control (planning/21 §10A); FR-69 profile integrity and no env/flag override (§3.1); FR-70 isolation checklist and token scheme (§2.1, §2.2); FR-73 SoD per SOD-9 with per-user checks; FR-75 ML per cell in the synthetic MVP; FR-81 suppression review cannot release an offer, no not-applicable illustrative assumption on DEC-66 policies. Tightenings only (12 §H.7.3).
- 2026-11-10 (contract 1.2.0, V-8 and V-9): FR-80 customer list replaced by the purpose-bound customer directory (DEC-68); FR-72 states the SDC-11 disclosure contract rules. FR count unchanged (82).
