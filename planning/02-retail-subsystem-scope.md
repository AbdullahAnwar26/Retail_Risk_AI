# 02 - Retail Banking AI Intelligence: Subsystem Scope

Labels: [DOC p.N] [REC] [ASSUME] [UNKNOWN] [MANAGER] [LEGAL-CONFIRM]. NYC scope update applies (see section 8 and change log), **superseded on 2026-11-10 by the two-track scope (section 8A; file 21; file 12 DEC-50..DEC-64)**; not legal advice. Planning only. Legal/regulatory points are "to be confirmed by legal/compliance"; none are invented here.

## 1. Purpose
Analyse customer financial behaviour, identify likely financial needs, estimate disengagement (churn), and produce explainable recommendations and retention insights. [MANAGER brief] One shared intelligence core; two modes (Standalone demo; Plugin with versioned contracts). The main platform (originally described as US+UK) has no implementation, repo or stack yet [authoritative], so all host-facing parts are contracts and adapters only.

## 2. In scope
| Area | Detail | Label |
|---|---|---|
| Customer intelligence | Unified customer financial profile, income/expense/savings/spending analysis, financial health indicators | [MANAGER] |
| Transaction intelligence | Normalisation, dedup, categorisation, recurring detection, internal transfers, refunds, reversals, pending, multi-currency | [MANAGER] |
| Segmentation | Multi-membership: Gen Z 18-29, blue-collar workers, further behavioural segments; teens 16-17 flagged protected, no commercial profiling | [MANAGER]; Gen Z overlay in DOC p.28 uses 18-25 (differs) |
| Financial needs prediction | Rule-first then baseline ML (e.g. savings gap, emergency buffer; v1 taxonomy per file 12 BQ-6). Credit-building need removed from v1 (DEC-49) | [REC] |
| Churn prediction | Disengagement risk with explainable drivers | [MANAGER]; definition [UNKNOWN] |
| Product recommendation | Explainable, catalogue-grounded, eligibility-gated per resolved jurisdiction (policy packs + catalogue `jurisdictions_offered`) | [MANAGER] |
| Retention intelligence | Suggested retention actions, priority queues, outcome tracking (suggestions only; outcome values per file 12 DEC-43; stories S8.6, S19.5 in file 10) | [MANAGER] |
| Risk and compliance module | Consent checks, minors guard (two-tier block list, file 14 §13.3), fairness/proxy monitoring, audit, explanation validation, human review flagging | [MANAGER]; DOC p.38, 43, Spec s4 |
| Central AI orchestrator | LLM with controlled tools and structured outputs; explanations; never decides | [MANAGER]; DOC p.39-40 pattern |
| Standalone mode | Own API, synthetic data, dashboard, AI assistant, audit view | [MANAGER] |
| Plugin mode | Versioned contracts (REST/OpenAPI-style), adapters, integration test harness without host | [MANAGER]; mechanism [REC] decided in architecture doc |
| Jurisdiction configuration (replaces US/UK config separation) | Currency, catalogue, eligibility, terms and protection controls held as market reference data plus versioned jurisdiction policy packs; MVP packs: US federal, NY State, NYC only (file 14) | [MANAGER] NYC scope; supersedes DOC p.3, 54 framing |
| Jurisdiction Policy Layer | PDP in M8 (Risk and Compliance), PEPs in every module, versioned approved policy packs, fail-safe to restrictive default (file 14) | User; [REC] |
| Regulatory applicability assessment | Per-row applicability of file 13 items, decided by bank legal/compliance, gating the NYC policy pack | [LEGAL-CONFIRM] |
| Deployment tracks (2026-11-10) | Track A "Conventional" (US / US-NY / US-NY-NYC and UK `GB` / `GB-ENG` / London `GB-ENG-LDN`, separate data cells `A-US`, `A-UK`) and Track B "Islamic" (Islamic bank, UAE onshore [ASSUME], separate deployment); one core, one contract, track = deployment profile config (file 21) | User (DEC-50); DEC-51..DEC-53 |
| Institution-type overlay | `institution_type` `CONVENTIONAL` or `ISLAMIC`; Islamic adds a tighten-only Sharia overlay (Sharia-compliant catalogue only, terminology, prohibited categories, Sharia approval state) | DEC-54, DEC-55 [SHARIA-CONFIRM] |
| Synthetic data | Customers, accounts, transactions, goals, catalogue, features, segments, predictions, recommendations, model versions, audit, consent; per cell and per-track catalogues (`CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH`, `CAT-B-AE-ISLAMIC-SYNTH`; DEC-56) | [MANAGER] |

## 3. Out of scope
| Item | Reason / Label |
|---|---|
| Credit approval, scoring, PD/LGD/EAD, grades, limits, affordability, automation rules, decline/adverse-action reasons and letters | Credit platform; DOC p.5-21, 51 [MANAGER] |
| Underwriting queues, collections workflows, stress/climate scenarios, champion-challenger, frontier | Credit platform |
| Mortgage, SME, corporate, auto, BNPL, overdraft decisioning | Credit platform (DOC p.51, 46) |
| Sending customer communications, changing accounts, blocking/pausing products | Bank systems own these (DOC p.53, printed 21 "system never changes accounts") |
| Commercial profiling, targeting, value prediction or model training for under-18s | Protected [MANAGER]; DOC p.31, 36, 43, 44. Two-tier block list (file 14 §13.3): commercial activities and model training are locked (Tier 1); non-commercial educational insight (Tier 2) only with a signed legal/compliance approval record |
| Inferring sensitive characteristics (race, religion, health, immigration status etc.) | [MANAGER]; DOC p.28, 43. Explicitly in both tracks: religion or observance is never inferred from merchants, donations, Islamic product holdings or the institution type (DEC-54) |
| Sharia rulings (which products, terms or categories are compliant) | The bank's Sharia Compliance function / ISSC [SHARIA-CONFIRM]; we hold their decisions as data only (DEC-55) |
| Other US states, Scotland/Wales/Northern Ireland, EU, Qatar, other GCC states, UAE financial free zones | Deferred (DEC-13 superseded only for London; file 21 §2) |
| Real customer data / real bank connections | [MANAGER] synthetic only until authorised |
| Fraud, AML, KYC | Not in brief [ASSUME] other subsystems |
| Production deployment, model-risk approval | Later, bank process |
| Final regulatory interpretation | To be confirmed by legal/compliance |

## 4. Boundaries with the credit platform
| Topic | Boundary [REC] | Evidence |
|---|---|---|
| Income segments S0-S5 | Credit side owns; we **read** (via adapter) as an optional attribute, never recompute for decisions | DOC p.22, 60, Spec |
| Overlay tags | We may carry read-only copies; our own segment memberships are separate | DOC p.28, 61 |
| Early-warning signals / watch-list | Credit owns; we may consume flags to suppress offers; we do not produce credit warnings. Overlap with our churn/needs signals needs a decision (file 12 DEC-02) | DOC p.32; [MANAGER] |
| "Pause offers" | We must honour a suppression flag from credit | DOC printed 21 |
| Credit-related recommendations (cards, loans, limit increases) | We may only emit a `CREDIT_REFERRAL` ("refer to the credit platform for assessment") with no eligibility, limit, rate or likelihood; eligibility and decision come only from the credit platform; no LLM or ML approves | DOC p.4, 31; [UNKNOWN] interface |
| Use of our outputs by the host | Retail insight and marketing suggestions only; every response and event labelled `not_for_credit_decisioning`; host terms prohibit use for credit eligibility, pricing or limits [REC] [LEGAL-CONFIRM] | file 15 §1 IC7; file 12 DEC-42 |
| Readiness record (16-17) | Raw behavioural facts originate in retail data; whether we compute/store it, and how, is [MANAGER]/legal. No profiling use by us | DOC p.14, 44, 58 |
| Customer comms | We output recommendation objects; bank sends | DOC p.53 |
| Governance | Retail models follow bank model-risk process if/when confirmed (SR 26-2 is supervisory guidance; file 12 DEC-19) | DOC printed 23 [UNKNOWN] |

## 5. Shared requirements we must respect (not discarded)
1. Separation of data, config and deployment per jurisdiction/deployment scope (DOC p.3, 40, 54 originally US/UK; now NYC first, other markets later via policy packs, `CTL-DATA-RESIDENCY` where needed) [MANAGER confirm].
2. Tokenised identifiers; consent per source/purpose/expiry; withdrawal stops use; view-logging (DOC p.8, 16, 40).
3. No credit marketing/profiling of under-18s; non-credit loyalty only (DOC p.31, 36, 43-44).
4. Offers follow eligibility; affordability is assessed only by the credit platform and reaches us, if at all, as a suppression flag (DOC p.31, Spec s4). The PDF's "affordability first" rule is quoted as credit-platform context, not as a gate we apply.
5. Banned characteristics and proxy reporting (DOC p.4, 38, 43); fairness monitoring by segment (Spec s4, FR-50/51 definitions elsewhere).
6. LLM rules: no decisions, numbers verified against computation, template fallback (DOC p.39-40).
7. Immutable replayable record for decisions that affect customers (DOC p.13, 21): [REC] apply to recommendations; retention periods to be confirmed by legal/compliance.
8. Contact restrictions/vulnerability flags honoured (DOC p.41, 54).
9. Runs inside bank, offline-capable, no data leaves (DOC p.40, 49) [REC: design with no mandatory external calls; LLM local/approved].
10. Referenced IDs FR-39, FR-50/51, FR-79..86: definitions are in prd.md (not provided).

## 6. Integration dependencies
| Dependency | Direction | Status |
|---|---|---|
| Identity/authN/authZ of host | host -> us | [UNKNOWN]; adapter interface only |
| Customer/account/transaction feed (batch, API, events) | host -> us | [UNKNOWN]; DOC p.53 lists API/batch/stream/file styles used by credit |
| Consent & preference service | host -> us | [UNKNOWN] |
| Product catalogue & eligibility service (market reference data; `jurisdictions_offered` per product) | host -> us | [UNKNOWN]; synthetic in standalone |
| Segment/tag and early-warning/suppression feed from credit | credit -> us | [UNKNOWN]; candidates CI-1..CI-9 in file 15 §11; ownership file 12 DEC-02, DEC-03 |
| Retention and recommendation outcome reports | host -> us | [REC] outcome endpoints in file 15 §2 |
| Insights/recommendations/predictions API + events | us -> host | We define v1 contract |
| Audit sink / logging | us -> host | [UNKNOWN] |
| LLM runtime approved by bank | infra | [UNKNOWN]; DOC p.40 says in-bank |

## 7. Assumptions
- [ASSUME] Adults only for MVP (2026-11-10: threshold per jurisdiction as config, `US` 18, `GB` 18, `AE` [ASSUME 21] restrictive default, all [LEGAL-CONFIRM]; DEC-60); under-18 records ingested at most to be excluded or routed to protected path.
- [ASSUME] "Gen Z" for us = ~18-29 behavioural/age-based; membership not tied to credit GENZ tag.
- [ASSUME] "Blue-collar" is inferred from observable, lawful signals (e.g. shift-pattern income, occupation if consented) not income band; needs definition and legal review.
- [ASSUME] Recommendations are advisory; downstream human/bank systems act.
- [ASSUME] Stack neutral until host stack known; choose widely portable options in architecture doc.
- [ASSUME] MVP currency USD (NYC); multi-currency accounts handled in data model; GBP and others added later as market reference data [MANAGER confirm]. **2026-11-10:** currency per cell: `A-US` USD, `A-UK` GBP, `B-AE` AED; no FX conversion between cells (DEC-59).
- [ASSUME] Churn defined in discovery (e.g. salary-account inactivity) and validated on synthetic data only.
- [REC] Marketing-style use of transaction data may require specific consent/legal basis: to be confirmed by legal/compliance.

## 8. NYC geographic and regulatory scope (authoritative update)
**Initial market:** New York City, USA. Jurisdictions modelled in MVP: US federal (`US`), New York State (`US-NY`), New York City (`US-NY-NYC`) only (file 14 section 2). **No 50-state rulebooks in MVP.** Not legal advice; every legal interpretation needs confirmation by the bank's legal and compliance teams [LEGAL-CONFIRM]. No law is assumed to apply: applicability depends on the bank's charter, regulatory status, products and activities, all [UNKNOWN] (file 13 section 2.1, section 6 Q1).

| Item | MVP | Notes |
|---|---|---|
| Federal + NY State + NYC policy packs | In (illustrative, pending legal approval) | Drafted from file 13 rows; status `PENDING_LEGAL_REVIEW` until approved; restrictive default meanwhile |
| Regulatory applicability assessment of file 13 rows | In (gate for the NYC pack) | Set by legal approver only [LEGAL-CONFIRM] |
| Jurisdiction Policy Layer (versioned, effective-dated, maker-checker) | In | Core is jurisdiction-neutral; no `if state == "NY"` in core |
| Other US states (e.g. `US-CA`, `US-NJ`) | Out (architecture ready) | Added later as packs, no core change (file 14 section 12) |
| UK and other international markets | Out; future expansion [MANAGER confirm]. **Superseded 2026-11-10:** UK (London) is in Track A and UAE (Islamic bank) is Track B (section 8A); other international markets stay out | Supersedes earlier "US and UK separate installs" framing (file 12 DEC-08, formerly D8, superseded; DEC-13) |
| Customers outside deployment scope / unknown jurisdiction | Restrictive default | `UNSUPPORTED_JURISDICTION` / `UNKNOWN_JURISDICTION`; never borrow NYC rules (file 14 section 10) |
| Credit decisions, regulated decisions | Out, unchanged | AI never approves/rejects credit; credit shown only as referral (`CTL-CREDIT-REFERRAL-ONLY`) |
| Sensitive characteristics / unsupported inference for commercial targeting | Out, unchanged | `CTL-SENSITIVE-EXCLUDE`, `CTL-NO-INFERENCE` |
| Minors | Protected journey kept separate; two-tier block list (file 14 §13.3): commercial profiling, marketing and model training always blocked; non-commercial educational insight only with a signed approval record | `CTL-MINOR-PROTECT`; file 13 rows S12, S13, F12; file 13 §6 Q4, Q14; file 12 DEC-22 |

## 8A. Two-track scope update (2026-11-10, authoritative; supersedes the NYC-only framing above)
Developer decision (file 12 DEC-50): *"NYC and London in one, and on the other side the Islamic bank."* Design: file 21; decisions DEC-50..DEC-64 (file 12 §H.7). Section 8 stays valid for the `A-US` cell.

| Item | Track A "Conventional" | Track B "Islamic" |
|---|---|---|
| Jurisdictions | `US` > `US-NY` > `US-NY-NYC`; `GB` > `GB-ENG` > `GB-ENG-LDN` [ASSUME local code] | `AE` > `AE-<emirate>` [ASSUME UAE onshore; emirate unknown; free zones out] |
| Institution type | `CONVENTIONAL` | `ISLAMIC` + Sharia overlay `PACK-INST-ISLAMIC` (tighten-only; DEC-54) |
| Data | Two cells `A-US`, `A-UK`, separate database each, in-region; aggregate-only across cells (DEC-52) | Separate deployment, bank on-prem, never mixed with Track A (DEC-53) |
| Legal content | US: file 13; UK: file 19 (written 2026-11-10); all PENDING_LEGAL_REVIEW = BLOCK [LEGAL-CONFIRM] | File 20 (written 2026-11-10) [LEGAL-CONFIRM] [SHARIA-CONFIRM] |
| Catalogue | Synthetic US and UK conventional | Synthetic Islamic (financing only as `CREDIT_REFERRAL`; DEC-56) |
| Languages / currency | en-US / USD; en-GB / GBP | ar + en (RTL) / AED (DEC-59) |
| Unchanged in both | Minors two-tier list (Tier 1 never relaxed), no credit decision (credit only as `CREDIT_REFERRAL`, `not_for_credit_decisioning`), synthetic data only, template-only explanations (DEC-06) | same |

Future expansion: add jurisdiction records, an applicability matrix for the new area, a policy pack, golden cases and approval, using existing controls (file 14 section 12).

## Change log (NYC scope update)
- Added section 8 (NYC geographic and regulatory scope, MVP in/out, future expansion).
- Replaced "US/UK config separation" in-scope row with jurisdiction configuration; added Policy Layer and applicability assessment rows.
- Reworded shared requirement 1 and currency assumption; UK/other markets now future expansion [MANAGER confirm].

## Change log (validation pass 2026-11-09)
- D-01: §2 risk module, §3 under-18 row and §8 minors row use the two-tier block list (file 14 §13.3).
- D-02: §5 item 4 rewritten; affordability is the credit platform's (suppression flag at most); the PDF rule is kept only as quoted credit context.
- D-10: §4 "eligible-to-be-assessed candidates" replaced by `CREDIT_REFERRAL` with no eligibility, limit, rate or likelihood.
- D-11: §4 new row on host use of outputs (`not_for_credit_decisioning`, DEC-42) [REC].
- D-09 / G-9: §2 retention row and §6 dependency reference outcome tracking (DEC-43; stories S8.6, S19.5).
- D-03 / D-06 / R-06: references to decisions now DEC-xx (file 12); file 13 rows and §6 questions written in full; credit candidates cited as CI-1..CI-9 (file 15 §11).
- File 13 re-verification: §4 governance row notes SR 26-2 is supervisory guidance (DEC-19).
- Sweep 02:16 (per market -> per resolved jurisdiction) and 02:71 (per market -> market reference data with `jurisdictions_offered`) corrected; 02:6/22/54/83/96/106-107 kept (historical / future expansion).

## Change log (readiness fixes 2026-11-09)
- §2 needs row: "credit-building need as an insight only" removed from v1 (DEC-49); taxonomy per file 12 BQ-6. Target market NYC vs UAE vs both is open (DEC-50); NYC content unchanged.

## Change log (two-track scope 2026-11-10)
- Header, §2 (deployment tracks, institution-type overlay, per-cell synthetic data), §3 (religion inference ban in both tracks, Sharia rulings out of scope, deferred markets), §7 (age threshold per jurisdiction DEC-60, currency per cell DEC-59), §8 UK row superseded, new §8A two-track table. Source: file 21, file 12 DEC-50..DEC-64. DEC-13 superseded for London.
- Files 19/20 recorded as written 2026-11-10 (§8A legal-content row).
