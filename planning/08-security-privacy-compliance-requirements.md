# 08 - Security, Privacy and Compliance Requirements

**Label key:** [DOC p.N] PDF page | [REC] recommendation | [ASSUME] assumption | [UNKNOWN] | [MANAGER] needs manager approval | [LEGAL-CONFIRM] legal/compliance must confirm.
**Owners:** DEV = subsystem developer; MGR = manager; SEC = bank security / CISO; LEG = legal/compliance; PRIV = privacy office; MRM = model risk / validation; HOST = host platform team; COMP = compliance; BSA = BSA/AML officer.
**Legal-point classes** (same as file 13 §1.3): **BINDING LAW** (statute/regulation, if the institution is within its scope) | **SUPERVISORY GUIDANCE** (agency guidance or letters; no enforceable requirement by their own terms) | **RECOMMENDED CONTROL** (a design or policy choice of this plan; not required by the cited source) | **UNRESOLVED APPLICABILITY** (whether or how a source reaches this bank or activity is open). Every requirement in §2 is a RECOMMENDED CONTROL unless a row says otherwise; §3 shows the class of each cited source.

> This document does **not** state what any law requires and is **not legal advice**. Legal research lives in **`13-nyc-regulatory-applicability-matrix.md`**; this file only references its rows by ID (e.g. F06, S12, C02) with the **planning applicability estimate and verification status recorded there**. Applicability depends on the bank's charter, regulatory status, products and activities, which are **[UNKNOWN]** (file 13 §2, Q-C1 to Q-C8). Every applicability statement is [LEGAL-CONFIRM]. Controls below are **design requirements** we build regardless, so that whatever legal concludes, the mechanism exists. All figures in the PDF are illustrative and "not legal advice" [DOC p.48].
>
> **Scope update (2026-11-09, authoritative user requirement):** MVP jurisdiction is **New York City, USA** (US federal + New York State + New York City only); no 50-state rulebooks. Legal rules are enforced through **jurisdiction policy packs** and the control catalogue in `14-jurisdiction-policy-layer-design.md` §5. UK and other markets are future expansion [MANAGER confirm].
>
> **Two-track scope update (2026-11-10, authoritative; file 21; file 12 DEC-50..DEC-64):** Track A "Conventional" = NYC + London (UK back in scope; DEC-13 superseded); Track B "Islamic" = Islamic bank, UAE onshore [ASSUME]. Two-track requirements: §2.5. Other US states and other countries stay future expansion.

## 1. Relevant PDF rules (context for this subsystem)

| Rule from PDF | Page | Relevance to us |
|---|---|---|
| Runs inside the bank; customer data does not leave | p.3, p.49, p.52, p.74 | Deployment constraint |
| Separate US and UK installations, no data crosses | p.3, p.40, p.54, p.74 | Superseded for this subsystem by deployment scoping (MVP single `US-NY-NYC` deployment); residency-driven separate deployments only for future markets [LEGAL-CONFIRM] |
| Names / SSN / NI numbers tokenised on arrival; keys in bank vault; only "reveal" permission sees names; every view logged | p.8, p.16, p.74 | PII handling |
| Consent per data source, purpose, expiry, withdrawal; without valid consent data "cannot be used at all"; withdrawal -> recalculate without it | p.8, p.16, p.56, p.74 | Consent gating |
| No marketing profiling until 18; under-18s only non-credit products/loyalty; credit never promoted to under-18s | p.31, p.43, p.44, p.67, p.81 | Minors |
| Marketing and next-best-offer respect segment eligibility, affordability first | p.64, p.81 | PDF context only: affordability is assessed only by the credit platform; we never assess it and only honour a credit-platform suppression flag (CMP-05) |
| LLM never decides, never chooses reasons, runs inside bank, every number checked, template fallback, person approves letters | p.39-40, p.73 | LLM guardrails |
| Bank systems send all customer communications; system never changes accounts itself | p.53, p.69 | Advisory-only outputs |
| Model needs model-risk approval; drift monitoring; performance drop -> switch off and fall back; overrides recorded with reason | p.32, p.38, p.43, p.71 | Model governance |
| Income bands / postcode can be proxies for protected characteristics; fairness tests by group; sensitive test data in a separate locked area models cannot read | p.30, p.38, p.43, p.64, p.81 | Fairness |
| Every decision explainable, recorded, replayable | p.13, p.21, p.50, p.75 | Audit / explainability |
| Maker-checker for policy/config changes | p.9, p.63 | Change control (now also policy packs) |
| Consumer Duty: good outcomes by customer group incl. vulnerable; vulnerability flag -> trained person | p.4, p.17, p.54 | UK regime - future market only; the vulnerability-routing control (CMP-15) is kept as internal baseline |
| Contact limits on frequency, time, channel; vulnerability flags to specialists | p.41 | Retention outreach |
| SOC 2 Type II, ISO 27001 aligned, annual pen test, SBOM, signed offline updates | p.40, p.74 | Security evidence |

## 2. Requirements

Control-catalogue cross-reference: the "CTL" column names the file 14 §5 control that enforces the requirement at runtime through the PDP and module PEPs. "-" means the requirement is a build/process control, not a policy control.

### 2.1 Security (SEC)

| ID | Requirement | Source | CTL (file 14) | Owner |
|---|---|---|---|---|
| SEC-01 | All API calls authenticated with OIDC/OAuth2 bearer tokens validated against a JWKS; no anonymous endpoints except `/health` | [REC]; file 15 §5 | - | DEV / HOST |
| SEC-02 | Role-based authorisation (proposed roles: viewer, analyst, relationship_staff, model_admin, auditor, **policy_author, policy_approver, policy_release** (pack build/sign/activation by ops), service_host); deny by default; least privilege with periodic access review; role-to-scope matrix in SEC-19 | [REC]; file 13 S01 (500.7, if covered entity) | - | DEV / MGR |
| SEC-03 | Token must carry `ri_deployment` equal to the instance `deployment_id` and `ri_jurisdictions` covering the request's jurisdiction context; mismatch -> 403 `JURISDICTION_NOT_PERMITTED`, audited (replaces the old `market` claim) | [REC]; file 15 §5 | `CTL-DATA-RESIDENCY` | DEV / HOST |
| SEC-04 | One DB, secret set and key set per deployment (MVP: one `US-NY-NYC` deployment); no network path between deployments; future residency-driven deployments use the same image | [DOC p.54, p.74]; file 14 §12 [LEGAL-CONFIRM] | `CTL-DATA-RESIDENCY` | DEV / SEC |
| SEC-05 | TLS 1.2+ in transit (mTLS for service-to-service where host requires) | [REC]; file 13 F02, S01 (500.15) | - | DEV / SEC |
| SEC-06 | Encryption at rest for DB and backups; sensitive fields additionally encrypted; keys in bank key system (standalone: local dev KMS) | [DOC p.74] [REC]; file 13 F02, S01 | - | SEC / DEV |
| SEC-07 | Append-only, hash-chained audit records for every read of customer-level data, every evaluation, config change, **policy pack activation, policy decision**, model status change and LLM interaction | [DOC p.8, p.74] [REC]; file 13 S01 (500.6) | `CTL-AUDIT-LOG` | DEV |
| SEC-08 | Logs contain no PII or raw transaction descriptions; correlation IDs only | [REC] | - | DEV |
| SEC-09 | Input validation against JSON Schema; size limits; idempotency keys on ingest; request schemas have no fields for excluded attributes | [REC]; file 15 §8 | `CTL-DATA-MINIMISE` | DEV |
| SEC-10 | Supply chain: pinned dependencies, SBOM per image, signed images **and signed policy packs**, vulnerability scan in CI | [DOC p.74] [REC]; file 14 §8.1 | - | DEV / SEC |
| SEC-11 | Annual independent pen test and SOC 2 / ISO 27001 evidence are platform-level obligations; subsystem supplies inputs | [DOC p.40, p.74] | - | MGR / SEC |
| SEC-12 | Can run with no internet access (offline-capable build, local LLM or templates) | [DOC p.40, p.74]; file 13 F15 | - | DEV |
| SEC-13 | LLM prompt-injection defence: transaction descriptions and any free text treated as untrusted data, delimited, never as instructions; LLM has read-only tools with fixed allow-list; no tool can write, send messages, change accounts or decide credit | [DOC p.69, p.73] [REC] | `CTL-LLM-GUARDRAIL` | DEV |
| SEC-14 | LLM output validator: every number, product code and customer fact in generated text must match tool results; else discard and use template | [DOC p.40] | `CTL-LLM-GUARDRAIL` | DEV |
| SEC-15 | LLM data-leakage control: prompts carry tokens and derived figures only, never names/IDs; LLM runtime inside bank boundary; no prompt/response retention or training by provider | [DOC p.73] [REC]; file 13 S02 (if covered entity), F15 | `CTL-LLM-GUARDRAIL` | DEV / SEC / MGR |
| SEC-16 | Secrets via environment/secret store, never in repo or images | [REC] | - | DEV |
| SEC-17 | Rate limiting and request-size limits per client | [REC] | - | DEV |
| SEC-18 | Policy store integrity: PDP refuses unsigned or hash-mismatched packs and overlays; not `ready` without a verified bundle; fails closed (`CTL-DEFAULT-DENY`) after max bundle age | [REC]; file 14 §8, §10 | `CTL-DEFAULT-DENY` | DEV / SEC |
| SEC-19 | **Role-to-scope and screen permission matrix** (gap G-2): roles (SEC-02) x API scopes (file 15 §5) x dashboard screens (customer view, reveal request, audit/decision log, policy packs, simulator, approval, review queue, model registry); deny by default; separation of duties (maker != approver; `policy_author` cannot approve or activate; `policy_release` cannot author or approve; auditors read-only; nobody but the host "reveal" function sees names); matrix kept with the OpenAPI spec and tested in CI. Proposed matrix: file 15 §5.1 | [REC] | - | DEV / SEC / MGR |
| SEC-20 | **Backup, restore and DR** (gap G-3): encrypted backups of DB, audit store, decision log, signed packs, model artefacts and config; keys in bank key system; RPO/RTO targets [MANAGER] (standalone demo: none); restore test at least quarterly and before go-live that re-verifies audit and decision-log hash chains and pack signatures; restored instance not `ready` until verification passes; backups inherit `CTL-RETENTION` and crypto-shredding (destroyed subject keys stay destroyed after restore, file 07 §8.2); BCDR hooks into the bank programme (NYC-01) | [REC]; file 13 S01 (500.16, if covered entity) | `CTL-RETENTION` | SEC / DEV / MGR |
| SEC-21 | **Observability and security monitoring**: structured PII-free logs, OpenTelemetry traces/metrics, `/metrics`; alerts on authentication/authorisation failures and jurisdiction-claim mismatches, PDP DENY/INDETERMINATE spikes, `PENDING_LEGAL_REVIEW` hits, bundle age, overlay rejections, LLM fallback and validator failures, anomalous assistant queries, drift breaches, data-quality reject rates (file 07 §4.1), human-review queue age, nightly audit-chain verification result; forwarding to host SIEM in plugin mode | [REC]; file 03 §3.11; file 13 S02 (if covered entity) | `CTL-AUDIT-LOG` | DEV / SEC |

### 2.2 Privacy (PRV)

| ID | Requirement | Source | CTL (file 14) | Owner |
|---|---|---|---|---|
| PRV-01 | Synthetic data only; no real customer data or connection to real bank systems without explicit authorisation and approved infrastructure | Brief [REC] | - | DEV / MGR |
| PRV-02 | Customers identified only by tokens; subsystem never stores names, SSN/ITIN, account numbers, full address, biometrics; re-identification is a host "reveal" function | [DOC p.8, p.16, p.74]; file 13 F01, S10, C04 | `CTL-DATA-MINIMISE` | DEV / HOST |
| PRV-03 | Consent checked per source and purpose before processing; expired or withdrawn consent blocks that use; withdrawal triggers recompute and audit; consent snapshot reference stored on each result | [DOC p.8, p.16, p.56, p.74]; file 13 §5.3 | `CTL-CONSENT-PURPOSE` | DEV / HOST |
| PRV-04 | Purpose separation: `INSIGHT` (service/explanation) vs `MARKETING`/`PERSONALISATION` (recommendations) are separate consent purposes; channel purposes (email/SMS/call) and affiliate marketing separate | [REC] [MANAGER]; file 13 F04, F10, F11, S11 | `CTL-CONSENT-PURPOSE`, `CTL-CHANNEL-ALLOW` | DEV / LEG |
| PRV-05 | Data minimisation: only fields required by an activity are mapped in (per-activity allow-list); transaction descriptions redacted to merchant/category; birth date reduced to year-month; coarse jurisdiction codes only | [REC]; file 15 §8 | `CTL-DATA-MINIMISE` | DEV |
| PRV-06 | No inference of sensitive characteristics (health, religion, ethnicity, sexual orientation, immigration status, pregnancy, presence of children, etc.); category taxonomy excludes such labels; gambling/payday categories used only for customer protection, never targeting | Brief [DOC p.32] [REC]; file 13 S14, C02 | `CTL-NO-INFERENCE`, `CTL-SENSITIVE-EXCLUDE` | DEV / LEG |
| PRV-07 | Retention periods per entity set by policy (`CTL-RETENTION`), not hard-coded; values [UNKNOWN] until confirmed; purge in report-only mode until then | [REC]; file 07 §8 [LEGAL-CONFIRM] | `CTL-RETENTION` | LEG / DEV |
| PRV-08 | Data subject requests (access, deletion, objection to profiling) supported via host-initiated API by token; profiling opt-out hook; deletion from append-only, hash-chained stores by crypto-shredding and token destruction, never by rewriting chained rows (method: file 07 §8.2) [LEGAL-CONFIRM] | [REC] [UNKNOWN]; file 13 S15 (pending bill) | `CTL-DSR-SUPPORT` | DEV / HOST / LEG |
| PRV-09 | Marketing opt-outs, channel consents (email, SMS, call, DNC) and contact preferences from host suppress recommendations and retention outreach; revocation invalidates queued suggestions | [DOC p.41] [REC]; file 13 F10, F11, S11 | `CTL-MARKETING-SUPPRESS`, `CTL-CHANNEL-ALLOW` | DEV / HOST |
| PRV-10 | A privacy impact assessment (DPIA-style) is prepared before any non-synthetic use | [DOC p.71]; file 13 S15 | - | LEG / PRIV / MGR |
| PRV-11 | Data lineage by source type (own-experience, affiliate, consumer reporting agency, third-party); CRA or affiliate-sourced features blocked unless an approved flag exists | [REC]; file 13 F04, F16 | `CTL-CONSENT-PURPOSE`, `CTL-DATA-MINIMISE` | DEV / LEG |
| PRV-12 | Data classification tags per field (NPI / "private information") and breach-scoping query by residence jurisdiction and partition; fast forensic export | [REC]; file 13 S05 | `CTL-AUDIT-LOG` | DEV / SEC |
| PRV-13 | Vendor and data-recipient register for every processor of customer data (LLM host, cloud, analytics) | [REC]; file 13 F01, F15 | - | MGR / PRIV |

### 2.3 Compliance, governance and customer protection (CMP)

| ID | Requirement | Source | CTL (file 14) | Owner |
|---|---|---|---|---|
| CMP-01 | **Minors (16-17):** `age_status=MINOR_PROTECTED` (or `AGE_UNKNOWN`) applies the **two-tier block list in file 14 §13.3**: tier 1 (`SEGMENTATION_COMMERCIAL`, `CHURN_PREDICTION`, `PRODUCT_RECOMMENDATION`, `MARKETING_ELIGIBILITY`, `RETENTION_SUGGESTION`, `CREDIT_PRODUCT_REFERRAL`, `MODEL_TRAINING`) always BLOCK; tier 2 (`PROFILE_BUILD`, `NEEDS_PREDICTION`, `SEGMENTATION_INSIGHT`, `EXPLANATION_GENERATION`, `ASSISTANT_QA`) BLOCK unless a signed `MINOR-APPROVAL-*` record allows a named non-commercial educational use; `TXN_ENRICHMENT` limited to age/routing checks. MVP processes adults 18+ only | Brief [DOC p.44, p.67]; file 13 S12 | `CTL-MINOR-PROTECT` (locked baseline) | DEV / LEG / MGR |
| CMP-02 | Credit products never recommended or referred for anyone under 18 in any jurisdiction | [DOC p.31, p.43, p.81] | `CTL-MINOR-PROTECT` (locked) | DEV |
| CMP-03 | **No credit decisions:** subsystem never approves/declines credit, sets limits or prices, issues adverse-action content, rejects a service, changes accounts or contacts customers; no contract type, enum or activity for these exists; outputs are advisory and labelled so, and labelled not-for-credit-decisioning (`permitted_use`, `not_for_credit_decisioning: true`; file 15 IC7), with host integration terms prohibiting credit use [LEGAL-CONFIRM] | Brief [DOC p.53, p.69]; file 14 §13.1; file 15 §3.3 | `CTL-CREDIT-REFERRAL-ONLY`, `CTL-ADVERSE-ACTION-HANDOFF` (locked) | DEV |
| CMP-04 | Recommendations only from the bank-supplied, versioned product catalogue (with `jurisdictions_offered`); eligibility and terms come from bank config, never generated by code authors or the LLM | Brief [REC] | - | DEV / MGR |
| CMP-05 | Credit products shown only as `CREDIT_REFERRAL` ("refer to the credit platform for assessment"), never as pre-approved, pending a defined credit-platform integration. The PDF rule "offers follow segment eligibility, affordability first" is owned by the credit platform; we never assess affordability, hold no affordability field or input, and only honour a credit-platform suppression flag (file 15 §11 CI-2) | [DOC p.64, p.81] [UNKNOWN]; file 13 F06, F07 | `CTL-CREDIT-REFERRAL-ONLY` | DEV / MGR |
| CMP-06 | Model governance: every model has a registry entry (purpose, materiality tier, data, features, assumptions, limitations, metrics, fairness report, artefact hash); status DRAFT until approved by MRM; only APPROVED models serve. While DEC-19 is blocked (DEC-47): churn and needs are served by the deterministic rule baseline (registry kind `RULESET`, labelled "non-model" per NYC-07); ML runs only in SHADOW on synthetic data, outputs logged and never shown | [DOC p.38, p.71]; file 13 F14 | `CTL-MODEL-APPROVED-ONLY` | MRM / DEV |
| CMP-07 | Model monitoring: drift (PSI), calibration and stability reports per run; outcomes analysis; breach -> automatic fallback to rules baseline/previous version and alert | [DOC p.32, p.38, p.71]; file 13 F14 | `CTL-MODEL-APPROVED-ONLY` | DEV / MRM |
| CMP-08 | Start with rules and transparent baselines; more complex models only with documented lift and explainability | Brief [DOC p.71] | - | DEV / MRM |
| CMP-09 | Fairness: excluded-feature list covering the union of protected categories identified in file 13 (federal, NY Exec. Law 296-a, NYCHRL) and known proxies (postcode/ZIP, remittance corridors, language, childcare spend); outcome parity reports on surfacing and suppression rates by group proxy and segment; differences require documented justification. **Method spec (gap G-5) [REC], to be approved under the fairness-proxy decision (§4, formerly 08-D6):** (a) metrics: surfacing rate, suppression rate, credit-referral rate and retention-offer rate per group and segment; impact ratio (group rate / most-favoured group rate, LL144-style format, file 13 C01 best practice); (b) alert threshold: impact ratio below 0.8 or above 1.25 for any group, or a statistically significant difference (two-proportion test, p < 0.05) [MRM sets final values]; (c) minimum group size 100 customers per cell, smaller cells reported as "insufficient" and never back-filled; (d) group proxy source: only the restricted store (CMP-10) holding host-supplied or legally approved test attributes; never inferred by us from transactions or names (`EXCL-FORBIDDEN-INFERENCE`); (e) cadence: every batch run and before each model or pack activation (simulator); (f) escalation: breach -> `CTL-FAIRNESS-MONITOR` alert to MRM and COMP, model/pack activation blocked until a documented justification or fix; (g) reports stored as audit records with model, pack and data versions | [DOC p.30, p.38, p.43, p.81]; file 13 F06, S06, C02 (C01 best-practice format) [LEGAL-CONFIRM list contents] | `CTL-SENSITIVE-EXCLUDE`, `CTL-PROXY-REVIEW`, `CTL-FAIRNESS-MONITOR` | DEV / MRM / LEG |
| CMP-10 | Sensitive attributes used for fairness testing held in a separate restricted store that models cannot read | [DOC p.38] | `CTL-SENSITIVE-EXCLUDE` | DEV / SEC |
| CMP-11 | Explainability: every prediction and recommendation returns fixed-library reason codes (fixed reason codes support explainability; outputs must not feed credit decisions - any such use needs a new manager + legal approval and is out of scope, file 15 IC7); LLM only words them; reason-code library versioned, owner COMP | [DOC p.11, p.73]; file 13 F06 | `CTL-EXPLANATION` | DEV / COMP |
| CMP-12 | Replay: stored inputs reference + versions (incl. policy pack versions and `as_of`) reproduce identical deterministic output and policy outcome | [DOC p.13, p.50]; file 14 §8.2 | `CTL-AUDIT-LOG` | DEV |
| CMP-13 | Maker-checker for reference data, catalogue, thresholds **and policy packs** (maker != approver; only legal/compliance sets `applicability_status`) | [DOC p.9, p.63]; file 14 §9 | - | DEV / MGR / LEG |
| CMP-14 | Overrides of a recommendation by staff recorded with mandatory reason; complaint capture; human review workflow for items flagged `requires_human_review` (states QUEUED / IN_REVIEW / CLEARED_TO_SHOW / REJECTED / ESCALATED / EXPIRED / CANCELLED, mandatory reason on reject/escalate, SLA [MANAGER], every transition audited; file 07 `review_item`, file 15 §2 outcome endpoints) (gap G-1) [REC] | [DOC p.43]; file 13 F05 | `CTL-AUDIT-LOG` | DEV / HOST |
| CMP-15 | Vulnerability flags (host-supplied, never inferred by us) suppress commercial recommendations and route to human handling | [DOC p.17, p.54] [REC]; file 13 F05, S08 | `CTL-VULNERABILITY-ROUTE` | DEV / HOST / LEG |
| CMP-16 | Retention outreach suggestions respect host contact limits and quiet hours; support-first actions (budgeting help) ranked before commercial offers where financial-stress signals exist; no dark-pattern retention nudges | [DOC p.41, p.69] [REC]; file 13 F05, F11, S08, S11 | `CTL-CONTACT-LIMIT`, `CTL-MARKETING-SUPPRESS` | DEV / LEG |
| CMP-17 | **Jurisdiction policy packs** (`PACK-GLOBAL-BASELINE`, `PACK-US`, `PACK-US-NY`, `PACK-US-NYC`) hold regulatory and protection controls as versioned, signed, legal-approved data; market reference data (currency, catalogue, terms refs) held separately; no jurisdiction branching in code; `PENDING_LEGAL_REVIEW` enforced as BLOCK; unknown/unsupported jurisdiction -> restrictive default | User; file 14 §1, §4, §10 | all; `CTL-DEFAULT-DENY` | DEV / LEG |
| CMP-18 | Customer-facing explanation languages per approved policy (PDF uses English/Spanish for US [DOC p.13, p.73]); plain-language and LEP considerations | [DOC p.13, p.73] [MANAGER]; file 13 S08 | `CTL-EXPLANATION` | MGR / LEG |
| CMP-19 | Model documentation pack aligned to the bank's model-risk standard (SR 26-2 vocabulary: purpose, materiality, inherent risk, effective challenge, per file 13 F14), format confirmed by MRM | [DOC p.71] [UNKNOWN]; file 13 F14 | - | MRM / DEV |

### 2.4 Controls implied by the NYC applicability matrix (new)

Each row is a design requirement derived from file 13. The condition column states when it matters; where a row depends on the bank's status, it is built as a capability and switched on by policy only once legal confirms.

| ID | Requirement | File 13 rows (applicability as recorded there) | Condition | CTL (file 14) | Owner |
|---|---|---|---|---|---|
| NYC-01 | **23 NYCRR 500-style controls, only if the operating entity is a covered entity:** MFA for all access (via host IdP); asset-inventory entries for every component incl. models and LLM; privileged-access review; encryption of NPI or CISO-approved compensating controls; audit trail; vulnerability management with faster SLAs; incident-response/BCDR hooks; third-party service provider controls; secure disposal; inputs to annual certification | S01 (Likely **if covered entity**; [VERIFIED-SOURCE]; BINDING LAW for covered entities, covered-entity status UNRESOLVED APPLICABILITY); S02 (Likely if S01 applies; SUPERVISORY GUIDANCE) | Q-C1, Q-C7, Q-C8 answered by LEG [LEGAL-CONFIRM] | `CTL-AUDIT-LOG`, `CTL-RETENTION` + build controls | SEC / DEV |
| NYC-02 | **AI-specific cyber controls (if covered entity):** AI/LLM component inventory with data classification; prompt/response logging with NPI redaction; anomalous-query monitoring on the assistant; vendor clause "no training on bank data, no retention"; human review of AI-generated code in the SDLC; AI entry in the cyber risk assessment | S02 ([VERIFIED-SOURCE] 2024, 2025, May 2026 and Sep 2026 letters; supervisory guidance, no new requirements) | Same as NYC-01 | `CTL-LLM-GUARDRAIL`, `CTL-AUDIT-LOG` | SEC / DEV |
| NYC-03 | **Under-18 safeguards:** locked minor block (CMP-01/02) using the two-tier list in file 14 §13.3; `AGE_UNKNOWN` treated as minor for commercial activities; under-13 records rejected or quarantined at ingest; host age-flag honoured; any 16-17 educational insight tagged with a recorded legal basis ("strictly necessary" purpose or consent, grantor recorded); no model training on minors' data; no sharing of minors' data for non-permitted purposes; no algorithmic feed and no notifications to minors 00:00-06:00; re-classification at 18th birthday with fresh consent check | S12 (Possible to Likely; NY Child Data Protection Act - BINDING LAW, reach to the bank UNRESOLVED APPLICABILITY - **open legal question**, file 13 §6 Q4), F12 (Unlikely for MVP), S13 (Unlikely), F07 (under-21 credit) | Always on (baseline, locked against packs and host overlays); tier 1 (commercial activities, credit referral, model training) can never be re-enabled; only tier 2 (named non-commercial educational uses) can be relaxed, and only by a signed `MINOR-APPROVAL-*` record [LEGAL-CONFIRM] | `CTL-MINOR-PROTECT`, `CTL-CONSENT-PURPOSE`, `CTL-CONTACT-LIMIT` | DEV / LEG / PRIV |
| NYC-04 | **No credit decisions and credit-adjacent caution:** CMP-03/05 plus: log every credit-product suppression with reason; under-21 card candidates flagged "requires independent ability to pay or 21+ co-signer, assessed by credit platform"; never imply pre-approval; block "tangible inducement" campaign tags for student segments; overdraft/BNPL/insurance out of catalogue unless legal approves | F06 (Possible; Reg B as amended by 91 FR 21620, in effect 21 Jul 2026), F07 (Possible), F08 (Unlikely), S03 (Unlikely), S06 (Possible), S16 (Unlikely/Possible), C02 (Possible; NYCHRL 8-107(4) and 8-107(17); 8-107(5)(d) only for housing/land/commercial-space financing) | Always on | `CTL-CREDIT-REFERRAL-ONLY`, `CTL-ADVERSE-ACTION-HANDOFF` | DEV / COMP |
| NYC-05 | **Sensitive-attribute and proxy exclusion:** `EXCL-SENSITIVE`, `EXCL-FORBIDDEN-INFERENCE`, `EXCL-PROXY`, `EXCL-PROTECTION-ONLY` lists (file 14 §13.2) applied at feature build, ranking and model training; model registry declares feature list; CI checks it against the union of applicable lists; categories called out in file 13 (citizenship/immigration status, presence of children, pregnancy, marital/partnership status, military/uniformed-service status, height, weight) included pending legal review; no "aggregate statistics about groups" features (e.g. neighbourhood demographics) | S06 (Possible), C02 (Possible; [VERIFIED-SOURCE] CCHR text: 8-107(4) public accommodations incl. advertising, 8-107(17) disparate impact; 8-107(5)(d) limited to housing/land/commercial-space financing), F06 (Possible), S14 (not law; vetoed) | Always on; list contents [LEGAL-CONFIRM] (file 14 §16 J6) | `CTL-SENSITIVE-EXCLUDE`, `CTL-PROXY-REVIEW`, `CTL-NO-INFERENCE` | DEV / LEG / MRM |
| NYC-06 | **Explainability and truthful content:** fixed reason codes; explanation text never overstates benefit or certainty; catalogue fees shown exactly (never LLM-generated); pre-release content review workflow for explanation templates; AI-assisted marking where policy requires | F05 (Likely), S08 (Likely; FAIR Act effective 17 Feb 2026 [VERIFIED-SOURCE]), F09, C05 (best practice) | Always on | `CTL-EXPLANATION`, `CTL-AI-DISCLOSURE`, `CTL-HUMAN-REVIEW` | DEV / COMP |
| NYC-07 | **AI/ML governance:** ML models (churn, needs, ranking) governed per CMP-06/07/19; rule-based components labelled "non-model" so the bank can classify them; **separate GenAI governance track** for the LLM orchestrator (prompt-template and tool registry, evaluation and red-team suite, version pinning, output validator, change control, template fallback), a RECOMMENDED CONTROL and bank policy decision: file 13 F14 records that SR 26-2 / OCC 2026-13 state generative and agentic AI models "are not within the scope of this guidance" and leave their governance to the bank's own risk management and governance practices; vendor-model documentation if any third-party model is used | F14 (Possible to Likely; [VERIFIED-SOURCE]; SUPERVISORY GUIDANCE), F15 (Likely; SUPERVISORY GUIDANCE), S02 (SUPERVISORY GUIDANCE), C01 (best-practice audit format), C06 (no NYC AI law identified; [UNVERIFIED]) | Size and MRM policy dependent (Q-A1, Q-A2, Q-C6) [LEGAL-CONFIRM] | `CTL-MODEL-APPROVED-ONLY`, `CTL-LLM-GUARDRAIL`, `CTL-FAIRNESS-MONITOR` | MRM / DEV / SEC |
| NYC-08 | **Channel and outreach controls:** per-customer `call_consent`, `sms_consent`, `dnc`, email opt-out from host; suggestions needing a channel without consent are suppressed; quiet hours in customer local time; revocation invalidates queued suggestions within a configurable window | F10 (Possible), F11 (Possible; FCC rule 91 FR 67214 effective 25 Nov 2026, text not read), S11 (Possible) | Only if outputs feed outreach (Q-P6) | `CTL-CHANNEL-ALLOW`, `CTL-CONTACT-LIMIT`, `CTL-MARKETING-SUPPRESS` | DEV / COMP |
| NYC-09 | **No personalised pricing:** subsystem never computes personalised prices or rates; a `price_personalised` flag and disclosure hook exist only if ever needed | S09 (Unlikely; exemption for GLBA-covered institutions per file 13) | Always on | - (design rule) | DEV |
| NYC-10 | **AML boundary:** no SAR, case or AML-alert data ingested; any host "restricted" flag used only to suppress outreach, never as a feature or in explanations | F13 (Unlikely; boundary; [UNVERIFIED]) | Always on | `CTL-MARKETING-SUPPRESS` | DEV / BSA |
| NYC-11 | **Breach readiness:** minimise "private information" held; classification tags; breach-scoping query by NY residency; forensic export to support notification timelines owned by the bank | S04 (Likely; deemed-compliance route), S05 (Likely), C07 (none identified) | Real-data use | `CTL-AUDIT-LOG` | SEC / PRIV |

### 2.5 Two-track requirements (2026-11-10; file 21; file 12 DEC-50..DEC-64)

Design requirements from the developer's two-track scope decision (DEC-50): Track A "Conventional" = NYC + London (cells `A-US`, `A-UK`), Track B "Islamic" = Islamic bank, UAE onshore [ASSUME]. They are design choices, not statements that any law applies; the related legal questions are [LEGAL-CONFIRM] (UK: file 19, written 2026-11-10; UAE and Sharia: file 20, written 2026-11-10) and Sharia content is [SHARIA-CONFIRM].

| ID | Requirement | Source | CTL (file 14 / file 21) | Owner |
|---|---|---|---|---|
| TT-01 | **Cell isolation (Track A):** `A-US` and `A-UK` each have their own database, subject-key store, audit and decision logs, object store, secrets/KMS scope and region; no cross-cell join, replication or backup restore into the other cell; only aggregate, non-personal figures (portfolio, small cells suppressed) cross cells; a residence/booking split across cells resolves to `UNKNOWN` (deny commercial) | DEC-52; UK GDPR international-transfer question [LEGAL-CONFIRM] | `CTL-DATA-RESIDENCY`, `CTL-DEFAULT-DENY` | SEC / PRIV / DEV |
| TT-02 | **Track B isolation:** separate deployment on bank premises with its own database, keys (deployment and demo-class), IdP realm, model registry, catalogue, packs, templates, logs and backups; no Track A data, model, feature or training set (synthetic included) ever enters Track B and vice versa; bootstrap refuses any artefact whose `track` tag differs from the profile | DEC-53; UAE manual ch. 33.2 (bank document, not a legal conclusion) | `CTL-DATA-RESIDENCY` + profile check | SEC / DEV |
| TT-03 | **LLM hosting per track:** template-only in MVP (DEC-06). If ever enabled: Track B no hosted or external LLM, bank on-prem model only, customer data, prompts and logs stay in the UAE (file 20 IQ3; third-party hosting or a vendor foundation model treated as not allowed until IQ3 is answered); Track A in-region per cell, no cross-region prompts without a transfer assessment. Enablement stays a no-go trigger until its blocking item is resolved | DEC-58 | `CTL-LLM-GUARDRAIL` | SEC / MGR |
| TT-04 | **No religion inference (both tracks):** religion or religious observance is never inferred from merchant categories, donations, Islamic product holdings or the institution type; a Track B customer is not assumed to be Muslim. Added to `EXCL-FORBIDDEN-INFERENCE` | DEC-54; extends NYC-05 | `CTL-NO-INFERENCE` | DEV / LEG |
| TT-05 | **Prohibited-category handling (Track B):** categories on the Sharia list are never offered, never in rewards or partner content, never a ranking/targeting feature, never shown with judgemental wording; neutral budgeting labels only | DEC-54 [SHARIA-CONFIRM] | `CTL-PROHIBITED-CATEGORY` | DEV / Sharia |
| TT-06 | **Sharia approval and separation of duties:** new role `sharia_approver` (maps to the Sharia Compliance function / ISSC; interim role per DEC-63); maker-checker; no identity holds both `policy_author` and `sharia_approver`, nor `sharia_approver` and `policy_release`; Track B packs need legal and Sharia approval before `APPROVED`; artefacts not `SHARIA_APPROVED` are BLOCK (`SHARIA_REVIEW_PENDING`) | DEC-55, DEC-63; extends SEC-02, SEC-19 | `CTL-SHARIA-APPROVAL` | SEC / COMP / Sharia |
| TT-07 | **Terminology and bilingual explanations (Track B):** Arabic and English templates, each with translation review and Sharia terminology review; output validator blocks prohibited terms and falls back to an approved template; numbers verified in both languages | DEC-54, DEC-59; extends CMP-18 | `CTL-TERMINOLOGY`, `CTL-EXPLANATION` | COMP / Sharia / DEV |
| TT-08 | **Age threshold per jurisdiction:** config map (`US` 18, `GB` 18, `AE` [ASSUME 21] restrictive default) [LEGAL-CONFIRM]; below threshold or unknown = protected; the two-tier list (file 14 §13.3) is unchanged and Tier 1 is never relaxed | DEC-60; extends CMP-01, NYC-03 | `CTL-MINOR-PROTECT` | LEG / DEV |
| TT-09 | **Profile integrity:** the deployment profile is signed and read-only; no request field can set or override track or institution type; profile, track, institution type and cell are written to every decision and audit record | DEC-51, DEC-61 | `CTL-AUDIT-LOG` | DEV / SEC |
| TT-10 | **Track A human review on possible significant decisions:** pending file 19 UQ4 (rows U02, U07) [LEGAL-CONFIRM], offer suppression applies immediately and creates a `review_item` for staff confirmation; every retention action except `RA_NO_ACTION` needs human review before it is shown or acted on; vulnerability routing always ends with a person (both Track A cells) | DEC-66 | `CTL-HUMAN-REVIEW`; `CTL-VULNERABILITY-ROUTE` | COMP |
| TT-11 | **Track B contact caps:** AE pack `CTL-CONTACT-LIMIT` with telemarketing parameters (consent by purpose x channel x product type x contact method x language; frequency caps; DNCR check as a host obligation; file 20 row E08 [LEGAL-CONFIRM]); until file 20 IQ4 is answered, in-app, push and digital messages count as telemarketing | DEC-67 | `CTL-CONTACT-LIMIT`; `CTL-CONSENT-PURPOSE`; `CTL-CHANNEL-ALLOW` | COMP / LEG |
| TT-12 | **UK entity type unknown:** no UK operating-entity type or UK data controller is assumed (file 19 UQ1, Q-U2); every GB pack stays PENDING_LEGAL_REVIEW = BLOCK; `A-UK` built as a separate controller's cell | DEC-65 (BLOCKED-EXTERNAL) | `CTL-DEFAULT-DENY` | MGR / LEG |

## 3. Regulatory references (pointers to file 13; no requirement asserted here)

Applicability and verification values are copied from file 13 §3 as **planning estimates** at retrieval date 2026-11-09 (after the file 13 re-verification pass). They are not legal conclusions; each requires confirmation by the bank's legal and compliance teams [LEGAL-CONFIRM]. **File 13 is the sole owner of these row IDs and their content**; the tables below are pointers only and must be refreshed from file 13 if it changes (R-08). Use file 13 for sources, deciding conditions and questions. "Class" = legal nature of the source (see legend at top); in every row the reach to this bank is **UNRESOLVED APPLICABILITY** until legal answers file 13 §2.

### 3.1 Federal

| File 13 row | Framework (short name) | Class | Applicability (file 13) | Verification (file 13) | Requirements here |
|---|---|---|---|---|---|
| F01 | GLBA privacy / Reg P | BINDING LAW | Likely | VERIFIED-SOURCE (partial scope) | PRV-02/03/04/09/13 |
| F02 | GLBA safeguards - Interagency Guidelines | BINDING LAW | Likely (insured bank) | VERIFIED-SOURCE | SEC-02/05/06/07/20, PRV-07, CMP-13 |
| F03 | FTC Safeguards Rule | BINDING LAW | Unlikely for a bank; Possible for nonbank operator | PARTIAL | SEC set designed for both |
| F04 | FCRA / Reg V affiliate marketing | BINDING LAW | Possible | VERIFIED-SOURCE | PRV-04, PRV-11, CMP-03 (IC7 label) |
| F05 | CFPA UDAAP | BINDING LAW | Likely | VERIFIED-SOURCE | CMP-04/14/15/16, NYC-06 |
| F06 | ECOA / Reg B (2026 amendments, 91 FR 21620, in effect 21 Jul 2026, under challenge in NFHA v. CFPB; circulars withdrawn) | BINDING LAW (for creditors); circular withdrawals = guidance | Possible | VERIFIED-SOURCE (litigation status UNVERIFIED) | CMP-03/05/09/11, NYC-04/05 |
| F07 | TILA / Reg Z / CARD Act | BINDING LAW | Possible (if credit recommended) | VERIFIED-SOURCE | NYC-04 |
| F08 | EFTA / Reg E | BINDING LAW | Unlikely | UNVERIFIED | NYC-04 |
| F09 | FTC Act s.5 | BINDING LAW | Unlikely for bank; Possible for nonbank | PARTIAL | NYC-06 |
| F10 | CAN-SPAM | BINDING LAW | Possible | VERIFIED-SOURCE | PRV-09, NYC-08 |
| F11 | TCPA | BINDING LAW | Possible | VERIFIED-SOURCE (Oct 2026 FCC rule 91 FR 67214, effective 25 Nov 2026: metadata verified, text not read) | PRV-09, NYC-08 |
| F12 | COPPA (amended 2025) | BINDING LAW | Unlikely for MVP | PARTIAL | NYC-03 |
| F13 | BSA/AML (boundary) | BINDING LAW | Unlikely | UNVERIFIED | NYC-10 |
| F14 | Model risk: SR 26-2 / OCC 2026-13 / FDIC FIL-15-2026 (supervisory guidance; supersedes SR 11-7 / OCC 2011-12); OCC 2025-26 | SUPERVISORY GUIDANCE ("expected to be most relevant" over $30bn; not a legal threshold); separate GenAI track = RECOMMENDED CONTROL | Possible to Likely | VERIFIED-SOURCE | CMP-06/07/19, NYC-07 |
| F15 | Third-party risk guidance (OCC 2023-17) | SUPERVISORY GUIDANCE | Likely if vendor/third-party components | VERIFIED-SOURCE | SEC-12/15, PRV-13, NYC-07 |
| F16 | CFPA s.1033 | BINDING LAW (status in flux) | Unlikely / Unknown | UNVERIFIED | PRV-11 |
| F17 | Credit-union equivalents (NCUA) | BINDING LAW | Unknown (only if credit union) | UNVERIFIED | Same as F01/F02 |
| F18 | CFPB posture 2025-2026 (context) | Context (guidance withdrawals) | Context row | PARTIAL | Do not drop controls because guidance was withdrawn |

### 3.2 New York State

| File 13 row | Framework (short name) | Class | Applicability (file 13) | Verification (file 13) | Requirements here |
|---|---|---|---|---|---|
| S01 | 23 NYCRR Part 500 | BINDING LAW (covered entities) | Likely **if covered entity** | VERIFIED-SOURCE | NYC-01, SEC-20 |
| S02 | DFS AI / cyber industry letters | SUPERVISORY GUIDANCE (no new requirements) | Likely if S01 applies | VERIFIED-SOURCE (2024, 2025, May 2026 and Sep 2026 letters) | NYC-02, SEC-15, SEC-21 |
| S03 | DFS Insurance Circular Letter 7 (2024) | SUPERVISORY GUIDANCE | Unlikely | UNVERIFIED | NYC-04 (insurance out of catalogue) |
| S04 | SHIELD Act data security | BINDING LAW | Likely (deemed-compliance route) | VERIFIED-SOURCE | NYC-11 |
| S05 | NY breach notification (GBL 899-aa) | BINDING LAW | Likely | VERIFIED-SOURCE | PRV-12, NYC-11 |
| S06 | Exec. Law 296-a (credit discrimination) | BINDING LAW | Possible | VERIFIED-SOURCE | CMP-09, NYC-04/05 |
| S07 | NY Human Rights Law / Banking Law provisions | BINDING LAW (provisions not identified) | Unknown | UNVERIFIED | CMP-09 |
| S08 | GBL 349 as amended (FAIR Business Practices Act, effective 17 Feb 2026) | BINDING LAW | Likely | VERIFIED-SOURCE | CMP-16/18, NYC-06 |
| S09 | GBL 349-a algorithmic pricing disclosure | BINDING LAW | Unlikely | VERIFIED-SOURCE | NYC-09 |
| S10 | GBL 399-ddd SSN confidentiality | BINDING LAW | Possible | VERIFIED-SOURCE | PRV-02 |
| S11 | GBL 399-z telemarketing | BINDING LAW | Possible | VERIFIED-SOURCE | NYC-08 |
| S12 | NY Child Data Protection Act | BINDING LAW | Possible to Likely (open legal question) | VERIFIED-SOURCE (899-ee/ff/jj/kk/ll read, no GLBA exemption found; 899-gg/hh/ii/mm not read) | CMP-01/02, NYC-03 |
| S13 | SAFE for Kids Act | BINDING LAW (from 25 Jan 2027) | Unlikely | VERIFIED-SOURCE (press release) | NYC-03 |
| S14 | NY Health Information Privacy Act | Not law (vetoed) | Not law (vetoed) | VERIFIED-SOURCE | PRV-06 (kept as RECOMMENDED CONTROL) |
| S15 | New York Privacy Act (bill) | Not law (pending bill) | Not law (pending) | VERIFIED-SOURCE | PRV-08, PRV-10 |
| S16 | NY BNPL Act; DFS overdraft proposals | BINDING LAW (BNPL Act); proposals not final | Unlikely / Possible | PARTIAL | NYC-04 |
| S17 | "Stop Hidden Loan Fees" (as named in brief) | UNRESOLVED (no law of that name found; FAIR Act closest verified match) | Unknown - not identified | UNVERIFIED | None until identified |

### 3.3 New York City

| File 13 row | Framework (short name) | Class | Applicability (file 13) | Verification (file 13) | Requirements here |
|---|---|---|---|---|---|
| C01 | Local Law 144 (AEDT) | BINDING LAW for employers only; used here as best-practice format (RECOMMENDED CONTROL) | Unlikely / not applicable; best-practice template only | VERIFIED-SOURCE | CMP-09 / NYC-07 (fairness report format) |
| C02 | NYCHRL public accommodations (8-107(4), incl. advertising) and disparate impact (8-107(17)); 8-107(5)(d) only for housing, land or commercial-space financing | BINDING LAW (local); reach to bank and national-bank preemption = UNRESOLVED APPLICABILITY | Possible | VERIFIED-SOURCE (CCHR text) | CMP-09, NYC-04/05 |
| C03 | NYCHRL credit history in employment | BINDING LAW (employment only) | Not applicable | VERIFIED-SOURCE | No customer insight to HR processes |
| C04 | Biometric Identifier Information law | BINDING LAW | Unlikely | VERIFIED-SOURCE | PRV-02 (no biometrics) |
| C05 | DCWP consumer rules (click-to-cancel; junk fees) | BINDING LAW / proposed rule | Unlikely | UNVERIFIED | NYC-06 (best practice) |
| C06 | NYC AI-specific laws beyond LL 144 | None identified | None identified | UNVERIFIED (absence claim) | NYC-07 |
| C07 | NYC privacy / breach ordinances | None identified (state law preempts local breach rules) | None identified | PARTIAL | NYC-11 |

### 3.4 Outside MVP scope (future expansion)

**Update 2026-11-10 (DEC-50, DEC-13 superseded):** UK (London) is now **in scope** as the `A-UK` cell of Track A; its regimes are assessed in `19-uk-london-regulatory-applicability-matrix.md` (written 2026-11-10), every row [LEGAL-CONFIRM] and every GB pack PENDING_LEGAL_REVIEW = BLOCK. The UAE Islamic bank is **Track B**; UAE regulatory and Sharia governance points are assessed in `20-islamic-bank-uae-regulatory-and-sharia-matrix.md` (written 2026-11-10) [LEGAL-CONFIRM] [SHARIA-CONFIRM]. No regime named below is stated to apply.

| Item | Status | Note |
|---|---|---|
| Other US states (e.g. Colorado and California AI rules named in the PDF [DOC p.54, p.71]) | Out of MVP scope (file 13 §7) | Added later as `PACK-US-<state>` after a new applicability matrix [LEGAL-CONFIRM] |
| UK regimes named in the PDF (UK GDPR incl. Arts 22A-22D, FCA Consumer Duty, Equality Act 2010, Consumer Credit Act s50, CONC, PRA SS1/23, SS2/21, electronic marketing rules) [DOC p.4, p.16-18, p.54, p.67, p.71] | ~~Future expansion~~ **In scope from 2026-11-10 (Track A, `A-UK` cell)** | Matrix: file 19 (written 2026-11-10); packs `PACK-GB`, `PACK-GB-ENG`, `PACK-GB-ENG-LDN`; separate data cell for residency (DEC-52) [LEGAL-CONFIRM] |
| SOC 2 Type II, ISO 27001 (assurance frameworks, not law) [DOC p.40, p.74] | Applies to platform evidence | Whether subsystem is in the platform's audit scope - [MANAGER] |

## 4. Decisions needed

File 12 is the single decision register (DEC-01..). The IDs below are this file's original local IDs, kept so that existing references resolve; each is cross-referenced to file 12, where the DEC ID is assigned (D-03).

| # | Decision | Owner | Register |
|---|---|---|---|
| D1 | Approve MVP = adults 18+ only; minors path disabled until legal sign-off | MGR / LEG | see file 12 (formerly 08-D1); DEC ID assigned in file 12 |
| D2 | Consent purposes taxonomy (INSIGHT vs MARKETING vs PERSONALISATION vs channel/affiliate/MINOR_EDU) and grantor model | LEG / HOST | see file 12 (formerly 08-D2); DEC ID assigned in file 12 |
| D3 | Whether our models fall under the bank model-risk policy (post-SR 26-2) and materiality tier; which framework governs the LLM orchestrator (separate GenAI track is a recommended control, bank policy decision) | MRM | see file 12 (formerly 08-D3); DEC ID assigned in file 12 |
| D4 | LLM runtime allowed for standalone demo and for plugin mode | MGR / SEC | see file 12 (formerly 08-D4); DEC ID assigned in file 12 |
| D5 | Retention periods per entity (values for `CTL-RETENTION`) | LEG | see file 12 (formerly 08-D5); DEC ID assigned in file 12 |
| D6 | Fairness group proxies permitted for testing and where they are stored; approval of the CMP-09 method spec | LEG / MRM | see file 12 (formerly 08-D6); DEC ID assigned in file 12 |
| D7 | Charter and regulatory status answers (file 13 Q-C1 to Q-C8), incl. whether the operating entity is a 23 NYCRR 500 covered entity | LEG / SEC | see file 12 (formerly 08-D7); DEC ID assigned in file 12 |
| D8 | NY Child Data Protection Act position for the teen journey (file 13 §6 Q4) | LEG / PRIV | see file 12 (formerly 08-D8); DEC ID assigned in file 12 |
| D9 | Contents of exclusion, forbidden-inference and proxy lists for US / NY / NYC (file 14 §16 J6) | LEG / MRM | see file 12 (formerly 08-D9); DEC ID assigned in file 12 |
| D10 | Default for unknown/unsupported jurisdiction (deny all vs insight-only) (file 14 §16 J4) | MGR / LEG | see file 12 (formerly 08-D10); DEC ID assigned in file 12 |
| D11 | Confirm UK/other markets are future expansion, not MVP (file 14 §16 J1) | MGR | see file 12 (formerly 08-D11); DEC ID assigned in file 12 |
| D12 | RPO/RTO targets and restore-test cadence (SEC-20); human-review SLA (CMP-14) | MGR / SEC | new in validation pass (formerly 08-D12); see file 12 DEC-36 (RPO/RTO) and DEC-37 (review SLA) |

## Change log (NYC scope update)

- 2026-11-09: Disclaimer rewritten: legal content now lives in file 13 and is referenced by row ID; scope-update banner (NYC: federal + NY State + NYC; UK/other markets future [MANAGER confirm]); [LEGAL-CONFIRM] label and PRIV owner added.
- §1: US/UK install and Consumer Duty rows re-framed for NYC scope.
- §2: every requirement cross-referenced to the file 14 control catalogue (CTL column) and to file 13 rows; SEC-03/04 now deployment + jurisdiction claims; PRV-07 retention via `CTL-RETENTION`; CMP-17 now jurisdiction policy packs; new SEC-18, PRV-11..13.
- New §2.4 NYC-01..NYC-11: Part 500-style controls only "if covered entity", AI cyber controls, under-18 safeguards (NY CDPA open question), no credit decisions, sensitive-attribute exclusion, explainability, AI/ML governance with separate GenAI track, channel controls, no personalised pricing, AML boundary, breach readiness.
- §3: market regime list replaced by pointers to file 13 rows (F01-F18, S01-S17, C01-C07) with applicability and verification status; Colorado/California and UK regimes moved to §3.4 out-of-scope/future.
- §4: decisions D7-D11 added.

## Change log (validation pass 2026-11-09)

- Header: owner legend adds COMP (compliance) and BSA (BSA/AML officer); owner "CMP" replaced by COMP in NYC-04/06/08 and "BSA Officer" by BSA in NYC-10 (D-24). Legal-point class legend added (BINDING LAW / SUPERVISORY GUIDANCE / RECOMMENDED CONTROL / UNRESOLVED APPLICABILITY).
- §1: affordability row marked as PDF context owned by the credit platform (D-02).
- SEC-02: role `policy_release` added (D-24). New SEC-19 role-to-scope/screen matrix (G-2, D-30), SEC-20 backup/restore/DR (G-3, D-31), SEC-21 observability and security monitoring [REC].
- PRV-08: erasure method via crypto-shredding, file 07 §8.2 (G-4, D-32).
- CMP-01 and NYC-03: aligned to the two-tier minor block list in file 14 §13.3; tier 1 never re-enabled, tier 2 only by `MINOR-APPROVAL-*` record (D-01).
- CMP-03: not-for-credit-decisioning label and host terms (file 15 IC7) (D-11). CMP-05: affordability removed as our gate/input; credit-platform suppression flag only (D-02). CMP-09: fairness method spec (metrics, thresholds, minimum group size, proxy source, cadence, escalation) (G-5, D-33). CMP-11: credit-feed wording replaced (D-36); reason-code library owner COMP. CMP-14: human review workflow states (G-1).
- File 13 "Wording to propagate" items 7-13 applied exactly (NYC-07 GenAI track = RECOMMENDED CONTROL / bank policy decision; F06 Reg B 91 FR 21620; F11 and NYC-08 TCPA 91 FR 67214; F14 FDIC FIL-15-2026 / supervisory guidance; NYC-02 Sep 2026 letter verified; NYC-05 C02 8-107(4)/(17), 8-107(5)(d) housing/land/commercial-space only, height/weight added; NYC-06 FAIR Act 17 Feb 2026). Item 18 applied to NYC-04 and the C02 row.
- §3: "Class" column added to all three tables; verification values refreshed from file 13 after its re-verification pass (S08 and C02 now VERIFIED-SOURCE, S12 sections read); note that file 13 is the sole owner of row IDs (R-08).
- §4: each decision cross-referenced "see file 12 (formerly 08-Dn); DEC ID assigned in file 12" (D-03); new D12 (RPO/RTO, review SLA).

## Change log (readiness fixes 2026-11-09)
- CMP-06: serving rule while DEC-19 is blocked (rule baseline serves; ML SHADOW only; DEC-47). CMP-09 (c) minimum group size 100 confirmed as the single value; file 12 DEC-21 now matches it (DEC-48).

## Change log (two-track scope 2026-11-10)
- New §2.5 TT-01..TT-09 (cell isolation, Track B isolation, LLM hosting per track, religion-inference ban, prohibited categories, `sharia_approver` separation of duties, bilingual terminology, age threshold per jurisdiction, profile integrity). §3.4: UK now in scope (file 19 in progress), UAE/Sharia pointer to file 20 (in progress). No law is stated to apply. Source: file 21, file 12 DEC-50..DEC-64.
- Files 19/20 now cited as written 2026-11-10; TT-03 tightened (Track B no hosted/external LLM, data stays in UAE, file 20 IQ3); TT-10 (DEC-66), TT-11 (DEC-67), TT-12 (DEC-65) added.
