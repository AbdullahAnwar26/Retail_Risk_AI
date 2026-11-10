# Retail Banking AI: Manager Approval Pack

Prepared 2026-11-10 for the project manager. Planning only. No application code has been written.

**How to read the labels.** Every item carries one label:

| Label | Meaning |
|---|---|
| **[Developer decision]** | Made by the developer on delegated authority (file 12 §H). Binding for planning; your countersignature is recommended. |
| **[Proposed design]** | How we plan to build it. Not yet built. |
| **[Needs legal confirmation]** | The bank's legal or compliance team must confirm. No legal review has happened yet. |
| **[Needs Sharia confirmation]** | The bank's Sharia Compliance function or Sharia committee (ISSC) must confirm. No Sharia review has happened yet. |
| **[Needs manager approval]** | Only you (or the bank, through you) can decide or supply this. |

Nothing in this pack is legal advice or a Sharia ruling. No licence, charter, approver name or manager decision is assumed.

## 1. What the system will do, and what it will not do

**What it does** [Developer decision]. The Retail Banking AI subsystem gives bank staff insight into retail customers. It:
- Reads (synthetic) transactions and categorises spending.
- Groups customers into segments and spots financial needs (for example, a missing emergency buffer or a budgeting need).
- Flags customers who may leave the bank (churn), using simple rules first.
- Suggests suitable products, services or education from a fixed catalogue, with a reason for each.
- Suggests retention actions for staff to review.
- Explains every output with approved, fixed text templates.

It runs on its own (with its own dashboard) or plugged into a future host banking platform. Both modes use the same core.

**What it will not do** [Developer decision]:
- It never makes a credit decision: no approve, decline, limit, pricing, affordability or adverse action. Credit products appear only as a referral to the credit team (`CREDIT_REFERRAL`).
- It never uses real customer data or connects to a real bank system at this stage.
- It never profiles or markets to under-18s.
- It never infers religion, health, ethnicity or other sensitive traits, and never targets on them.
- It never gives legal advice or a Sharia ruling.

## 2. The two deployment tracks

**[Developer decision]** (DEC-50, design in file 21). The same system is deployed in two separate tracks.

| | Track A: Conventional bank | Track B: Islamic bank |
|---|---|---|
| Markets | New York City (US federal, NY State, NYC) and London (UK, England, London) | Assumed UAE onshore. **[Needs manager approval]**: not yet confirmed (file 20 IQ1) |
| Data | Two separate data "cells": `A-US` (US region) and `A-UK` (UK region). Personal data never crosses between them (DEC-52) | One cell `B-AE`, on the bank's own systems in the UAE (DEC-53) |
| Products | Conventional synthetic catalogue | Synthetic Islamic catalogue; only Sharia-approved items may be suggested (DEC-54, DEC-56) |
| Languages | English (US and UK) | Arabic and English, right-to-left layout (DEC-59) |
| Extra layer | None | A "Sharia layer" that can only make rules stricter (DEC-54) |

**Why separate** [Developer decision]:
- **Different rules.** Islamic banking adds Sharia rules on products, wording and approvals. Keeping them in their own track avoids mistakes leaking either way.
- **Data location.** Track B customer data must stay in the UAE on the bank's own systems (file 20 IQ3). Sharing a deployment with Track A would break that.
- **Simple, testable rule.** "Nothing crosses between tracks" is easy to prove. Inside Track A, US and UK data are also kept apart, which avoids any UK-to-US transfer question. **[Needs legal confirmation]** before any real data (DEC-52).

## 3. What is shared and what is isolated

**[Proposed design]** (file 21 §9).

| Item | Shared across tracks? | Track A | Track B |
|---|---|---|---|
| Core software and API contract | **Shared** (same release) | Same | Same |
| Rule catalogue (list of control types) | **Shared** | Same | Same |
| Deployment settings ("profile") | Separate | Track A profile | Track B profile |
| Customer databases, keys, audit logs | Separate | One per cell (`A-US`, `A-UK`) | `B-AE` only |
| Legal rule packs | Separate | US, NY, NYC; UK, England, London | UAE + Sharia layer |
| Product catalogue and wording | Separate | English | Arabic + English |
| Models and training data | Separate (even synthetic ones) | Track A only | Track B only |
| User logins and approvers | Separate | Legal approver | Legal approver **and** Sharia approver |
| Combined reporting | Only anonymous totals inside Track A | Yes, totals only, small groups hidden | Never combined with Track A |
| Hosting | Separate | Bank-controlled, per region | Bank's own systems in the UAE only |

## 4. Major safeguards

| Safeguard | What it means | Label | Reference |
|---|---|---|---|
| No credit decisions | Credit appears only as a referral. No approve, decline, limit, price or affordability output | Developer decision | DEC-42, file 12 §H.4 |
| "Not for credit decisioning" label | Every output and event carries `not_for_credit_decisioning`. Host contract wording still needs legal | Developer decision; wording **Needs legal confirmation** | DEC-42 |
| Synthetic data only | No real customers, products, rates or bank systems until legal items are resolved | Developer decision | 12 §H.4 condition 1; DEC-56 |
| Minors protected | Two-tier block list; Tier 1 is never relaxed. Adult age set per country (US 18, UK 18, UAE assumed 21) | Developer decision; ages **Needs legal confirmation** | DEC-09, DEC-60, file 14 §13.3 |
| No sensitive targeting | No use or inference of religion, health, ethnicity or their proxies. A Track B customer is never assumed to be Muslim | Developer decision; list content **Needs legal confirmation** | DEC-23, DEC-54 |
| Rule packs blocked | Every legal rule pack stays "pending legal review" and is enforced as a block until a **named** legal approver signs | Developer decision | DEC-11, DEC-63, 12 §H.4 condition 2 |
| Sharia approval is separate | In Track B a pack, product or template needs legal **and** Sharia approval. The developer can never hold the Sharia approver role | Developer decision; process **Needs Sharia confirmation** | DEC-55, file 20 SDC-17 |
| Machine learning in shadow only | Simple rules serve results. ML runs in the background on synthetic data, never shown, until model-risk scope is settled | Developer decision | DEC-47, DEC-19 |
| Template-only output | Explanations use fixed, approved templates. No AI-written free text | Developer decision | DEC-06 |
| No external AI model in Track B | No hosted or external large language model. Only bank-hosted, in the UAE, if ever enabled by a separate decision | Developer decision; **Needs legal confirmation** | DEC-58, file 20 IQ3 |
| Human review | Track A: a person reviews offer suppression, every retention action (except "no action") and vulnerability routing | Developer decision; **Needs legal confirmation** | DEC-66, file 19 UQ4 |
| Contact limits in Track B | In-app and push messages treated as telemarketing, with contact caps, until legal answers | Developer decision; **Needs legal confirmation** | DEC-67, file 20 IQ4 |
| Demo watermark | Demo outputs say "illustrative - not legal advice - not for real customers" (Track B adds "not a Sharia ruling") | Developer decision; wording needs compliance sign-off before GATE-3A | DEC-44, DEC-55 |

## 5. Gates GATE-1, GATE-2 and GATE-3

Gate IDs come from file 11. Batches come from file 12 §H.4 and §H.7.2.

| Gate | What it covers | Items in the batch | Current status |
|---|---|---|---|
| GATE-1 Scope approval | Scope, boundaries and business definitions (churn, blue-collar, Gen Z, needs, retention outcomes, demo KPIs), ownership with the credit platform, adults only, demo posture | DEC-01, DEC-02, DEC-04, DEC-09, DEC-43, DEC-44, DEC-49; BQ-1..BQ-7, BQ-11; track scope DEC-50, DEC-59 | **Developer-ratified on delegated authority; manager countersignature recommended** |
| GATE-2 Architecture approval | Architecture and technology stack (Python/FastAPI, PostgreSQL, React, OpenAPI), deployment profiles, Track B isolation | DEC-05, DEC-08; DEC-51, DEC-53 | **Developer-ratified on delegated authority; manager countersignature recommended.** Still open: exact software version pins and the UI design system (readiness P-3) |
| GATE-3 Contract review | API contract v1, no live credit feed, approval workflow owner, contract defaults, role and permission matrix, data cells | DEC-03 / BQ-10, DEC-12, DEC-31, DEC-38, DEC-52, DEC-61, DEC-62; interim integration owner (DEC-63) | **Developer-ratified on delegated authority; manager countersignature recommended** |

**What your countersignature means.** You confirm, as an independent check, that you accept the scope, architecture and contract decisions above. This matters most for GATE-3, where the developer is both interim integration owner and ratifier.

**What it does not mean.**
- It is **not** legal approval. No rule pack becomes approved or active.
- It is **not** Sharia approval. No Track B product or template becomes Sharia-approved.
- It does **not** by itself authorise implementation. Writing code needs a separate, explicit implementation approval (section 9).

## 6. Decisions and confirmations needed from you

**[Needs manager approval]** unless shown otherwise. Fill in the last column.

| # | Item | Ref | Needed by | Name / decision / date |
|---|---|---|---|---|
| 1 | Named legal/compliance approver and backup, Track A (interim role today: Head of Compliance) | DEC-11, DEC-63 | GATE-3A | |
| 2 | Named legal/compliance approver and backup, Track B | DEC-11, DEC-63 | GATE-3A | |
| 3 | Named Sharia approver, Track B (interim role today: Sharia Compliance function / ISSC) | DEC-63, DEC-55 | GATE-4A for Track B | |
| 4 | Track A bank facts: US charter, regulator and NY status; UK authorisation | DEC-10; file 13 §6 Q1-Q3 | GATE-2A | |
| 5 | Track B bank facts: UAE licence type (full Islamic bank, Islamic window, branch, free zone), emirate, Sharia governance set-up | DEC-10; file 20 IQ1 | GATE-2A (Track B) | |
| 6 | London entity: UK-incorporated bank, UK branch of the NYC bank, or other; and the UK data controller | DEC-65; file 19 UQ1 | Before UK rule-pack content | |
| 7 | Countersign GATE-1 batch | file 12 §H.7.2 | Recommended before code | |
| 8 | Countersign GATE-2 batch | file 12 §H.7.2 | Recommended before code | |
| 9 | Countersign GATE-3 batch | file 12 §H.7.2 | Recommended before code | |
| 10 | GATE-2 remainder: software version pins and UI design system | Readiness P-3 | Before the first build story and dashboard visuals | |
| 11 | Model-risk scope and tier under the bank's policy; whether a separate AI governance track is adopted | DEC-19 | GATE-7 | |
| 12 | Whether the subsystem is in SOC 2 / ISO 27001 audit scope | DEC-07 | GATE-12 | |
| 13 | Consent purposes and grantor model **[Needs legal confirmation]** | DEC-18 | GATE-4A, before real data | |
| 14 | Data retention periods per record type **[Needs legal confirmation]** | DEC-20 | GATE-12 | |
| 15 | Fairness testing: permitted proxies, thresholds, escalation **[Needs legal confirmation]** | DEC-21 | GATE-6 | |
| 16 | Minors: NY Child Data Protection Act position, 16-17 consent **[Needs legal confirmation]** | DEC-22; file 13 §6 Q4 | GATE-6 (not needed for adults-only MVP) | |
| 17 | Exclusion, forbidden-inference and proxy lists **[Needs legal confirmation]** | DEC-23 | GATE-4A | |
| 18 | Which customer details link a customer to a jurisdiction **[Needs legal confirmation]** | DEC-25 | GATE-4A | |
| 19 | Host platform identity system and event transport (host facts) | DEC-30, DEC-33 | GATE-11 | |
| 20 | Explicit approval to start implementation (section 9) | 12 §H.4, §H.7 | Before any code | |

Items 11-19 are all the remaining BLOCKED-EXTERNAL items in file 12 §H.7 (with DEC-10, DEC-11 and DEC-65 above). The build continues safely on restrictive assumptions until each is resolved.

## 7. Legal and Sharia reviews needed

No legal or Sharia review has happened. The three matrices are technical research. "Verified" means we found the primary source text; it does not mean the law applies.

| File | What it covers | Rows | Verified / partial / unverified | Top questions | Who should review |
|---|---|---|---|---|---|
| 13 | US federal, New York State and NYC rules (Track A, `A-US`) | 42 (F01-F18, S01-S17, C01-C07) | 27 / 6 / 9 | file 13 §6 Q1 charter, Q2 NY cyber rule status, Q4 minors (NY Child Data Protection Act), Q5 credit-adjacent recommendations under fair-lending rules, Q10-Q11 model risk and AI governance | Bank's US legal, compliance, privacy and model-risk teams **[Needs legal confirmation]** |
| 19 | UK and London rules (Track A, `A-UK`) | 25 (U01-U25) | 9 / 8 / 8 | UQ1 entity type, UQ4 automated decisions and human review, UQ5 lawful basis, UQ9 financial promotions, UQ11 transfers and deployment | Bank's UK legal, compliance and data-protection teams **[Needs legal confirmation]** |
| 20 | UAE law and central-bank rules (E rows) and Sharia requirements (SH rows) (Track B) | 35 (E01-E21, SH01-SH14) | 17 / 8 / 10 | Legal: IQ1 licence, IQ2 data protection law, IQ3 hosting, IQ4 telemarketing. Sharia: IQ5 scope of Sharia approval, IQ14 prohibited categories, IQ15 approved terms, IQ16 Takaful, IQ17 late-payment text | UAE legal and compliance **[Needs legal confirmation]**; the bank's Sharia Compliance function / ISSC **[Needs Sharia confirmation]** |

All Sharia lists in the design (prohibited categories, terms, product structures) are synthetic placeholders until the bank's Sharia function supplies them (file 21 TT-R4).

## 8. Unresolved blockers

None of these blocks planning or the synthetic data phase. They block real data, rule-pack approval or later gates.

| Blocker | Ref | What it blocks | Safe assumption meanwhile |
|---|---|---|---|
| Bank charter / licence per track | DEC-10; file 20 IQ1 | Legal applicability decisions (GATE-2A), rule-pack content | No charter assumed; every row pending and enforced as block |
| No named legal approver (either track) | DEC-11, DEC-63 | Any approved or active rule pack | Role exists but is unassigned; all packs blocked |
| No named Sharia approver (Track B) | DEC-55, DEC-63 | Any Sharia-approved product, template or pack | Nothing customer-facing is Sharia-approved |
| UK entity type and data controller | DEC-65; file 19 UQ1 | UK rule-pack content | UK cell built as a separate controller |
| Model-risk scope | DEC-19 | Showing ML results | Rules serve; ML in shadow (DEC-47) |
| Consent purposes | DEC-18 | Real data | Marketing denied without explicit synthetic consent |
| Retention periods | DEC-20 | Real data | Values are configuration; none hard-coded |
| Fairness method | DEC-21 | Fairness sign-off (GATE-6) | Synthetic group labels only; minimum group 100 (DEC-48) |
| Minors legal position | DEC-22 | Any minor insight | No profiling of minors |
| Exclusion and proxy lists | DEC-23 | Real data, pack approval | Broad starter list, all excluded |
| Jurisdiction attributes | DEC-25 | Pack approval | Missing or conflicting = unknown = deny |
| Audit scope | DEC-07 | GATE-12 | Build as if in scope |
| Host identity and events | DEC-30, DEC-33 | Plugin integration (GATE-11) | Mock identity provider; no message broker |
| GATE-2 version pins and UI design system | Readiness P-3 | First build story versions, dashboard visuals | None chosen yet |
| Track B hosting beyond the bank's own systems | DEC-58; file 20 IQ3 | Any AI model or third-party hosting in Track B | Not allowed |

## 9. Proposed first implementation phase: PH-4 synthetic data generator

**[Proposed design]** (file 11 PH-4; DEC-56; file 21 §5).

**What it produces.**
- Reproducible synthetic customers and transactions, one seed per data cell (`A-US`, `A-UK`, `B-AE`). Outputs never mix across cells.
- Realistic edge cases: refunds, reversals, duplicates, pending items, transfers and multiple currencies.
- Test personas, for example a 24-year-old NYC shift worker, a London salaried renter and a Dubai or Abu Dhabi salaried customer, plus control cases in each cell: a minor, unknown age, consent withdrawn, unknown or out-of-scope jurisdiction, and a credit-request refusal.
- Three synthetic catalogues: `CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH` and `CAT-B-AE-ISLAMIC-SYNTH`. No real product names, rates, fees or profit rates.
- A data dictionary, data-quality rules and a validation report per cell (GATE-4, light sign-off).

**Why first.** Every later module (transactions, segments, needs, churn, recommendations, dashboard, demos) needs data to run and be tested. Synthetic data needs no legal sign-off on real customers. It also proves the safety paths (minors, unknown jurisdictions, credit refusal) from day one.

**What it does not do.** It uses no real data and connects to no bank system. It makes no credit decision. It does not approve or activate any rule pack. It trains no production model. It does not decide any Sharia question; Islamic items are illustrative only.

**Conditions before it may start** [Needs manager approval]:
1. An explicit implementation approval, recorded in the decision register and work log (12 §H.4, §H.7). Countersignature of GATE-1/2/3 is recommended first.
2. Only after that approval are the code-building skills re-enabled (they are switched off today in `.claude/settings.json`).
3. Synthetic data only stays in force (12 §H.4 condition 1).
4. GATE-2 version pins chosen before the first build story (readiness P-3).

## 10. Status statement

- No application code, database or infrastructure has been written.
- Implementation has not been approved.
- GATE-1, GATE-2 and GATE-3 are developer-ratified on delegated authority; manager countersignature is recommended.
- No legal review and no Sharia review has taken place. Every rule pack is `PENDING_LEGAL_REVIEW` and enforced as a block.
- No legal approver or Sharia approver has been named.

## Validation results

An independent technical check of the two-track design ran on 10 November 2026. Full detail is in `handoff/two-track-validation-report-2026-11-10.md`. **A clean technical check is not legal, Sharia or management approval.**

| Result | Detail |
|---|---|
| Overall | The design is internally consistent. Readiness verdict: **CONCERNS, no blockers**. |
| Technical checks | API contract (v1.2.0, 47 operations; revised 10 November 2026 to close V-8 and V-9): lint 0 errors, 0 warnings. Contract and event schema check: pass. 30 targeted checks of the new rules: all as expected. Role and permission matrix: pass (514 negative tests, plus 10 tests that the retired customer-list call is not served). Data model cross-check: pass (67 entities). Architecture lint: 0 findings. |
| Fixed during the check (design text only) | Stronger privacy controls on the portfolio totals view (rounding, hiding related small groups, per-user query limits and audit), so totals cannot be combined to identify a person. Tighter isolation between the US and UK data cells (logs, backups, support access, model training). Customer tokens cannot be reversed or linked across cells or tracks. Deployment profile is signed, checked at startup, and cannot be switched by configuration. A reviewer cannot release an offer the system suppressed (DEC-66). Sharia approver must be a different person from the legal approver. |
| Closed in contract 1.2.0 (design only) | **V-8:** only analysts and relationship staff can list customer tokens, and only for an approved, registered purpose, with a narrowing filter or an exact token, small page and total limits, daily limits, and an audit record for every attempt, including refusals. Service and audit accounts cannot list customers; auditors read the audit records instead. The old list call is retired. New decision **DEC-68** needs your countersignature. **V-9:** Track B suggestions now carry the Sharia disclosure references that file 20 SDC-11 requires (Sharia basis, ISSC approval, certificate, Key Facts Statement, in Arabic and English). A disclosure can show as approved only when the bank's Sharia approval is recorded; the demo always shows "illustrative - not a Sharia ruling". Track A never carries these fields. The wording and several details still need the bank's Sharia function [SHARIA-CONFIRM]. |
| Reclassified | DEC-01, DEC-02, DEC-12 and the interim-owner item of DEC-63 are working positions that need **your** countersignature; they are beyond developer authority. |
| Needs people | Licence per track (DEC-10, DEC-65); named legal and Sharia approvers (DEC-11, DEC-63); legal answers (file 19 UQ4, file 20 IQ3 and IQ4, DEC-60); Sharia lists; model-risk scope (DEC-19, machine learning stays in shadow mode). |

Recommended next step: your countersignature of this pack, now including DEC-68. Implementation (PH-4) starts only on explicit instruction.

## Change log

- 2026-11-10 (contract 1.2.0): "Validation results" updated after V-8 and V-9 were closed as design (contract 1.2.0, DEC-68); no other section changed.
