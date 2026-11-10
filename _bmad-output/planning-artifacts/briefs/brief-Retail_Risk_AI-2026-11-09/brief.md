---
title: "Product Brief: Retail Banking AI Intelligence Subsystem (two tracks: Conventional NYC + London; Islamic bank)"
status: final
created: 2026-11-09
updated: 2026-11-10
workflow: bmad-product-brief (v6.12.4), create intent, fast path, facilitated from planning pack
sources: [planning/01, planning/02, planning/09, planning/12 §H, planning/18, planning/21]
---

# Product Brief: Retail Banking AI Intelligence Subsystem

> **Scope (DEC-50, DECIDED by the developer 2026-11-10; planning/21).** Two separate tracks on one shared core: **Track A "Conventional"** = NYC (US federal, NY State, NYC) and London (UK, England, London), one deployment profile with separate US and UK data cells; **Track B "Islamic"** = an Islamic bank, assumed UAE onshore [ASSUMPTION], a separate bank-hosted deployment with a tighten-only Sharia overlay that never shares data or models with Track A. UK applicability (planning/19) and UAE + Sharia applicability (planning/20) were written on 2026-11-10. DEC-13 (UK deferred) is superseded.

> Imported from the approved-with-conditions planning pack (`planning/00-18`). Facilitator answers come from the pack and the file 12 §H approval-pass decisions. Items marked **DECIDED-DEFAULT** need manager ratification at the gate shown; since 2026-11-10 the GATE-1, GATE-2 and GATE-3 batches are DECIDED (developer-delegated), manager countersignature recommended (planning/12 §H.7). Items marked **[LEGAL-CONFIRM]** need the bank's legal/compliance teams. This brief is not legal advice.

## Executive Summary

The Retail Banking AI Intelligence Subsystem analyses how retail customers manage money. It finds their likely financial needs, estimates disengagement (churn) risk, and gives staff explainable, catalogue-grounded product and retention suggestions. It is one module of a wider banking platform that has no implementation yet. All host-facing parts are therefore versioned contracts and adapters around **one shared intelligence core**. The same core runs as a **standalone demo** (own API, dashboard, synthetic data) and as a **plugin** called by the host platform. *Source: planning/02 §1; planning/04 §1.*

There are **two tracks** (DEC-50). **Track A** serves New York City (US federal, New York State and NYC rules) and London (UK, England and London rules) from one deployment profile, with US and UK personal data in separate cells and regions (DEC-52). **Track B** serves an Islamic bank, assumed UAE onshore [ASSUMPTION], as a separate bank-hosted deployment (DEC-53); its Sharia rules are a tighten-only institution overlay: catalogue items with recorded `SHARIA_APPROVED` status only, approved terminology ("finance"/"profit", not "loan"/"interest"), prohibited categories never offered or used as features, and Sharia approval next to legal approval (DEC-54, DEC-55). Rules are not written into code. They live in versioned, signed **policy packs**, read by one policy decision point (PDP) inside the core. Packs stay `PENDING_LEGAL_REVIEW` (and Track B artefacts unapproved by the Sharia function) and are enforced as BLOCK until named approvers sign them. Other US states and other countries are future expansion, added as packs and cells without changing the core. *Source: planning/21; planning/02 §8; planning/14 §1, §12; planning/12 §H.7 DEC-50..DEC-64.*

Why now: the platform needs retail intelligence that is safe by construction. That means advisory only, no credit decisioning, protection for minors, sensitive-attribute exclusion, and a full replayable audit trail. It must also be ready to plug into a host whose stack is still unknown. *Source: planning/05 §1 PR-1..PR-9; planning/03 §1.*

## The Problem

- Bank staff have no single, explainable view of a customer's financial behaviour, needs and disengagement risk. Without it, retention and product conversations are generic or late. *Source: planning/01; planning/02 §1.*
- The research PDF defines no retail churn, needs or recommendation requirements (planning/12 §B MS3). The platform's own prd.md (platform FR-39, FR-50/51, FR-79..86) and the manager presentation were never provided (MS1, MS2; gap accepted at GATE-1 per 12 §H.3).
- Regulatory applicability depends on the bank's charter, products and channels, which are unknown (DEC-10, BLOCKED-EXTERNAL). The product must therefore work safely before legal answers exist. *Source: planning/13 §2.1; planning/12 §H.1.*

## The Solution

Nine modules on one core: Transaction Intelligence (M2), Customer Intelligence (M1), Segmentation (M3), Financial Needs (M4), Churn (M5), Product Recommendation (M6), Retention Intelligence (M7), Risk & Compliance with the PDP (M8), and a Central AI Orchestrator (M9). *Source: planning/05 §2; planning/06 §1.*

- Rules and deterministic money calculations come first. ML is used only when it beats a rules baseline on a time-split evaluation. The LLM is for wording only and is **disabled in the MVP**: explanations come from templates (DEC-06, DECIDED). *Source: planning/05 §1 PR-1, PR-2; planning/12 §H.1.*
- Every output carries a policy `decision_id`, its versions and reason codes. Every output is labelled `not_for_credit_decisioning` (DEC-42, DECIDED; host-terms wording [LEGAL-CONFIRM] by GATE-11).
- Outputs are advisory. The subsystem never changes accounts, never contacts customers and never approves or declines credit. Credit appears only as a "refer to the credit platform for assessment" referral. *Source: planning/02 §3, §4; planning/14 §13.1.*

## What Makes This Different

This brief makes no market claims. The differentiators are design properties:
1. **Jurisdiction rules as data.** A new state or country is a new pack, not a code change. *Source: planning/14 §12.*
2. **Fail-safe by default.** Unknown or out-of-scope jurisdiction, pending legal review, unknown age, unknown consent and PDP errors all lead to the restrictive outcome (DEC-14, DEC-24 DECIDED). *Source: planning/14 §10.*
3. **One core, two modes, proven by parity tests.** The standalone and plugin paths must give identical outputs on the golden set. *Source: planning/04 §5.1, §7.*
4. **Safety as invariants, not features.** There is no credit-decision type in any contract. A two-tier minors block list is locked against packs and host overlays. A host overlay may tighten rules, never loosen them (DEC-17 DECIDED). *Source: planning/14 §11, §13.*

## Who This Serves

- **Primary (MVP demo):** the project manager, who approves the demo (GATE-10). Also bank staff roles: relationship staff, retention reviewers (`retention_reviewer`, DEC-37 DECIDED-DEFAULT), analysts, compliance reviewers, auditors, and the policy author, approver and release roles; in Track B also the Sharia approver (`sharia_approver`, DEC-55). *Source: planning/15 §5.1; planning/08 SEC-02; planning/21 §4.3.*
- **Secondary:** host-platform integrators using contract v1. Legal/compliance, who decide applicability and approve packs (DEC-11: interim role-based approvers per track decided, DEC-63; named persons **not yet recorded**, BLOCKED-EXTERNAL). The bank's Sharia Compliance function / ISSC for Track B. Model-risk (DEC-19, BLOCKED-EXTERNAL).
- **Customers are data subjects, not users** in the MVP. The MVP covers adults in the two tracks' jurisdictions (age of majority per jurisdiction: US and GB 18, AE 21 [ASSUMPTION, restrictive], DEC-60). Minors appear only as synthetic records that prove the block path (DEC-09, DECIDED developer-delegated). *Source: planning/09 §1.*

## Success Criteria

These are the demo KPIs from BQ-11 (DECIDED-DEFAULT, ratify GATE-1, checked GATE-10). They validate the pipeline on synthetic data; they are not business evidence:
1. 100% of scripted personas run end to end.
2. 100% pass on control golden cases in every cell: minor block, consent withdrawal, unknown, out-of-scope, cross-cell or cross-track jurisdiction, PENDING_LEGAL_REVIEW block, credit-request refusal; in Track B also Sharia-review block, terminology fallback and prohibited-category exclusion.
3. Churn rules baseline and ML reported with PR-AUC, ROC-AUC and Brier score on a time split. ML becomes eligible for model-risk review only if it beats the baseline by at least 0.02 PR-AUC [ASSUMPTION]. Until DEC-19 is resolved, the rule baseline serves and ML runs in SHADOW only (DEC-47).
4. 100% of recommendations are catalogue items with reason codes and a gate trace. Zero credit-decision outputs.
5. 100% of outputs carry their rule or model version and `not_for_credit_decisioning`.
6. Plugin parity: outputs identical to standalone on the golden set.
7. Evaluate calls have p95 latency under 1 s in the demo environment.
8. The same seed gives identical data, metrics and subgroup report.

Business pilot KPIs (retention rate, uplift) are deferred to a real-data pilot. *Source: planning/12 §H.2 BQ-11; planning/09 §4.*

## Scope

**In (MVP, Track A NYC + London and Track B Islamic bank, adults, synthetic data only):**
- Synthetic data generator.
- Transaction intelligence.
- Profile and features.
- Multi-membership segments: Gen Z 18-29, blue-collar from consented self-declared occupation only, `VARIABLE_INCOME`.
- Rule-based needs.
- Churn rules baseline plus one transparent ML model.
- Gated recommendations.
- Retention suggestions with outcome capture and a human review queue.
- Jurisdiction policy layer.
- Deployment profiles, Track A data cells (`A-US`, `A-UK`) and a separate Track B deployment (DEC-51..DEC-53).
- Institution-type (Sharia) overlay with four controls and a Sharia approval state (DEC-54, DEC-55).
- Per-cell synthetic catalogues: US and UK conventional, and an Islamic catalogue (Wakala and Mudarabah deposits, Takaful, and Murabaha, Tawarruq, Ijarah and Islamic card as credit referral only) (DEC-56).
- Per-track models, Arabic and English for Track B, and two scripted demos (DEC-57, DEC-59, DEC-64).
- Illustrative federal, NY State and NYC packs, and UK and AE packs once planning/19 and planning/20 allow, all left PENDING_LEGAL_REVIEW. The demo runs them in a watermarked `ILLUSTRATIVE` pack state that only a synthetic standalone demo can load (DEC-44).
- Policy simulator and maker-checker workflow.
- Template-only explanations and assistant.
- Standalone API and dashboard.
- Contract v1 with a mock-host parity test.

*Source: planning/09 §1 (MoSCoW); planning/12 §H.2 BQ-1..BQ-7.*

**Out:**
- Credit decisions, limits, pricing, affordability or adverse action.
- Real data or real bank connections.
- Customer messaging and account actions.
- Any commercial use of minors' data: profiling, prediction or model training.
- Sensitive-trait inference.
- Fraud, AML or KYC.
- Other US states, other UK nations, EU, Qatar, other GCC states and UAE free zones.
- Sharia rulings: the bank's Sharia function decides; we hold its decisions as data.
- Hosted LLM (if ever enabled: Track B bank on-prem only, Track A in-region, DEC-58).
- Live credit-platform feed: synthetic stubs only (DEC-03, DECIDED-DEFAULT, GATE-3).
- Deep learning, uplift modelling, production deployment.

*Source: planning/02 §3; planning/09 §1-2.*

**Conditions on everything above (12 §H.4):**
- Synthetic data and synthetic catalogues only.
- No pack is approved or activated until DEC-11 names an approver (and, for Track B, a named Sharia approver).
- The GATE-1, GATE-2 and GATE-3 batches are ratified by developer delegation (2026-11-10); manager countersignature recommended.
- No application code until the developer gives an explicit implementation instruction.

## Vision

If the MVP demo succeeds, the subsystem becomes the platform's retail-intelligence plugin. A pilot then runs on real data, after legal resolves the charter (DEC-10), consent purposes (DEC-18), retention periods (DEC-20), proxy lists (DEC-23) and jurisdiction attributes (DEC-25). New markets come through the add-a-jurisdiction procedure. Retention effectiveness is measured on real outcomes before any uplift modelling. *Source: planning/09 §2; planning/14 §12; planning/12 §H.4.*

## Open items carried by this brief

- **Developer-delegated ratification (2026-11-10, 12 §H.7.2; manager countersignature recommended):** DEC-01, 02, 04, 09, 43, 44, 49 and BQ-1..BQ-7, BQ-11 (GATE-1); DEC-05, 08 (GATE-2); DEC-03, 12, 31, 38 (GATE-3). DEC-13 and DEC-28 superseded. Still DECIDED-DEFAULT: DEC-27, 36, 37, 40, 47 and BQ-8.
- **BLOCKED-EXTERNAL:** DEC-07, 10 (per track), 11 (named persons), 18, 19, 20, 21, 22, 23, 25, 30, 33. DEC-50 is resolved. Full list in `planning/12 §H.7`; detail in `prds/.../prd.md §8`.
- **Two-track decisions:** DEC-50..DEC-67 (planning/12 §H.7; planning/21). UK matrix planning/19 (rows U01-U25) and UAE/Sharia matrix planning/20 (rows E01-E21, SH01-SH14, design constraints SDC-01..SDC-18) written 2026-11-10; DEC-65 (UK entity type) BLOCKED-EXTERNAL.
- **Readiness-fix decisions (planning/12 §H.6):** DEC-44 (ILLUSTRATIVE demo state, ratify GATE-1), DEC-45 (pack lifecycle), DEC-46 (governance endpoints), DEC-47 (rule baseline serves, ML shadow; MRM at GATE-7), DEC-48 (fairness minimum group size 100), DEC-49 (credit-building need removed; GATE-1).
- **Missing sources:** MS1-MS4 (planning/12 §B).

## Change log
- 2026-11-09 (readiness fixes): market note (DEC-50); success criterion 3 (DEC-47); scope line on the ILLUSTRATIVE demo packs (DEC-44); open items updated. NYC content unchanged.
- 2026-11-10 (two-track scope): market note replaced by the two-track scope (DEC-50 resolved); executive summary, users, success criterion 2, scope in/out, conditions and open items updated for planning/21 and 12 §H.7. Minors, no-credit-decision and synthetic-only rules unchanged.
- 2026-11-10 (readiness re-run fixes): Track B overlay wording no longer asserts compliance ("catalogue items with recorded `SHARIA_APPROVED` status only").
