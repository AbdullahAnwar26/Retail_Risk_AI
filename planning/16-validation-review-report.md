# 16 - Validation Review Report (planning pack files 00-15)

Status: READ-ONLY review, 2026-11-09. No existing file was edited. Scope: files `00` to `15` in `G:\Retail_Risk_AI\planning\`, every file read in full (not only change logs). A file `17-bmad-installation-recommendation.md` appeared in the folder during the review; it is outside this review's scope and is not in file 00's index.
Line numbers refer to the files as they stood at review time. "Fix" text is written so that another agent can apply it mechanically; where a fix touches several files, each file/line is listed.

## 0. Summary

| Severity | Count |
|---|---|
| High | 2 |
| Medium | 15 |
| Low | 27 |
| **Total** | **44** |

Top 5 issues:
1. **D-01 (High)** - Minors' protected journey is defined four different ways (14 §13.3 / 07 §8.1 vs 08 CMP-01 + NYC-03 vs 05 l.161 vs 10 S6.3). `NEEDS_PREDICTION`, `PROFILE_BUILD` and `MODEL_TRAINING` are not blocked for minors by the control that is supposed to block them.
2. **D-02 (High)** - "Affordability" leaks into our gates (10 S8.2 "affordability-flag input", 08 CMP-05, 02 §5 item 4, 13 F05 controls). This contradicts 09 "Won't: affordability" and 02 §3. No data field, contract or owner exists for it.
3. **D-03 (Medium)** - Decision IDs collide. File 08 §4 D1-D11 and file 12 §A D1-D9 are different decisions under the same IDs. File 12 says it is the consolidated register, but it omits 08 D2-D10, 14 J2-J9 and 15 I1-I6.
4. **D-04 (Medium)** - The credit-integration candidates O1-O9 are numbered differently in 07 §9 and 15 §11, so the same ID means different things. 15 also drops the readiness record and adds the referral handoff.
5. **D-05 (Medium)** - The forbidden-term CI linter (14 §13.1, 15 §3.3) would fail on legitimate contract terms that are already designed: recommendation status `DECLINED`, `approval_status: APPROVED` in policy-pack reads, and "Model approved" events.

## 1. Cross-document consistency

### 1.1 Verified consistent (no action)
- **Module numbering:** M1 = Customer Intelligence and M2 = Transaction Intelligence in 05 §2, 06, 09, 10 and 14 §6. The build order M2 -> M1 -> M3 -> M4/M5 -> M6 -> M7 is the same in 05 §5, 06 §4 and 11 §2. M8 hosts the PDP in 02, 05, 06, 09, 10 S14.3 and 14 §6.
- **Control IDs:** all 23 `CTL-*` IDs referenced anywhere in 00-15 are defined in 14 §5, and every defined control is used. All four `EXCL-*` lists are defined in 14 §13.2.
- **Requirement IDs:** SEC-01..18, PRV-01..13, CMP-01..19 and NYC-01..11 are all defined in 08, and no reference points to a missing ID.
- **File 13 rows:** every F/S/C row referenced (F01-F18, S01-S17, C01-C07) exists. The tally in 13 §3.4 (42 rows: 25 verified, 8 partial, 9 unverified) recounts correctly and matches 00 §5 and 08 §3.
- **Section references:** every "file 14 §x", "file 13 §x" and "file 15 §x" reference resolves to a real heading. The two exceptions are D-14 (14 internal "§9") and D-27 ("file 14 s12" formatting).
- **Epic-to-phase map:** 10 l.5 (E13=P2A, E14=P3A, E15=P3A/P4A, E16=P4A, E17=P8A, E18=P6/P8A, E19=P3/P11) matches 11 §7. The gates G2A, G3A, G4A and G8A exist in 11. The legal question cross-reference in 12 §G2 (Q1-Q24) matches 13 §6.
- **Event names:** the names in 03 §3.8, 04 §5/§6 and 14 §7/§10 match the CloudEvents types in 15 §9. The policy entities in 07 §2.2 match 14 §3.2, apart from the minor type differences in D-21.
- **Labels:** [DOC]/[REC]/[ASSUME]/[UNKNOWN]/[MANAGER]/[LEGAL-CONFIRM] are used consistently. 01, 09, 10 and 12 omit [LEGAL-CONFIRM] from their legend line but 09/10/12 use it in the body (Low, D-41).

### 1.2 Inconsistencies found
These are listed in §6 as defects D-01 to D-44. The main clusters are:
- **Minors' block list** (D-01).
- **MoSCoW vs epic priority:** 09 rates the policy simulator, the pack maker-checker and the model registry as Should, while 10 rates them as Must (D-07, D-08).
- **ID-scheme collisions across files** (D-06).
- **Retention outcome tracking:** in scope in 02/06 but absent from 07 and 15 (D-09).
- **Endpoint naming:** 06 M1 still uses `GET /customers/{id}/profile` (D-22).
- **Enum and type drift:** channel `APP` vs `MOBILE_APP`, `decision_id` uuid vs string, and `protection_status` missing `AGE_UNKNOWN` (D-19, D-20, D-21).
- **Stale status lines:** file 00 still shows files 13-15 as pending (D-12). The 05 change log contradicts its §2 (D-13).

## 2. Scope (Retail Banking AI only; credit decisioning out)

**Every module belongs to Retail Banking AI.** M1-M7 are retail intelligence, M8 is a compliance guard (it is not credit risk despite the name "Risk & Compliance"), and M9 is the orchestrator. The PDP/policy layer is cross-cutting infrastructure inside the retail core.

**Legitimate boundary items (keep):**
- Read-only consumption of S0-S5 income segments and tags (07 O1, 15 O1).
- "Pause offers" / early-warning flags used only to suppress offers (O2).
- `CREDIT_REFERRAL` as the only credit-related output (14 §13.1, 15 §3.3).
- Locked `CTL-CREDIT-REFERRAL-ONLY` and `CTL-ADVERSE-ACTION-HANDOFF` (refusal and handoff only; nothing is generated).
- PD/grade "not accepted in v1" (07 O9, 15 O8).
- Credit-building need as an insight only (06 M4).
- The under-21 CARD Act wording flag, which is a label and not an assessment (08 NYC-04).
- The readiness record not built by us (07 O6).

**Leaks or near-leaks (fix):**

| Item | Where | Assessment |
|---|---|---|
| Affordability gating | 10 l.53 S8.2 "affordability-flag input"; 08 l.83 CMP-05 "affordability first"; 02 l.57 §5 item 4; 13 l.91 F05 controls "Affordability-first and suitability rules"; 08 l.19 (PDF context, OK) | **Leak (D-02).** It implies we gate on affordability, yet 09 l.34 Won't, 02 l.30 Out of scope, and no field, contract or credit-signal candidate exists. |
| "Eligible-to-be-assessed candidates" | 02 l.48; 13 l.219 (§5.4) | **Near-leak (D-10).** It implies we determine eligibility. 15 §3.3 says a referral "carries no eligibility". 13 Q-P2 itself flags this as "close to that line". |
| No use restriction on outputs | 15 §1 C2, §12; 13 F04/F06 note the risk that outputs feed credit | **Gap (D-11).** M1 profile features (income stability, spend vs income) are exactly the affordability inputs credit uses (01 l.43). Nothing stops the host using them for credit eligibility, which creates FCRA "consumer report" and Reg B risk. |
| Reason-code rationale "if an output ever feeds a credit process" | 08 l.89 CMP-11; 13 l.92 F06 controls | **Wording (D-36).** It normalises a future credit use. |
| PD/grade "not to be used for marketing without legal approval" | 07 l.298 O9; 12 l.11 D3 | **Wording (D-37).** It implies future marketing use of a credit-risk score is a legal-only call. It also needs manager approval and fairness review. |

Nothing in the data model (07), contracts (15), MVP (09) or epics (10) contains scoring, grading, limit-setting, PD or adverse-action generation. The 14 §13.1 and 15 §3.3 denylists hold, subject to D-05.

## 3. Core sharing (standalone vs plugin)

**Verified consistent in every document.** One core image and package, two adapter wirings:
- 03 §1, §6 items 1/6
- 04 §1, §5.1 table "identical code, same image", guarantees (1)-(4), import-linter, no `if MODE ==`
- 05 §4/§6/§7 "PDP - same code as standalone"
- 06 §3 "Shared package, identical"
- 09 §4 "Plugin proof ... results identical"
- 10 S2.3, S11.2, S14.7, S19.3 parity
- 11 P11 exit "Parity with standalone"
- 14 G5, §11
- 15 C1, §13 equivalence test

Mode-specific items are confined to adapters or process: the host overlay (merge code is in `core/policy`, fed only in plugin), the dashboard and simulator UI (a standalone API client), and approval roles (process).

Two Low clarifications:
- 04 l.188 "Shadow mode flag" does not say where it lives. It must be in the inbound adapter or response layer, not in `core/` (D-38).
- 14 §11 "Who approves" differs by mode. That is a process difference, not a logic difference, and is acceptable.

## 4. NYC-first consistency and US/UK/market sweep

NYC-first is stated consistently: federal + NY State + NYC; `deployment_scope=[US-NY-NYC]`; no 50-state rulebooks; future jurisdictions as packs. It appears in the 02 §8, 03-08 banners, 09 §7, 10 l.5, 11 §7, 12 §G, 13 header and 14 §0/§2. UK, other states and other markets are "future [MANAGER confirm]" everywhere (00, 02, 03 J1, 04, 05, 07, 08 D11, 12 N4, 14 J1).

The sweep used the terms `UK`, `US/UK`, `US+UK`, `GBP`, `$`, `Consumer Duty`, `market`, `markets`, `market_code`, `per market`, `install(s)`, `PRA`, `SS1/23`, `CONC`, `NI numbers`, `postcode`, `GB`, `PACK-GB` and `currency`/`USD`. File 13 only matched `$10bn`/`$30bn`/`$20m` regulatory thresholds (intentionally retained). `currency`/`USD`/`multi-currency` hits (02 l.12/83, 03, 07 l.12/191, 15) are all correct ISO-4217 or USD-reporting design and are not listed individually.

| File | Line | Excerpt | Classification | Proposed correction |
|---|---|---|---|---|
| 00 | 56 | "Remaining mentions of US/UK ... full read-through for stragglers is still to do" | Accidental leftover (status) | Replace "a full read-through for stragglers is still to do" with "read-through done; see file 16". |
| 00 | 65 | "replaces the earlier 'separate US and UK installs' framing" | Historical information | Keep. |
| 00 | 72 | "$150 vs $45, GBP750 vs GBP1,000" | Historical information (PDF) | Keep. |
| 00 | 84 | "Files 01-12 drafted (global US/UK framing)" | Historical information | Keep. |
| 01 | 8 | "commercial lending; US+UK" | Historical information (PDF) | Keep. |
| 01 | 18 | "Separate US and UK installations ... Strong constraint on us" | Incorrect or outdated requirement | Change the Note cell to "Strong constraint in PDF; superseded for this subsystem by NYC scope (file 02 §5 item 1, file 14 §0)". |
| 01 | 21 | "SR 26-2 (US), PRA SS1/23 (UK)" | Historical information (PDF) | Keep. |
| 01 | 32, 34, 38 | "UK CONC 5.2A", "UK notices", "CONC, MCOB, UK GDPR" | Historical information (credit, out of scope) | Keep. |
| 01 | 55 | "US/UK separation of data, config, installation ... Binding [REC]" | Incorrect or outdated requirement | Change "Binding [REC]" to "Superseded by NYC scope: deployment scoping + jurisdiction packs (file 02 §5 item 1; file 14 §0)". |
| 01 | 60 | "Banned attributes and proxies (postcode)" | Intentionally retained context | Keep (PDF wording). |
| 01 | 61 | "UK Consumer Duty outcomes by group ... Colorado/California" | Future expansion | Append "- UK and other states are future expansion (file 08 §3.4)". |
| 01 | 62 | "retention (US >=25 months; UK card life +6 yrs)" | Historical information (credit) | Keep. |
| 01 | 82 | "Product catalogues, terms, eligibility per market" | Historical information | Optional: change "per market" to "per jurisdiction (MVP NYC)". |
| 01 | 83 | "SR 26-2 / SS1/23" | Historical information | Keep. |
| 01 | 88, 91, 100 | "GBP inferred", "$/GBP ambiguous", "GBP750 vs GBP1,000" | Historical information (PDF) | Keep. |
| 02 | 6 | "(originally described as US+UK)" | Historical information | Keep. |
| 02 | 16 | "eligibility-gated per market" | Accidental leftover requiring correction | Change to "eligibility-gated per resolved jurisdiction (policy packs + catalogue `jurisdictions_offered`)". |
| 02 | 22, 54, 96, 106-107 | "replaces US/UK config separation", "originally US/UK", "UK and other international markets ... future" | Historical information / Future expansion | Keep. |
| 02 | 71 | "Product catalogue & eligibility service per market" | Accidental leftover requiring correction | Change "per market" to "(market reference data; `jurisdictions_offered` per product)". |
| 02 | 83 | "MVP currency USD (NYC) ... GBP and others added later" | Future expansion | Keep. |
| 03 | 8, 19, 81, 142, 264, 271-273 | "supersedes ... US and UK installs", "rather than a `market_code`", "replacing the old `market` claim" | Historical information | Keep. |
| 03 | 93, 192, 223 | "market reference data, MVP: US" | Intentionally retained context (defined concept) | Keep. |
| 03 | 209 | "`deployment_scope=[GB]`" | Future expansion | Keep. |
| 04 | 5, 203-206 | "one image per market instance (US, UK)" | Historical information | Keep. |
| 04 | 62, 155, 167 | "`config/reference/US/` market reference data" | Intentionally retained context | Keep. |
| 04 | 68 | "future: PACK-US-<state>, PACK-GB ..." | Future expansion | Keep (also change "s12" to "§12", D-27). |
| 05 | 6, 38, 103, 180, 229, 296-299 | "GB", "PACK-GB", "future international market", "US/UK instances ... removed" | Future expansion / Historical information | Keep. |
| 06 | 15 | "apply eligibility, market, teen and suppression gates" | Accidental leftover requiring correction | Change "market" to "jurisdiction (PDP + `jurisdictions_offered`)". |
| 06 | 17 | "policy config per market" | Accidental leftover requiring correction | Change to "jurisdiction policy packs + market reference data". |
| 06 | 63 | "eligibility and market filter from catalogue" | Accidental leftover requiring correction | Change "market filter" to "`jurisdictions_offered` filter". |
| 06 | 65, 76, 95, 112 | "catalogue is market reference data", "NYC-market catalogue (other markets later)" | Intentionally retained context / Future expansion | Keep. |
| 07 | 6, 201, 303 | "one database per market (US/UK)" replaced | Historical information | Keep. |
| 07 | 53, 205 | "Future `US-CA`, `GB`", "future international market" | Future expansion | Keep. |
| 07 | 101 | `customer_token UK` | Not a market reference (ER "unique key") | Keep. |
| 07 | 261 | "2M US / 1M UK synthetic customers for credit" | Historical information (PDF) | Keep. |
| 07 | 270 | "not by per-market config" | Historical information | Keep. |
| 08 | 15 | "Separate US and UK installations ... Superseded" | Historical information | Keep. |
| 08 | 16 | "Names / SSN / NI numbers tokenised" | Historical information (PDF) | Keep. |
| 08 | 23 | "Income bands / postcode can be proxies" | Intentionally retained context (PDF) | Keep. |
| 08 | 26 | "Consumer Duty ... UK regime - future market only" | Future expansion | Keep. |
| 08 | 40 | "replaces the old `market` claim" | Historical information | Keep. |
| 08 | 87 | "known proxies (postcode/ZIP, ...)" | Intentionally retained context | Optional: "ZIP code (PDF: postcode)". |
| 08 | 95 | "market reference data (currency, catalogue, terms refs)" | Intentionally retained context | Keep. |
| 08 | 183, 200, 204-208 | "UK regimes named in the PDF ... Future expansion" | Future expansion | Keep. |
| 09 | 3, 38, 48, 87, 99-100 | "Other states and UK are deferred" | Future expansion | Keep. |
| 09 | 15 | "eligibility/market/teen/suppression gates" | Accidental leftover requiring correction | Change "market" to "jurisdiction". |
| 09 | 46 | "SR 26-2 / SS1/23 style" | Historical information | Optional: drop "/ SS1/23" (UK, future). |
| 09 | 47, 74 | "NYC-market product terms / catalogue" | Intentionally retained context | Keep. |
| 09 | 77 | "replaces the US/UK switch" | Historical information | Keep. |
| 10 | 5, 140-141, 149 | "other states and UK are future", "changed from US/UK" | Future expansion / Historical information | Keep. |
| 10 | 22 | "with money types and market partitioning" (S3.1 story text) | Accidental leftover requiring correction | Change "market partitioning" to "deployment-scope and jurisdiction-context partitioning". |
| 10 | 30 | "AC: separate files per market (NYC only in MVP...)" | Accidental leftover (minor) | Change to "AC: catalogue files carry `jurisdictions_offered`; market reference data versioned (NYC only in MVP)". |
| 10 | 52 | "only from the market catalogue ... exists in catalogue for customer market" | Accidental leftover requiring correction | Change to "only from the catalogue ... exists in the catalogue version and its `jurisdictions_offered` includes the customer's resolved jurisdiction". |
| 10 | 53 | "gates for consent, market, teen, suppression and affordability-flag input" | Accidental leftover requiring correction (+ scope leak, D-02) | Change to "gates for consent, jurisdiction (PDP), minor protection, suppression (incl. credit-platform 'pause offers')". |
| 11 | 12 | "jurisdiction/market reference-data model" | Intentionally retained context | Keep. |
| 11 | 86, 90-91 | "other US states and UK ... deferred" | Future expansion | Keep. |
| 12 | 3 | "the planning pack (files 01-11)" | Accidental leftover requiring correction | Change to "files 00-15". |
| 12 | 13 | "PostgreSQL per market" (D5) | Accidental leftover requiring correction | Change to "PostgreSQL per deployment". |
| 12 | 16, 39, 41, 58, 67-68, 93-94 | "US/UK installs SUPERSEDED", "Consumer Duty ... SUPERSEDED" | Historical information | Keep. |
| 12 | 31-34 | "$150 vs $45", "GBP750", "UK income bands" | Historical information (PDF) | Keep. |
| 12 | 44 | "language requirements per market" | Accidental leftover requiring correction | Change "per market" to "per jurisdiction (US, US-NY, US-NY-NYC)". |
| 13 | 50, 52, 91, 92, 164, 267, 329 | "$10bn", "$30bn", "$20m" | Intentionally retained context (legal thresholds) | Keep. |
| 14 | 12-16, 18 | "US and UK, separate installs" (Before column) | Historical information | Keep (but see D-15 for l.18). |
| 14 | 42, 463, 469, 556 | "`GB`, `GB-WLS`, `EU`" | Future expansion | Keep. |
| 14 | 470 | "market reference data" | Intentionally retained context | Keep. |
| 14 | 490 | "ZIP/postcode" | Intentionally retained context | Keep. |
| 15 | 171 | "Replace the old `market` claim" | Historical information | Keep. |

## 5. Missing requirements and dependencies (truly absent or materially incomplete)

| # | Gap | Evidence of absence | Proposed home |
|---|---|---|---|
| G-1 | **Human review workflow.** `CTL-HUMAN-REVIEW` / `requires_human_review` exist, but there is no reviewer queue, SLA, UI, outcome recording or plugin handoff. | grep: no "review queue"; 06 M7 and 15 have only the flag | 06 M6/M7, 15 §2 (review outcome endpoint), 10 new story (D-29) |
| G-2 | **Role-to-scope/screen permission matrix.** Roles exist (08 SEC-02, 15 §5) and scopes exist (15 §5), but no mapping, and no dashboard access rules (e.g. who may see audit, simulator or the reveal function). | none | 08 new SEC-19 table (D-30) |
| G-3 | **DR / backup / restore.** Only "encryption ... backups" (08 SEC-06) and "BCDR hooks if covered entity" (NYC-01). No RPO/RTO and no restore test for hash-chained audit and decision log. | grep: no RTO/RPO/disaster | 08 new SEC-20, 05 §9 NFR row (D-31) |
| G-4 | **DSR deletion vs append-only, hash-chained stores.** PRV-08 and the `POST /v1/dsr-requests` endpoint exist, but there is no design for deleting from `audit_record`/`policy_decision_log` without breaking the hash chain (e.g. tokenised subject refs + key destruction, or tombstones). | grep: no erasure/tombstone/crypto-shred | 07 §8 new bullet, 08 PRV-08 (D-32) |
| G-5 | **Fairness testing method.** Metric definitions, thresholds, how group proxies are obtained without inference, minimum group sizes, cadence and escalation are missing. Only "outcome parity", Fairlearn and the LL144 format are named. | 08 CMP-09, 03 §3.4 | 08 CMP-09 expansion + D6 (D-33) |
| G-6 | **Data quality rules.** "Data-quality flags" (06 M1) and `DATA_QUALITY_INSUFFICIENT` (15) exist, but there are no DQ dimensions, thresholds (e.g. minimum months of history), owner, or ingest rejection criteria beyond schema. | grep: "data quality" absent | 07 new §4.1, 06 M1 (D-34) |
| G-7 | **Accessibility** of the standalone dashboard (and staff-facing explanation text). | grep: no accessib/WCAG | 05 §9 NFR row, 10 S10.1 AC (D-35) |
| G-8 | **Synthetic data realism/validation method.** How generated distributions are checked (sanity statistics, edge-case coverage, label-leakage audit) and who signs off G4. | 07 §7 "Realism [ASSUME]" only | 07 §7 new row, 10 S4.4 AC (D-39) |
| G-9 | **Retention outcome tracking data and contract** (see D-09). | 07 `retention_action` has no outcome fields; 15 has no endpoint | 07, 15 |
| G-10 | **Operational runbooks and documentation.** No operator runbook (pack activation and rollback, key rotation, fail-closed recovery), user guide or reason-code library ownership/versioning. | grep: no runbook/"reason code library" | 11 P12 deliverables (D-40) |

Verified **present** (not gaps):
- consent withdrawal (07 §8, PRV-03/09, 15 `consent.applied`)
- model monitoring/drift (CMP-07, 03 §3.11, S7.3)
- observability (03 §3.11)
- vendor/LLM terms (03 §3.14, SEC-15, PRV-13, NYC-02)
- performance targets (05 §9, 15 §7)
- test strategy elements (03 §3.12, 14 §14, 15 §13, S12.3)
- threat model (S12.1)
- secrets (SEC-16)

## 6. Prioritised defect list

| ID | Sev | File(s) and line | Problem | Exact proposed fix |
|---|---|---|---|---|
| D-01 | High | 14 l.497 (§13.3); 07 l.281 (§8.1); 08 l.79 (CMP-01), l.107 (NYC-03); 06 l.27 (M1), l.52 (M4); 05 l.161; 10 l.42 (S6.3) | Minor block list inconsistent. 14/07 block only SEGMENTATION_COMMERCIAL, CHURN, PRODUCT_RECOMMENDATION, MARKETING_ELIGIBILITY, RETENTION, CREDIT_REFERRAL. That leaves `PROFILE_BUILD`, `NEEDS_PREDICTION`, `MODEL_TRAINING`, `EXPLANATION_GENERATION` and `ASSISTANT_QA` permitted for minors. 08 CMP-01/NYC-03 allow educational insights **only after an approval record** and forbid training on minors' data. 05 l.161 says minors "stop after M2" (ambiguous since renumbering: M2 now precedes M1). 10 S6.3 says "no features/predictions". The "locked" vs "approval-record unlockable" semantics are unstated. | (1) 14 l.497: replace the activity list with "`SEGMENTATION_COMMERCIAL`, `CHURN_PREDICTION`, `PRODUCT_RECOMMENDATION`, `MARKETING_ELIGIBILITY`, `RETENTION_SUGGESTION`, `CREDIT_PRODUCT_REFERRAL`, `MODEL_TRAINING` always BLOCK; `PROFILE_BUILD`, `NEEDS_PREDICTION`, `SEGMENTATION_INSIGHT`, `EXPLANATION_GENERATION`, `ASSISTANT_QA` BLOCK unless a `MINOR-APPROVAL-*` record allows a named non-commercial educational use; `TXN_ENRICHMENT` limited to age/routing checks". (2) 14 l.498: append "`CTL-MINOR-PROTECT` is locked against packs and host overlays; only a signed `MINOR-APPROVAL-*` record can relax the second group, never the first". (3) 07 l.281: replace the list with "see file 14 §13.3 (two-tier block list)". (4) 05 l.161: replace "Minor customers stop after M2 for any commercial path" with "Minor customers are blocked per file 14 §13.3 after age/routing checks in M2; no M1 profile or later module output without a legal approval record". (5) 06 l.27 M1: add `CTL-MINOR-PROTECT` to the Controls list. (6) 06 l.52: change "post-PEP tags needs insight-use vs commercial-use" to "... ; minors blocked unless approval record (file 14 §13.3)". (7) 10 l.42 S6.3: keep the AC and add "aligned to file 14 §13.3 two-tier list". |
| D-02 | High | 10 l.53 (S8.2); 08 l.83 (CMP-05); 02 l.57 (§5 item 4); 13 l.91 (F05 controls cell) | "Affordability" appears as a gate or rule we apply. This contradicts 02 l.30 and 09 l.34 ("Won't: ... affordability"). There is no field, contract or O-candidate for it. | 10 l.53: replace "and affordability-flag input" with "and credit-platform suppression flags (pause offers)". 08 l.83: replace "respect 'offers follow segment eligibility, affordability first'" with "the PDF rule 'offers follow segment eligibility, affordability first' is owned by the credit platform; we never assess affordability and only honour a credit-platform suppression flag (07 O2 / 15 O2)". 02 l.57: change to "Offers follow eligibility; affordability is assessed only by the credit platform and reaches us, if at all, as a suppression flag (DOC p.31, Spec s4)". 13 l.91: replace "Affordability-first and suitability rules." with "Suitability rules (affordability is assessed only by the credit platform)." |
| D-03 | Med | 08 l.186-200 (§4 D1-D11); 12 l.7-17 (§A D1-D9), l.3; 07 l.42, l.280; 13 l.210, l.250; 02 l.96 | The same D-IDs denote different decisions in 08 and 12 (e.g. D8 = NY CDPA in 08, superseded US/UK installs in 12). 12 claims to compile all decisions but omits 08 D2, D3, D5, D6, D8, D9, D10, 14 J2-J9 and 15 I1-I6. 08 D1 duplicates 12 D9, 08 D4 duplicates 12 D6, 08 D11 duplicates 12 N4 and 14 J1. | Rename the 08 §4 IDs D1..D11 to `SD1..SD11` (security/privacy decisions) and update the referrers: 07 l.42 "D2 in file 08" -> "SD2 in file 08"; 07 l.280 "D1" -> "SD1"; 13 l.210 "decision D2 in doc 08" -> "SD2 in file 08"; 13 l.250 "D1 in doc 08" -> "SD1 in file 08". In 12, add section "G4. Cross-file decision index" listing 08 SD1-SD11, 14 J1-J9 and 15 I1-I6, each mapped to its 12 equivalent (SD1=D9, SD4=D6, SD7=N1, SD8=Q4, SD10=J4, SD11=N4=J1) or marked "new". |
| D-04 | Med | 07 l.288-298 (§9 O1-O9); 15 l.250-260 (§11 O1-O9); 15 l.299 (I7) | O-IDs diverge: 07 O6 = readiness record, O7 = next-best offers, O8 = eligibility pre-check, O9 = PD/grade. 15 O6 = referral handoff, O7 = eligibility pre-check, O8 = PD/grade, O9 = next-best offers. 15 omits the readiness record. | Make 07 §9 the master list. In 15 §11 renumber to match 07 (O6 readiness record [OPEN][MANAGER] "not built in MVP", O7 next-best offers, O8 eligibility pre-check, O9 PD/grade) and add **O10** credit referral handoff (Us -> credit/host). Add O10 to 07 §9 with the same text. Update 15 l.299 "O1-O9" to "O1-O10" and 12 l.11 "all nine candidates" to "all ten candidates". |
| D-05 | Med | 14 l.479 (§13.1 denylist `approve*`, `decline*`); 15 l.121 (§3.3); 07 l.38 (status `DECLINED`); 15 l.33 (`DECLINED` outcome), l.37 (policy packs read incl. approval metadata), l.226 ("Model approved") | The forbidden-term linter would fail on legitimate, already-designed terms (customer declined an offer; policy/model `APPROVED`). Teams will then weaken the linter ad hoc. | 07 l.38 and 15 l.33: rename status `DECLINED` to `NOT_TAKEN_UP`. 14 l.479 and 15 l.121: add "Linter scope: credit-decision terms in recommendation, insight, prediction and event payload schemas; explicit allow-list for governance fields `approval_status`, `APPROVED` (policy/model lifecycle), `approved_by`, `approved_at`". |
| D-06 | Med | 12 l.21-26 (§B M1-M4); 14 l.22-31 (goals G1-G8) and refs in 03 l.26/182, 04 l.89; 05 l.10-20 (principles P1-P9); 15 l.11-16 (C1-C6) and refs 03 l.70, 04 l.32; 10 story IDs S13.x-S19.x; 13 Q-C/Q-P/Q-A vs §6 Q1-Q24; 12 l.32 "Q5, p.44" | ID schemes collide across files: M1-M4 (missing material) vs modules M1-M9; 14 goals G1-G8 vs roadmap gates G1-G12; 05 P1-P9 vs phases P1-P12; 15 C1-C6 vs file 13 rows C01-C07; stories S13.1 vs file 13 row S13; Q1 (13 §6) vs Q-C1 (13 §2); PDF "Q5". | 12 §B: rename M1-M4 to MS1-MS4. 14 §1: rename G1-G8 to PG1-PG8 and update 03 l.26 and l.182 ("file 14 G1" -> "file 14 PG1") and 04 l.89. 05 §1: rename P1-P9 to AP1-AP9. 15 §1: rename C1-C6 to IC1-IC6 and update 03 l.70 and 04 l.32 ("file 15 C4" -> "file 15 IC4"). 10: prefix story IDs with "ST" (ST13.1 ...), or always write "story S13.1" vs "row S13". 13 §6: number the questions L1-L24 and update 12 §G2 headings (Q1 -> L1 ...), 10 l.132 ("Q4, Q13, Q14" -> "L4, L13, L14") and 02 l.100. 12 l.32: write "(PDF Q5, p.44)". |
| D-07 | Med | 09 l.22 (Should: simulator, golden cases, pack maker-checker); 09 l.86 (§7 MVP slice includes simulator + golden cases); 10 l.107 (E15 [Must]), l.122 (E17 [Must]); 11 l.81 (P8A gate before demo) | Priority conflict. 09 MoSCoW rates the pack workflow and simulator as Should; 09 §7 and 10 treat them as Must, and the demo step "legal-review-pending fail-safe" needs them. | 09 l.22: change "Should" to "Must" and the rationale to "Needed for legal-review-pending demo and pack approval (file 14 §9, §14)". |
| D-08 | Med | 09 l.28 (Should: model versioning + drift); 10 l.48 (S7.3 [S]); 14 l.282 (`CTL-MODEL-APPROVED-ONLY`, M4/M5); 08 l.84 CMP-06 | `CTL-MODEL-APPROVED-ONLY` (in Must PEPs) needs a model registry with status, but the registry is only Should. | 10 l.48: split S7.3 into "S7.3a Model registry with status DRAFT/APPROVED/RETIRED, version on every prediction. M." and "S7.3b Drift/monitoring view. S.". 09 l.28: change to "Must: model registry with approval status; Should: drift/monitoring view". |
| D-09 | Med | 02 l.17 ("outcome tracking"); 06 l.16 (M7 outputs "outcome records"); 07 l.39 (`retention_action` has no outcome fields); 15 l.33-34 (feedback only for recommendations) | Retention outcome tracking is in scope but has no data model or contract. | 07 l.39: add fields `outcome (NOT_ACTIONED/CONTACTED/RETAINED/CLOSED/UNKNOWN)`, `outcome_reason`, `outcome_at`, `outcome_source (HOST/STAFF)`. 15 §2: add row "Retention action outcome \| `POST /v1/retention-actions/{action_id}/outcomes` \| Host reports outcome + reason \| `ri.retention.feedback` \| -" and add `ri.retention.feedback` to the 15 §5 scopes. |
| D-10 | Med | 02 l.48; 13 l.219 (§5.4) | "Eligible-to-be-assessed candidates" implies an eligibility judgement by us (13 Q-P2 flags it as close to the line). | 02 l.48: replace "We may only **surface as eligible-to-be-assessed candidates**; eligibility and decision come from credit platform" with "We may only emit a `CREDIT_REFERRAL` ('refer to the credit platform for assessment') with no eligibility, limit, rate or likelihood; eligibility and decision come only from the credit platform". 13 l.219: replace "surfaced only as 'eligible to be assessed by credit platform'" with "surfaced only as `CREDIT_REFERRAL` (refer for assessment), with no eligibility claim". |
| D-11 | Med | 15 l.12 (C2), §12 l.271; 13 l.90 (F04 question), l.92 (F06) | Nothing restricts host use of our outputs (profile features, scores) for credit eligibility. That creates FCRA "consumer report" and Reg B risk and indirectly brings credit decisioning into scope. | 15 §1: add principle "IC7 Purpose limitation: every response and event carries `permitted_use: ["RETAIL_INSIGHT","MARKETING_SUGGESTION"]` and `not_for_credit_decisioning: true`; host integration terms prohibit use of outputs for credit eligibility, pricing or limits [LEGAL-CONFIRM]". Add the two fields to the 15 §3.2 response example. Add to 08 CMP-03: "outputs labelled not-for-credit-decisioning (file 15 IC7)". |
| D-12 | Low | 00 l.12, l.48-50, l.58, l.78 | Stale status: steps 7-9 "running/pending", "the research agent is consulting", file index shows 13-15 "(pending)", "Not yet read end to end". File 17 is not indexed. | l.48-50: set Output to "File 13", "Files 14, 15", "Updated files 02-12" and the Who column to "(done)". l.58: change "is consulting"/"is designing" to past tense. l.78: remove "(pending)" three times; append "\| 16 validation review \| 17 BMAD installation recommendation". l.12: append "Validation review: file 16". |
| D-13 | Low | 05 l.298 | Change log says "numbering mismatch with files 06/14 noted", but §2 l.36 says aligned and 00 l.56 says fixed. | Replace with "numbering aligned with files 06/14 (M1 = Customer Intelligence, M2 = Transaction Intelligence)". |
| D-14 | Low | 14 l.233 | "A monotonicity property is tested ... (§9)", but testing is §14 (§9 is the approval workflow). | Change "(§9)" to "(§14)". |
| D-15 | Low | 14 l.18 | "Consequence for other files (to be updated by another agent)" is a stale process note; the updates are done. | Replace "(to be updated by another agent)" with "(applied in files 07, 08, 09 on 2026-11-09)". |
| D-16 | Low | 14 l.363-364 | The illustrative YAML cites "row NYC-?" and `matrix_ref: "13#NYC-TBD"`, but NYC rows in file 13 are C01-C07 (NYC-xx are file 08 requirement IDs). | Change to `citation: "TBD - see 13-nyc-regulatory-applicability-matrix.md row C0?"` and `matrix_ref: "13#C-TBD"`. |
| D-17 | Low | 14 l.529-531 (§15) | `POL-US-0003`, `POL-US-0004` and `POL-US-NY-0002` are shown as "APPROVED/APPLICABLE (illustrative)", contradicting the invariant "illustrative: true cannot be APPROVED" (l.361, l.384). | In the Status cells, replace "APPROVED/APPLICABLE (illustrative)" with "APPROVED/APPLICABLE (hypothetical - assumes legal approval and `illustrative: false`)". |
| D-18 | Low | 14 l.298-306 (§6) vs 14 §5 l.267-283 and 06 l.65, l.71 | The §6 "Main controls" per module omit controls that §5 says are enforced there: MINOR-PROTECT for M6/M7/M9, EXPLANATION for M6/M7, ADVERSE-ACTION-HANDOFF for M6, HUMAN-REVIEW for M7. 06 M7 also omits HUMAN-REVIEW and EXPLANATION. | 14 §6: M6 row add "MINOR-PROTECT, EXPLANATION, ADVERSE-ACTION-HANDOFF"; M7 row add "MINOR-PROTECT, HUMAN-REVIEW, EXPLANATION"; M9 row add "MINOR-PROTECT". 06 l.71: add `CTL-HUMAN-REVIEW`, `CTL-EXPLANATION`. |
| D-19 | Low | 14 l.192 (`MOBILE_APP`, ...); 14 l.530-537; 15 l.108, l.110 (`APP`) | Channel enum drift: `APP` vs `MOBILE_APP`. | In 14 §15 and 15 §3.2, replace `APP` with `MOBILE_APP` (keep `EMAIL`, `PUSH`). |
| D-20 | Low | 04 l.103; 15 l.90 | `protection_status` lists only `ADULT\|MINOR_PROTECTED`; `AGE_UNKNOWN` (07 l.27, 14 l.193) is missing. | 04 l.103: change to `"ADULT\|MINOR_PROTECTED\|AGE_UNKNOWN"`. 15 §3.2: add a comment or enum note listing all three values. |
| D-21 | Low | 14 l.171 (`uuid decision_id`); 07 l.62/176 (`string`); 14 l.316, 15 l.92 (`dec_01J...`) | `decision_id` type mismatch. | 14 l.171: change `uuid decision_id PK` to `string decision_id PK` (prefixed ULID `dec_...`). |
| D-22 | Low | 06 l.25 | M1 interface `GET /customers/{id}/profile` does not match 15 §2. | Replace with "`POST /v1/customers/{customer_token}/insights:evaluate`, `GET /v1/customers/{customer_token}/insights` (file 15 §2)". |
| D-23 | Low | 15 l.32 (Recommendations row), l.47 | Activity mapping omits `CREDIT_PRODUCT_REFERRAL` and `MARKETING_ELIGIBILITY`; the composite evaluate endpoint is not in the §2 table. | l.32 Activity cell: "PRODUCT_RECOMMENDATION, CREDIT_PRODUCT_REFERRAL, MARKETING_ELIGIBILITY". Add a §2 row "Composite evaluate \| `POST /v1/customers/{customer_token}/evaluate` \| Aggregate (file 04) \| union of sub-scopes \| per sub-result". |
| D-24 | Low | 08 l.4 (owner legend); 08 l.108, 110, 112, 114 ("CMP", "BSA Officer"); 12 l.79-89; 14 l.422-423 ("Release role", "Ops") | Owner codes CMP/BSA are used but not in 08's legend (CMP also clashes with the CMP-xx requirement prefix). The release/ops role is not in SEC-02. | 08 l.4: append "; COMP = compliance; BSA = BSA/AML officer". Replace owner "CMP" with "COMP" in 08 l.108/110/112 and 12 l.79/81/85/86/88. 08 l.39 SEC-02: add role `policy_release` (ops activation) to the list; mirror in 15 l.172. |
| D-25 | Low | 11 l.28-38 (§3), l.10, l.82 | The gate table lacks G3A, G4, G5 and G9 decision-makers. The P1 entry says "Planning docs 01-11". The P10 (changed) entry omits its P8A dependency for the pending-review demo. | §3: add rows "G3A \| Manager + compliance engineer \| Policy layer", "G4 \| Manager \| Synthetic data", "G5 \| Manager \| Transaction slice", "G9 \| Manager + security \| AI safety". l.10: "Planning docs 00-15 drafted". l.82 Entry cell: "P9, P8A". |
| D-26 | Low | 10 l.148, l.150 | Story counts are wrong: E13-E19 contain 33 stories (not ~35); total 83 (not ~85). | l.148: "about 35 stories" -> "33 stories". l.150: "about 85 across 19 epics" -> "83 across 19 epics". |
| D-27 | Low | 04 l.23, l.68, l.70 | Section references use "s6", "s12", "s7" instead of "§". | Replace "s6" -> "§6", "file 14 s12" -> "file 14 §12", "(s7)" -> "(§7)". |
| D-28 | Low | 10 l.23 vs l.135; l.72 vs l.137; l.71 vs l.135 | Duplicate stories: S3.2 Contract v1 ~ S19.1; S11.2 mock host ~ S19.3; S11.1 contract publication ~ S19.1. | 10 l.135/137: add "(supersedes/extends S3.2 and S11.1)" to S19.1 and "(same as S11.2; merge at BMAD re-homing)" to S19.3. |
| D-29 | Med | 06 M6/M7; 10 E8/E10; 15 §2 | Gap G-1: no human review workflow. | 10 E8: add "S8.5 Human review queue. As relationship staff, I want items flagged `requires_human_review` in a queue with approve-to-show/reject + mandatory reason, so that obligations are met. AC: in standalone a dashboard queue; in plugin a `review_required` flag + outcome via `POST /v1/recommendations/{rec_id}/outcomes`; decisions audited (CMP-14). S." 06 l.65: append "human-review outcome recorded per CMP-14". |
| D-30 | Med | 08 §2.1; 15 §5 | Gap G-2: no role-to-scope/screen matrix. | 08: add "SEC-19 Role-to-scope matrix (roles x 15 §5 scopes x dashboard screens: customer view, audit, policy packs, simulator, approval); deny by default; maintained with the OpenAPI spec \| [REC] \| - \| DEV / SEC / MGR". 10 S12.1 AC: add "role-to-scope matrix implemented and tested". |
| D-31 | Med | 08 §2.1; 05 §9 | Gap G-3: no DR/backup. | 08: add "SEC-20 Backup and restore: encrypted backups of DB, decision log and signed packs; restore test verifies the hash chain; RPO/RTO targets [MANAGER] (standalone demo: none)". 05 §9: add row "Backup/restore \| RPO/RTO [MANAGER]; restore test keeps audit hash chain verifiable \| [REC]". |
| D-32 | Med | 07 §8 l.270-276; 08 l.68 PRV-08; 15 l.43 | Gap G-4: DSR deletion conflicts with append-only, hash-chained audit and decision log. | 07 §8: add bullet "DSR delete/objection: personal rows deleted or anonymised; audit and decision-log rows keep only `subject_ref` tokens, and the host's token mapping is destroyed (crypto-shredding), so the hash chain stays intact; method [LEGAL-CONFIRM]". 08 PRV-08: append "(method: file 07 §8)". |
| D-33 | Med | 08 l.87 CMP-09; 08 l.195 (decision on fairness proxies) | Gap G-5: fairness method undefined. | 08 CMP-09: append "Method spec (to be approved under the fairness-proxy decision in 08 §4): metrics (surfacing rate, suppression rate, impact ratio per LL144 format), alert threshold [MRM], minimum group size, proxy source (restricted store only, never inferred), cadence per batch, escalation to MRM". |
| D-34 | Med | 06 l.24; 07 §4; 15 l.158 | Gap G-6: no data-quality rules. | 07: add "§4.1 Data quality" table (completeness, minimum history months for each module, currency validity, duplicate ratio, stale-pending ratio; threshold -> `DATA_QUALITY_INSUFFICIENT`; owner DEV; thresholds [ASSUME]). 06 l.24: append "(rules: file 07 §4.1)". |
| D-35 | Low | 05 §9; 10 l.64 (S10.1) | Gap G-7: no accessibility requirement. | 05 §9: add row "Accessibility \| Dashboard WCAG 2.2 AA target; plain-language templates \| [REC] [MANAGER]". 10 S10.1 AC: add "keyboard navigation and contrast checks pass". |
| D-36 | Low | 08 l.89 (CMP-11); 13 l.92 (F06 controls) | The wording "so specific principal reasons exist if an output ever feeds a credit process" normalises a future credit use. | 08 l.89: replace with "(fixed reason codes support explainability; outputs must not feed credit decisions - any such use needs a new manager + legal approval and is out of scope)". 13 l.92: replace "so that if outputs ever feed a credit decision, specific principal reasons exist" with "(outputs are not to feed credit decisions; see file 15 IC7)". |
| D-37 | Low | 07 l.298 (O9); 12 l.11 (D3) | PD/grade use "for marketing without legal approval" implies legal-only gating of a credit-risk score for targeting. | 07 l.298 and 12 l.11: change to "not consumed in v1; any future use requires manager approval, legal confirmation and a fairness review, and never as a ranking feature for credit products". |
| D-38 | Low | 04 l.188 | The shadow mode flag's location is unspecified (risk of `if MODE` logic in core). | Append "Implemented in the inbound API adapter/response layer via config; core is unaware of shadow mode". |
| D-39 | Low | 07 l.262; 10 l.31 (S4.4) | Gap G-8: no synthetic-data validation method. | 07 §7: add row "Validation \| Distribution sanity report, edge-case coverage, label-leakage audit, persona balance; reviewed at G4 \| [REC]". 10 S4.4 AC: add "validation report produced". |
| D-40 | Low | 11 l.21 (P12 deliverables) | Gap G-10: no runbooks or reason-code library governance. | 11 P12 Deliverables: append "operator runbook (pack activation/rollback, signing-key rotation, fail-closed recovery), reason-code library (versioned, owner COMP)". |
| D-41 | Low | 01 l.5; 09 l.3; 10 l.3; 12 l.3 | [LEGAL-CONFIRM] is missing from the label legends (09, 10 and 12 use it in the body). 01 has no NYC banner. | Add "[LEGAL-CONFIRM] legal/compliance must confirm" to each legend. 01: insert after l.3 "> NYC scope update: US/UK constraints below are PDF context; for this subsystem they are superseded by file 02 §8 and file 14 §0." |
| D-42 | Low | 03 l.204; 12 l.64; 10 l.93 | Question references are ambiguous: "Q-legal 7"; N1 cites "file 13 Q1-Q3" for charter **and products/channels** (products are Q-P1..P9). | 03 l.204: "Q-legal 7" -> "file 13 §6 Q7" (or L7 after D-06). 12 l.64 and 10 l.93: "(file 13 Q1-Q3)" -> "(file 13 §2.1 Q-C1..Q-C8, §2.2 Q-P1..Q-P9; §6 Q1-Q3)". |
| D-43 | Low | 13 l.210 (§5.3) vs 07 l.42, 04 l.98, 15 l.65 | Consent purpose names differ (`SERVICE_INSIGHT`, `MARKETING_PERSONALISATION` vs `INSIGHT`, `MARKETING`, `PERSONALISATION`). Both are marked illustrative. | 13 l.210: replace the purpose list with "`INSIGHT`, `PERSONALISATION`, `MARKETING`, `AFFILIATE_MARKETING`, `CHANNEL_EMAIL`, `CHANNEL_SMS`, `CHANNEL_CALL`, `MINOR_EDU`" to match 07 (still illustrative, final per D2/SD2). |
| D-44 | Low | 14 l.53-72 (§3.1); 07 l.36 vs 15 l.252 | 14 §3.1 lacks the `illustrative` and `supersedes[]` fields that 07 l.56, 14 §4.2 and §8.1 use. Segment source enum differs (`source=HOST` vs `source:"CREDIT"`). | 14 §3.1: add rows "`illustrative` \| bool \| true -> cannot be APPROVED" and "`supersedes[]` \| policy refs \| explicit pre-emption only (§4.2)". 15 l.252: change `source:"CREDIT"` to `source:"HOST"` with `origin:"CREDIT_PLATFORM"`, or add CREDIT to the 07 l.36 enum. |

Also noted (no defect raised): 15 l.43 offers a DSR endpoint in v1 while 09 l.50 defers "Full DSR support". This is acceptable if the 15 row is marked "placeholder; full support post-MVP". It is optional to add that text.

## 7. Verified consistent (short list)
1. One shared core; standalone and plugin differ only by adapters and config, and this is stated in every architecture, MVP, epic and contract file (§3).
2. No credit-decision type, field, activity or event exists in 07, 14 or 15. `CREDIT_REFERRAL` is the only credit output. The credit platform owns S0-S5, early warning, readiness record, PD and adverse action.
3. NYC-first (federal + NY State + NYC; `deployment_scope=[US-NY-NYC]`; no 50-state rulebooks) is consistent in 02-15. UK and other states are future expansion everywhere, with [MANAGER confirm].
4. Fail-safe semantics (PENDING_LEGAL_REVIEW -> BLOCK, UNKNOWN/UNSUPPORTED jurisdiction -> restrictive default, review_due expiry, PDP error -> DENY) agree across 04, 05, 06, 08, 09, 10, 11, 14 and 15.
5. All `CTL-*`, `EXCL-*`, SEC/PRV/CMP/NYC and file 13 row IDs referenced are defined. The file 13 tally is arithmetically correct.
6. Module numbering (M1 Customer, M2 Transaction), the build order and the PDP-in-M8 ownership are consistent.
7. Epic-to-phase-to-gate mapping (E13-E19 to P2A, P3A, P4A, P8A and G2A, G3A, G4A, G8A) is consistent between 10 and 11.
8. Legal disclaimers ("not legal advice", [LEGAL-CONFIRM], applicability depends on charter) appear in every file that carries legal content. No file asserts a law applies.
9. Minors are excluded from MVP processing (adults 18+; minors are synthetic only, to prove the block path) in 02, 07, 08, 09, 10 and 14. Only the detailed block list conflicts (D-01).
10. Consent withdrawal is designed end to end (07 §8, PRV-03/09, `consent.applied` event, 09 success criterion, 15 scenario library).
