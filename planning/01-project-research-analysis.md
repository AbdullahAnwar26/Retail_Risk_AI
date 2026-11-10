# 01 - Project Research Analysis

Status: DRAFT for approval. Planning only; no code. Source: manager's research pack "AI Risk Score Card Global" (81 PDF pages, Nov 2026), analysed via text extraction notes.

> NYC scope update: US/UK constraints below are PDF context; for this subsystem they are superseded by file 02 §8 and file 14 §0 (initial market New York City; jurisdiction policy packs; UK and other states future expansion).

Label legend: [DOC p.N] stated in PDF (PDF page index) | [REC] my recommendation | [ASSUME] assumption to confirm | [UNKNOWN] not in evidence | [MANAGER] needs manager/team answer | [LEGAL-CONFIRM] legal/compliance must confirm. Decisions arising from this analysis are tracked in file 12 (DEC-xx).

## 1. Headline finding
- [DOC p.1, p.51] The PDF is a **credit-risk scorecard product** (cards, loans, salary advance, mortgages, commercial lending; US+UK). Its scope table lists lending only. **Retail customer intelligence, churn, retention, transaction analysis and product recommendation are not defined as features anywhere in the pack.** (All four note files agree.)
- [DOC p.46, p.22(printed)] Only near-neighbours: "offers", "next-best offers", "Gen Z readiness and loyalty", "inclusion engine", "product designer" in platform Release 3; "collections help" in Release 4. Boundary vs. our subsystem [UNKNOWN] -> [MANAGER].
- [REC] Treat the PDF as (a) context for the overall platform, (b) a source of **shared constraints** our subsystem must respect, (c) a list of **candidate integration points**. Do not treat credit requirements as ours.

## 2. Classification

### 2.1 Overall platform (context; not ours to build)
| Item | Ref | Note |
|---|---|---|
| Positioning: runs inside bank, beside existing decision engine; "AI recommends, people accountable" | DOC p.2, 49-51 | Principles apply by analogy |
| Separate US and UK installations; no data crosses | DOC p.3, 16, 40, 49, 54 | Strong constraint in PDF; superseded for this subsystem by NYC scope (file 02 §5 item 1, file 14 §0) |
| 3 deployment modes: Challenger / Add-on / Full engine | DOC p.4, 53 | Mirrors our Standalone/Plugin idea [REC: reuse vocabulary] |
| Integration styles: real-time scoring API (<300 ms), nightly batch (50M+ accounts), event streaming, secure file drop; bank systems stay system of record and send ALL customer comms | DOC p.53 | Only integration points defined; none name retail intelligence |
| Model governance: model-risk approval, drift, auto switch-off/fallback, SR 26-2 (US), PRA SS1/23 (UK) doc packs | DOC p.23(printed 23)/pdf~71, p.32, 38 | Whether it extends to retail models [UNKNOWN] |
| Security: tokenisation, encryption, per-source consent, view-logging, offline capable, SOC 2/ISO 27001 | DOC p.8, 16, 40, pdf~72 (printed 26) | Reusable security baseline |
| LLM rules: runs in bank, never decides, never picks reasons, numbers verified, template fallback | DOC p.39-40, printed 25 | Directly reusable for our orchestrator |
| Release plan R1-R4, ~3 months, 14-18 people, "high-risk" | DOC p.45-46, printed 22 | Delivery context only |
| Pilot path: back-test 3-5 yrs, 6-month shadow, limited live; vendor approval 12-24 months | DOC printed 22 | [REC] no promise of production dates |
| Roles/RACI (no retail/marketing role listed) | DOC printed 24 | RACI table garbled |

### 2.2 Credit-risk-only (out of scope for us)
| Item | Ref |
|---|---|
| PD/LGD/EAD/expected loss; 20-grade master scale | DOC p.10-11, 22, 59 |
| Affordability (UK CONC 5.2A) / ability-to-pay (US CARD Act) | DOC p.10, 14, 18 |
| Limit = lowest of caps; automation rule (starter <=1,000 at grade <=12; never auto-decline) | DOC p.11-12, 19-20, 27 |
| Reason codes (<=4), adverse-action/counteroffer letters, UK notices | DOC p.11, 13, 19, 21 |
| Segment policy matrix (products, limits, DTI, residual minimums, LTI) | DOC p.27-28, printed 15, Spec s2 |
| Frontier / opportunity-loss, champion vs challenger, stress scenarios, climate | DOC p.33-37, 62, printed 17-20 |
| Maria / Tom 15-step flows and worked examples | DOC p.5-21, 57-58 |
| Regulatory packs (Reg B, FCRA, MLA, CONC, MCOB, UK GDPR 22A-D) | DOC p.54 |

### 2.3 Retail-subsystem (candidate ownership; our area)
| Item | Ref | Comment |
|---|---|---|
| Transaction behaviour (income steadiness, rent, spend vs income, 12-month cash flow) | DOC p.5-6, 8, 16, 22 | Used by credit today; we analyse it for retail purposes |
| Early-warning signal families (income shock, credit stress, spending warning, external) | DOC p.32, 37, printed 20 | Overlaps our churn/needs signals; owner [UNKNOWN] |
| Customer segments & overlays (GENZ 18-25, THIN_FILE, GIG_SELF_EMPLOYED, RETIREE, NEW_TO_COUNTRY, UNDER_18_READINESS) | DOC p.28, 61, Spec s1 | Income-based S0-S5 defined by credit side; ours are behavioural/life-stage and multi-membership [REC] |
| Gen Z / blue-collar products and loyalty (starter card, salary advance, credit builder, savings boosts, rewards, student discounts, graduation path) | DOC p.35-36, printed 19 | Product catalogue inputs for recommendation |
| Readiness record (16-17: saving, no overdraft, money lessons) | DOC p.14, 36, 44, 58 | Data origin retail; consumption credit; governance [UNKNOWN] |
| "Pause offers", "offer help before missed payment", collections contact limits & vulnerability flags | DOC p.32, 41, printed 21 | Constraints on our outreach |
| Next-best offers | DOC p.46, printed 22 | Boundary [MANAGER] |
| Churn prediction, retention, financial-needs, portfolio analytics | - | **Not in PDF** [UNKNOWN]; defined by user brief only |

### 2.4 Shared requirements and integration points
| Ref/ID | Requirement | Source | Our treatment |
|---|---|---|---|
| (no ID) | US/UK separation of data, config, installation | DOC p.3, 40, 54 | Superseded by NYC scope: deployment scoping + jurisdiction packs (file 02 §5 item 1; file 14 §0) |
| (no ID) | Tokenised identifiers; per-source consent with purpose/expiry/withdrawal; every personal-data view logged | DOC p.8, 16, 40 | Binding; withdrawal must stop use |
| (no ID) | No credit marketing / no marketing profiling of under-18s; non-credit loyalty only; first credit offer at 18 after full check | DOC p.31, 36, 43, 44, printed 16/19, Spec s4 | Binding; aligns with minors safeguard (two-tier block list, file 14 §13.3) |
| (no ID) | Offers follow segment eligibility, affordability first | DOC p.31, Spec s4 | Credit-platform context, not our gate: affordability is assessed only by the credit platform; we honour only its suppression flag ("pause offers", file 15 §11 CI-2); segment eligibility for our offers comes from our catalogue and policy gates (file 02 §5 item 4) |
| (no ID) | Bank systems send all customer communications | DOC p.53 | We emit recommendations/insights, never send |
| (no ID) | Banned attributes and proxies (postcode); report suspected proxies to model risk | DOC p.4, 38, 43 | Fairness requirement |
| (no ID) | UK Consumer Duty outcomes by group, vulnerable customers; Colorado/California AI rules 2027 | DOC p.4, 54 | Applicability to retail [to be confirmed by legal/compliance] - UK and other states are future expansion (file 08 §3.4) |
| (no ID) | Immutable, replayable record; retention (US >=25 months; UK card life +6 yrs) | DOC p.13, 21 | Credit-specific; retail retention [UNKNOWN] |
| (no ID) | Segment re-check nightly; up after 2 months, down after 3 | DOC p.22, 31, Spec s3 | Interface candidate: read segment/tags |
| **FR-39** | Under-18 readiness record, no credit | DOC Spec (cited only, p.28 tag table) | **Definition lives elsewhere (prd.md)** |
| **FR-50 / FR-51** | Fairness suites (approval rate, limits, APR, decline reasons by segment x protected proxy) | DOC Spec s4 | **Definition elsewhere** |
| **FR-79 to FR-86** | Income segmentation requirements | DOC Spec s6 "see prd.md s4.15" | **Text absent from PDF; obtain prd.md s4.15** |
| (none) | NFR- IDs | - | No NFR IDs anywhere in the PDF |

Candidate integration points (all [UNKNOWN] as contracts): read income segment + overlay tags; read daily early-warning flags / watch-list; provide "pause offers" suppression honoured by us; supply readiness-record inputs (retail -> credit); consume bank data platform / event stream / nightly batch; shared SSO/consent service.

## 3. Missing information and unanswered questions
| # | Gap | Label |
|---|---|---|
| 1 | Main platform stack, repo, auth, API style, DB, messaging | [UNKNOWN] (no implementation exists - authoritative) |
| 2 | prd.md (FR-39, FR-50/51, FR-79..86 text) | [MANAGER] obtain |
| 3 | Ownership of next-best offers, Gen Z loyalty, early-warning "pause offers" (platform R3 vs our subsystem) | [MANAGER] |
| 4 | Whether we consume/duplicate credit early-warning signals | [MANAGER] |
| 5 | Interface for segment/tag read, readiness-record write | [UNKNOWN] |
| 6 | Presentation referenced in brief (manager deck) not in notes | [UNKNOWN] |
| 7 | Teen journey: any permitted retail analytics (e.g. savings nudges) | [to be confirmed by legal/compliance] |
| 8 | Retail data retention, consent purposes for marketing/analytics use of transaction data | [to be confirmed by legal/compliance] |
| 9 | Product catalogues, terms, eligibility per jurisdiction (MVP NYC) | [UNKNOWN] - synthetic only |
| 10 | Does governance (SR 26-2 / SS1/23) cover retail churn/recommendation models | [UNKNOWN]; SR 26-2 is supervisory guidance and leaves GenAI to the bank's own practices (file 13 row F14; file 12 DEC-19) |
| 11 | Churn definition, retention actions, KPIs | [UNKNOWN] [REC: define in discovery] (file 12 DEC-04, BQ-1) |
| 12 | "Blue-collar" definition: PDF only uses S2 label "Core/blue-collar" (income band) | [UNKNOWN] (file 12 DEC-04, BQ-2) |

## 4. Extraction caveats
- Text extracted with pdftotext; charts/graphics read as text only. Pound sign garbled (GBP inferred).
- Garbled tables (do not rely on cell values): segment policy matrix (p.27-28, printed 15), RACI (printed 24), glossary alignment, regulatory pack table (p.54), tag/trigger/effect columns (p.61).
- Ambiguous charts: p.26, 35, 37, 62, printed 17 (value-to-label pairing). Grade-to-month rows scrambled in Maria nightly example.
- Comparator symbols (>=, <) and arrows lost; "$/GBP" ambiguous; UK segment thresholds on p.22 not visible there (found at p.60).
- Page references are PDF page index; printed numbers differ for pp.49-77 (printed = PDF - 48).
- All figures are marked illustrative/simulated; "not legal advice" (p.48).

## 5. Inconsistencies found
| # | Inconsistency | Detail | Impact on us |
|---|---|---|---|
| 1 | 12 vs 15 steps | Contents/leadership plan (p.50, 55) describe 12 steps; Maria/Tom walkthroughs (p.8-21) use 15 | None directly; do not cite step numbers |
| 2 | Minimum payment $150 vs $45 | $150 = at the $5,000 request (p.6, 10, 39); ~$45 = at the $1,500 limit (p.29, 57; residual $560-$45=$515). Likely both valid for different amounts, but p.39 letter sample quotes $150 beside the $1,500 limit | Low; shows letter numbers must be tied to final limit |
| 3 | GBP750 vs GBP1,000 | Tom EAD ~75% of GBP1,000 = GBP750 (p.18); Q5 (p.44) uses 750 without explanation | None (credit); illustrates need to state formulas |
| 4 | PDF page order repeats | pp.43-48 are end-of-document (sec 17-19, glossary); pp.49-62 restart at cover (printed 1-14); pack contains the leadership plan, with part of Part 1 duplicated in summary form | Page refs may be ambiguous; always cite PDF index |
| 5 | Gen Z age band | Tag GENZ = 18-25 (p.28, 61); user brief says Gen Z ~18-29 | [REC] our segment is behavioural, 18-29, distinct from the credit tag; confirm (file 12 DEC-04, BQ-3) |
| 6 | Starter card limits | p.27 vs p.15/Spec garbled for S1/S2 US ranges | None |
| 7 | Release timing | "2 to 3 months" delivery vs 16-week Gantt vs 12-24 month vendor approval | Do not inherit timeline |
| 8 | Contents says Part 3 pp.78-81 "S0-S5"; p.27 matrix shows only S0-S3 | Extraction truncation likely | None |
| 9 | Notice timing: 30 days vs 90 days (counteroffer) | Trigger wording ambiguous (p.13, 54) | None (credit) |

## Change log (validation pass 2026-11-09)
- D-41: NYC scope banner and [LEGAL-CONFIRM] legend added.
- D-02: §2.4 "affordability first" row re-labelled as credit-platform context, not our gate.
- Sweep 01:18 and 01:55 (US/UK separation "strong constraint" / "Binding") marked superseded by NYC scope; 01:61 notes UK and other states as future expansion; 01:82 "per market" -> "per jurisdiction (MVP NYC)". Historical PDF items (01:8, 21, 32-38, 60, 62, 83, 88-100) kept.
- Gap questions 10-12 cross-referenced to file 12 DEC-04 / DEC-19 and business questions BQ-1, BQ-2; SR 26-2 noted as supervisory guidance (file 13 re-verification).
