# 03 - Technology Compatibility Report

Subsystem: **Retail Banking AI Intelligence** (customer intelligence, transaction analysis, financial needs, churn, product recommendation, retention).
Status: PLANNING ONLY. No code, no repository and no host stack exist yet.

**Label key:** [DOC p.N] supported by the research PDF (PDF page index) | [REC] recommendation | [ASSUME] assumption | [UNKNOWN] not known | [MANAGER] needs manager approval | [LEGAL-CONFIRM] legal/compliance must confirm.

> **Scope update (2026-11-09, authoritative user requirement):** the initial market is **New York City, USA** (US federal + New York State + New York City only). There are **no 50-state rulebooks** in the MVP. Other US states and international markets are added later as **configurable, versioned jurisdiction policy packs** without rewriting the shared core (design: `14-jurisdiction-policy-layer-design.md`; contracts: `15-integration-contracts.md`; legal research: `13-nyc-regulatory-applicability-matrix.md`). This supersedes the earlier "separate US and UK installs / US and UK configs" framing; UK and other markets are future expansion [MANAGER confirm]. Nothing here is legal advice; applicability of any law depends on the bank's charter, status, products and activities, which are [UNKNOWN] [LEGAL-CONFIRM].

> **Two-track scope update (2026-11-10, authoritative; file 21, file 12 DEC-50..DEC-64):** the NYC-only scope above is superseded. Track A "Conventional" = NYC + London (data cells `A-US`, `A-UK`); Track B "Islamic" = Islamic bank, UAE onshore [ASSUME], separate bank-hosted deployment. The stack is unchanged; the only technology consequences are: PostgreSQL **per data cell** (DEC-52), Track B on bank premises (DEC-53), a signed **deployment-profile loader** at bootstrap (DEC-51), LLM hosting per track (DEC-58) and Arabic/RTL support in the dashboard (DEC-59). UK is no longer future expansion; other US states and other countries still are.

---

## 1. Evidence position (read this first)

| Question from brief | Evidence found | Label |
|---|---|---|
| Host backend language / framework | None. No repo, manifest or tech doc exists. The PDF is a business/research pack, not a technical spec. | [UNKNOWN] |
| Host frontend, DB, auth, messaging, CI, test conventions | None. | [UNKNOWN] |
| Host deployment model | Runs inside the bank's own data centre / private cloud, can run with no internet, updates delivered as signed packages. | [DOC p.40, p.74] |
| Market separation | PDF: separate US and UK installations; no data crosses between them [DOC p.3, p.40, p.54, p.74]. **Superseded for this subsystem** by the NYC-first requirement: one deployment scoped to `US-NY-NYC`; jurisdictions are policy packs; separate deployments only where data residency requires it (future international) [LEGAL-CONFIRM]. | User; [DOC p.54]; file 14 §0 |
| Host integration styles the platform expects | Real-time scoring API (target under 300 ms), nightly batch from bank data platform, event streaming of payments/account events, secure file drop for pilots. Bank systems remain system of record and send all customer communications. | [DOC p.53] |
| LLM hosting constraint | LLM runs inside the bank: bank-approved model or open model of US or European origin; if unavailable, standard templates are used. | [DOC p.39-40, p.73] |
| Security evidence expected of the platform | SOC 2 Type II, ISO 27001 aligned, annual pen test, SBOM, code escrow. | [DOC p.40, p.74] |
| Model types the platform anticipates | Constrained gradient boosting, explainable glass-box, traditional scorecard. | [DOC p.71] |
| Legal/regulatory applicability | Researched in file 13 (federal F01-F18, NY State S01-S17, NYC C01-C07). Every applicability call there is conditional on the bank's charter (Q-C1 to Q-C8) and is [LEGAL-CONFIRM]. | file 13 |

**Consequence:** every stack choice below is *conditional*. The design rule that makes this safe is **contracts over language coupling**: the host integrates with us only through versioned HTTP/JSON contracts (OpenAPI + JSON Schema) and optional event/file contracts, never by importing our code. If the host later picks Java/.NET/Node, nothing in the contract changes. [REC] A second rule makes jurisdiction change safe: **legal rules are data, not code** (file 14 PG-1). [REC]

---

## 2. Project needs that drive the choices

| Need | Source | Label |
|---|---|---|
| Deterministic money calculations (exact decimals, no float) | Brief rule; PDF shows all sums rule-calculated | [DOC p.10, p.18] [REC] |
| Transparent baseline ML first (rules, logistic regression, clustering); GBM only if justified | Brief; PDF model types | [DOC p.71] [REC] |
| Explainability per prediction / recommendation | PDF principle "every decision explainable and replayable" | [DOC p.50] |
| LLM only for orchestration/explanation; numbers in LLM text verified against calculation; template fallback | PDF AI writer rules | [DOC p.40, p.73] |
| Two modes over one core; host stack unknown | Brief | [REC] |
| Runs offline/inside bank | PDF | [DOC p.40, p.74] |
| **Jurisdiction-aware behaviour**: NYC (federal + NY State + NYC) in MVP; other states/countries added as versioned, approved policy packs with no core rewrite | User (authoritative); file 14 | User |
| **Policy decisions replayable and auditable** (versioned, effective-dated, maker-checker approved, signed) | User; [DOC p.9, p.13, p.63]; file 14 PG-3 | User |
| **Fail-safe on legal uncertainty** (`PENDING_LEGAL_REVIEW` -> restrictive default) | User; file 14 PG-4 | User |
| Small team (one developer for MVP), learning domain | Brief | [ASSUME] |
| Demo to manager quickly; synthetic data only | Brief | [REC] |

---

## 3. Layer-by-layer evaluation

Scoring: Fit = fit to needs above; Cx = complexity added (Low/Med/High); Host risk = chance of clashing with an unknown host stack.

### 3.1 Language / runtime

| Candidate | Fit | Cx | Host risk | Verdict |
|---|---|---|---|---|
| **Python 3.12** | Best ML/analytics ecosystem; `decimal` for money; fast to demo | Low | Low if exposed only via HTTP | **Recommended** [REC] |
| Java 21 / Kotlin (Spring) | Common in banks; weaker ML iteration | Med | Low | Reconsider only if host mandates JVM for all services [MANAGER] |
| TypeScript / Node | Good for API+UI; weak ML | Med | Low | Rejected for core |

What it does: implements the shared intelligence core and API. Main project uses it? [UNKNOWN]. Compatible? Yes via HTTP contracts. Unneeded complexity? No.

### 3.2 API framework

| Candidate | Notes | Verdict |
|---|---|---|
| **FastAPI + Pydantic v2** | Generates OpenAPI 3.1 + JSON Schema from typed models, so contracts are produced from one source; async; light | **Recommended** [REC] |
| Django + DRF | Batteries (admin, ORM) we do not need; heavier | Rejected |
| Flask | Contracts must be hand-maintained | Rejected |

Contract style: REST/JSON with OpenAPI as the published artefact. gRPC rejected for MVP (host stack unknown; adds codegen burden). [REC] Every customer-level request carries `jurisdiction_context` and every response carries the policy `decision_id` and pack versions (file 15 IC4), plus the purpose-limitation labels `permitted_use` and `not_for_credit_decisioning: true` (file 15 IC7). [REC]

### 3.3 Operational DB and analytics store

| Candidate | Notes | Verdict |
|---|---|---|
| **PostgreSQL 16** | `NUMERIC` money, row-level security, JSONB for explanations and policy params, mature in banks, runs on-prem | **Recommended system store** [REC] |
| **DuckDB + Parquet** (embedded) | Columnar batch feature computation and synthetic-data generation without a cluster | **Recommended for batch analytics only** [REC] |
| SQLite | Fine for unit tests; no RLS, weak concurrency | Tests only |
| Warehouse (Snowflake/BigQuery) / Spark | Cloud or cluster dependency conflicts with "inside the bank, offline" [DOC p.74] | Rejected |

One DB **per deployment** (MVP: one deployment, `deployment_scope = [US-NY-NYC]`; **2026-11-10: one PostgreSQL database per data cell** - `A-US` and `A-UK` in their own regions, `B-AE` on bank premises; no cross-cell joins, aggregate-only cross-cell views; DEC-52, DEC-53); rows carry `deployment_id` and jurisdiction context rather than a `market_code` - see 07. A separate deployment/DB is created only if a future market needs data residency [LEGAL-CONFIRM] [MANAGER]. Host DB choice [UNKNOWN]: we never share a DB with the host; data crosses only via adapters. [REC]

### 3.4 ML libraries

| Candidate | Use | Verdict |
|---|---|---|
| **scikit-learn** | Logistic regression (churn, needs), k-means / rules for behavioural clusters, calibration, metrics | **Recommended baseline** [REC] |
| LightGBM (monotone constraints) | Only if baseline is beaten on held-out data and explainability holds; aligns with "constrained gradient boosting" [DOC p.71] | Deferred, gated [REC] |
| SHAP | Needed only for GBM; linear models explain via coefficients x feature deltas | Deferred with LightGBM |
| Fairlearn | Group outcome metrics for fairness reports [DOC p.38, p.81]; supports `CTL-FAIRNESS-MONITOR` (file 14 §5) | Recommended (light) [REC] |
| Deep learning (PyTorch etc.) | No data or justification | Rejected |

Transaction categorisation: **rules (MCC + keyword dictionaries held as market reference data, MVP: US) first**, ML text classifier only as fallback with confidence threshold. Category taxonomy must not contain labels on the forbidden-inference list (`EXCL-FORBIDDEN-INFERENCE`, file 14 §13.2). [REC]

Model governance alignment: the non-generative ML (churn, needs, recommendation ranking) is within the principles of **SR 26-2 / OCC Bulletin 2026-13** per file 13 row F14 (which records that it supersedes SR 11-7 and SR 21-8, and that OCC 2026-13 rescinds OCC 2011-12 and 2021-19; FDIC version FIL-15-2026). It is supervisory guidance, "expected to be most relevant" to banks over $30bn (a relevance statement, not a legal threshold); it may also be relevant to smaller banks with significant model risk. Classification: **SUPERVISORY GUIDANCE**; whether and how deeply it reaches this bank is **UNRESOLVED APPLICABILITY** (bank size and MRM policy, Q-A1, Q-C6) [LEGAL-CONFIRM]. Tooling implication: model registry and validation evidence packs, not a new library. [REC]

### 3.5 Feature store

| Option | Verdict |
|---|---|
| Feast / Tecton | Rejected for MVP: one consumer, batch features, no online low-latency feature serving need proven [REC] |
| **Versioned feature tables in PostgreSQL** (`feature_snapshot` with `as_of`, `feature_set_version`) computed by DuckDB batch | **Recommended**; gives point-in-time correctness for leakage prevention without new infrastructure. Feature build drops attributes on the applicable exclusion lists (`CTL-SENSITIVE-EXCLUDE`) and records the feature-list hash [REC] |

Revisit if the host already runs a feature store [UNKNOWN] or real-time features are required [MANAGER].

### 3.6 Orchestration / LLM layer

| Candidate | Notes | Verdict |
|---|---|---|
| **Thin in-house orchestrator** (typed tool registry, JSON-schema structured outputs, deterministic tool calls, output validator, template fallback) behind an `LLMPort` | Small, auditable, provider-agnostic; matches "LLM never decides, numbers verified, template fallback" [DOC p.40, p.73] | **Recommended** [REC] |
| LangChain / LangGraph | Large dependency surface, fast-moving APIs, harder to audit | Rejected for MVP |
| Provider SDK called directly from modules | Couples core to a vendor | Rejected |

LLM adapters: (a) `TemplateOnly` (always available, default in tests), (b) local open-weight model served on-prem, (c) bank-approved endpoint in plugin mode. Model/provider choice is **platform-wide** [MANAGER] because the PDF restricts origin and hosting [DOC p.73]. Standalone demo with synthetic data may use a hosted API only if the manager approves [MANAGER]. **2026-11-10 (DEC-58):** MVP stays template-only in both tracks; if ever enabled, Track B allows only adapter (b) on bank premises (never an external API; UAE manual ch. 33.2) and Track A runs in-region per cell (no cross-region prompts without a transfer assessment [LEGAL-CONFIRM]). Arabic outputs pass `CTL-TERMINOLOGY` (DEC-54) and the number check. Whether the LLM may be used at all for a given activity/jurisdiction is a policy decision (`CTL-LLM-GUARDRAIL{allow_llm}`, file 14 §5); `allow_llm=false` forces templates. [REC]

**Separate LLM governance track.** File 13 row F14 records (SR 26-2 attachment fn. 3, [VERIFIED-SOURCE]) that generative AI and agentic AI models "are not within the scope of this guidance" and that "a banking organization's risk management and governance practices should guide the determination of appropriate governance and controls" for tools not covered (Q-A2). This is supervisory guidance and does not itself require a separate governance regime. Our reading of that wording is not a legal conclusion [LEGAL-CONFIRM]. Recommended control (bank policy decision, flagged for legal, compliance and MRM): the stack should support a separate GenAI governance pack (file 13 §5.6): prompt-template and tool registry with versions, version-pinned model artefacts, output validator (file 08 SEC-14), evaluation and red-team suite, change control, prompt/response logging with NPI redaction (file 13 S02), and template fallback. The NYDFS AI industry letters (file 13 row S02; **SUPERVISORY GUIDANCE**, no new requirements by their own terms) are relevant only **if** the bank is a 23 NYCRR 500 covered entity (row S01: **BINDING LAW** for covered entities; covered-entity status = **UNRESOLVED APPLICABILITY**, Q-C7) [LEGAL-CONFIRM]. Classification of this paragraph: separate LLM track = **RECOMMENDED CONTROL**. [REC]

Batch job orchestration: plain CLI commands run by cron / host scheduler; no Airflow/Celery for MVP. [REC]

### 3.7 Frontend dashboard

| Candidate | Notes | Verdict |
|---|---|---|
| **React + TypeScript + Vite** (thin client of the public API) | Uses exactly the API the host would use, so the demo proves the contract; typed client generated from OpenAPI; common in bank front ends [ASSUME] | **Recommended** [REC] |
| Streamlit | Fastest to build, but tempts direct Python calls that bypass the API and auth; weak RBAC | Rejected (acceptable only for internal model notebooks) |
| Server-rendered (Jinja/HTMX) | Simple, but less reusable | Fallback if UI effort must be minimal [MANAGER] |

**2026-11-10 (DEC-59):** the dashboard supports locales en-US, en-GB, ar-AE and en-AE, right-to-left layout for Arabic (CSS logical properties, `dir` attribute; i18n message catalogues; no new framework), and currency formatting per cell (USD, GBP, AED). In plugin mode the dashboard is **not shipped**; the host renders our results. The standalone dashboard adds a read-only policy-pack and decision-log view and the policy-pack simulator page (file 14 §14). [REC]

### 3.8 Messaging / events

| Candidate | Verdict |
|---|---|
| **None for MVP; transactional outbox table + CloudEvents-shaped JSON schemas defined** (incl. `policy.pack.activated`, `policy.review.required`, `policy.overlay.rejected`, file 15 §9) | **Recommended**: contracts exist, transport deferred [REC] |
| Kafka adapter | Add only when host confirms event streaming [DOC p.53 mentions it for the credit platform] [UNKNOWN] |
| RabbitMQ / Redis streams | Rejected unless host uses them |

### 3.9 Authentication / authorisation

| Candidate | Verdict |
|---|---|
| **OIDC/OAuth2 bearer JWT validation via JWKS + role/scope + jurisdiction claims** (`ri_deployment`, `ri_jurisdictions`, replacing the old `market` claim; file 15 §5), behind an `IdentityPort` | **Recommended** [REC] |
| Standalone: local demo issuer (Keycloak container, or a minimal dev issuer) with demo roles incl. `policy_author`, `policy_approver`, `policy_release` (role-to-scope matrix file 15 §5.1) | Keycloak recommended if laptop resources allow; else dev issuer [REC] |
| Plugin: validate host-issued tokens; service-to-service mTLS where the host requires it | [UNKNOWN] host IdP; [MANAGER] |
| Session cookies / own user DB | Rejected (would duplicate host identity) |

The PDF lists SSO under platform Release 2 [DOC p.70], so identity is a platform concern. MFA for human access is a host IdP responsibility; it is a design requirement (**RECOMMENDED CONTROL**) because 23 NYCRR 500.12 (**BINDING LAW** for covered entities) may apply **if** the bank is a covered entity (file 13 row S01; covered-entity status = **UNRESOLVED APPLICABILITY**) [LEGAL-CONFIRM].

### 3.10 Packaging / deployment

| Candidate | Verdict |
|---|---|
| **Core as an internal Python package (wheel) + one container image for the API/batch; docker-compose for standalone (api, postgres, ui, optional keycloak, optional local LLM)** | **Recommended** [REC] |
| Kubernetes / Helm | Deferred to host decision [MANAGER] |
| Microservice per module | Rejected: no scaling or team-boundary justification (brief forbids unjustified microservices) |
| Separate policy microservice / external policy server | Rejected for MVP: PDP is an in-process library (no network hop), also exposed as `POST /v1/policy/decisions` (file 14 §6) [REC] |
| Signed, offline-installable images + SBOM; **policy packs shipped as separately signed artefacts** (not baked into the image) | Recommended to match platform expectations [DOC p.40, p.74] and allow pack updates without a code release [REC] |

### 3.11 Observability

| Candidate | Verdict |
|---|---|
| **Structured JSON logs (no PII) + OpenTelemetry traces/metrics + Prometheus-format `/metrics`**; correlation ID propagated from host | **Recommended** - vendor neutral, works with whatever the host collects [REC] |
| Vendor APM agent | Rejected (host-dependent) |
| Model monitoring: scheduled drift/stability reports (PSI, calibration) stored as audit records | Recommended [DOC p.32, p.38] [REC] |
| Policy metrics: decisions by outcome / activity / jurisdiction, `PENDING_LEGAL_REVIEW` hit counts, overlay rejections, bundle age | Recommended (file 14 §7, §8.3) [REC] |
| Operational SLOs, alerting, audit-chain verification job, human-review queue age, data-quality reject rates | Recommended; requirements in file 08 SEC-21 and file 05 §9 [REC] |
| Backup/restore tooling: PostgreSQL native backups (e.g. `pg_basebackup` + WAL archiving) encrypted with bank keys; restore drill verifies audit and decision-log hash chains | Recommended; requirement file 08 SEC-20; RPO/RTO [MANAGER] [REC] |

### 3.12 Testing (cross-cutting)

pytest; Hypothesis property tests for money maths (refunds/reversals/FX invariants) **and for policy properties** (monotonicity, determinism, fail-safe, locked baseline; file 14 §14); Schemathesis against OpenAPI; golden-file contract examples; policy golden cases and simulator diff; host simulator (see 04). Pact deferred until a real host team exists. [REC]

### 3.13 Policy engine / policy-as-data (new)

What it does: decides, per request, which jurisdiction policies apply to an activity for a customer context and returns `PERMIT` / `PERMIT_WITH_OBLIGATIONS` / `DENY` plus obligations (file 14 §4). Main project uses one? [UNKNOWN].

| Candidate | Fit | Cx | Host risk | Verdict |
|---|---|---|---|---|
| **In-house PDP library (`core/policy`)**: policies as **YAML** validated by **JSON Schema 2020-12**, compiled to canonical JSON + signed manifest; **small closed predicate DSL** (`all/any/not`, `eq`, `in`, `in_subtree`, `lt/gt`); per-control combining rules from a versioned control catalogue; Pydantic models mirror the schema | Auditable by non-engineers (legal reviews YAML + golden cases); deterministic; replayable by `as_of`; no extra runtime; identical in both modes | Low-Med | Low (in-process, exposed via HTTP for hosts) | **Recommended** [REC] (file 14 §8) |
| OPA / Rego | Mature, general policy language, sidecar or WASM | Med | Med (extra runtime; Rego hard for legal reviewers) | Deferred; revisit at first new jurisdiction if the DSL is outgrown (file 14 J9) [REC] |
| AWS Cedar (open source) | Typed, analysable authorisation language | Med | Med | Deferred, same trigger [REC] |
| Business rules engine (Drools etc.) | Powerful, JVM, heavy | High | Med | Rejected |
| Jurisdiction branches in code (`if state == "NY"`) | Simple at first, unmaintainable, unreviewable by legal | Low then High | - | **Rejected**; CI lint forbids jurisdiction literals in `core/` outside the policy package (file 14 PG-1) |

Supporting choices [REC]:

| Concern | Choice |
|---|---|
| Policy source of truth | `policy-packs/` repository, changes by pull request; maker-checker approval (maker != approver) recorded in policy records (file 14 §9) |
| Pack integrity | Pack manifest with sha256 per file + detached signature; signing key held by compliance (standalone: dev key). PDP refuses unsigned or mismatched packs. Signing tool [UNKNOWN] - align with the platform's image-signing tool [MANAGER] |
| Storage | `PolicyStorePort`: local adapter (signed files on disk, metadata in Postgres) and host adapter (plugin overlay push) |
| Caching | Immutable effective bundle cached in-process, invalidated on activation; no TTL staleness (file 14 §8.3) |
| Jurisdiction-aware config | Two separate kinds of data: (1) **jurisdiction policy packs** (`PACK-GLOBAL-BASELINE`, `PACK-US`, `PACK-US-NY`, `PACK-US-NYC`) = legal controls, approved by legal; (2) **market reference data** (currency, catalogue, category maps, languages, timezone) = product/config data under maker-checker. Neither is selected by a `MARKET=` switch; the deployment declares `deployment_scope` (MVP `[US-NY-NYC]`) and the PDP resolves the jurisdiction set per customer context |
| Jurisdiction codes | ISO 3166-1 / 3166-2 plus custom local codes (`US-NY-NYC`) held as a reference-data tree (file 14 §2) |
| Testing | Schema + invariant tests, golden cases, Hypothesis property tests, policy-pack simulator CLI (file 14 §14) |
| Content | All MVP policies carry `illustrative: true` and cannot be APPROVED until legal confirms the corresponding file 13 row [LEGAL-CONFIRM] |

### 3.14 Hosting and vendor considerations (tied to file 13 questions)

These do not change the recommended stack; they are constraints the eventual hosting/vendor decision must satisfy. Each depends on answers in file 13 §2 and is [LEGAL-CONFIRM]. Classification: every technology implication below is a **RECOMMENDED CONTROL**. The cited sources are **BINDING LAW** (rows F01 Reg P, F03 FTC Safeguards Rule, S01 Part 500) or **SUPERVISORY GUIDANCE** (rows F14 SR 26-2, F15 OCC 2023-17, S02 DFS letters); whether each reaches this bank is **UNRESOLVED APPLICABILITY** (charter, covered-entity status, size).

| Consideration | File 13 reference | Technology implication | Label |
|---|---|---|---|
| Where the LLM is hosted (in-bank, cloud, third-party API) and whether vendor models/data are used | Q-P8; F15, S01, S02, F14 | Keep `LLMPort` adapters swappable; default template-only; no NPI in prompts to any external provider (file 08 SEC-15); prompt/response logging with redaction | [MANAGER] [LEGAL-CONFIRM] |
| Vendor/processor receiving nonpublic personal information (cloud, LLM host, analytics) | F01 (Reg P service-provider exception), file 13 §6 Q7 | Data-flow and vendor register as a maintained artefact; contractual-use flag per processor; tokenised data only | [LEGAL-CONFIRM] |
| Third-party relationship due diligence | F15 (OCC 2023-17), S02 (DFS Oct 2025 TPSP letter, if covered entity) | Vendor evidence pack: SBOM, SOC 2/ISO evidence, pen test, data location, subprocessors, exit plan, **AI no-training / no-retention clause** | [LEGAL-CONFIRM] |
| Part 500 controls if the operating entity is a covered entity | S01 (Q-C1, Q-C7, Q-C8) | MFA via host IdP; asset inventory entries for every component incl. models and LLM; encryption at rest/in transit; audit trail; vulnerability management | [LEGAL-CONFIRM] |
| Nonbank operator (fintech/BaaS model) | F03 (FTC Safeguards Rule), F09; Q-C4 | Same SEC controls designed to satisfy both F02 and F03 so the hosting entity choice does not force redesign | [LEGAL-CONFIRM] |
| Offline capability reduces third-party dependency | F15 | Offline-capable build, local LLM or templates (file 08 SEC-12) | [REC] |
| Future international deployment / data residency | file 14 §16 J8 | Same image, separate deployment with `deployment_scope=[GB]` etc. via `CTL-DATA-RESIDENCY`; no cross-deployment data | [LEGAL-CONFIRM] [MANAGER] |

---

## 4. Recommended stack (summary)

| Layer | Choice | Decision owner |
|---|---|---|
| Runtime | Python 3.12 | Subsystem [REC] |
| API | FastAPI + Pydantic v2, REST/JSON, OpenAPI 3.1; jurisdiction context in every customer-level request | Subsystem [REC] |
| Store | PostgreSQL 16 per data cell (2026-11-10: `A-US`, `A-UK`, `B-AE`; DEC-52, DEC-53); DuckDB+Parquet for batch | Subsystem [REC] |
| ML | scikit-learn, Fairlearn; LightGBM+SHAP gated | Subsystem [REC]; model approval process [MANAGER] |
| Feature store | Postgres feature snapshot tables | Subsystem [REC] |
| **Policy engine** | In-house PDP library; YAML + JSON Schema policy packs; closed predicate DSL; signed manifests; OPA/Cedar deferred | Subsystem [REC]; pack approval by legal/compliance [LEGAL-CONFIRM] |
| **Jurisdiction config** | Policy packs (federal / NY State / NYC; future packs) + market reference data; `deployment_scope` per deployment | Subsystem [REC]; scope [MANAGER] |
| LLM orchestration | In-house thin orchestrator behind `LLMPort`; separate GenAI governance track (recommended control; bank policy decision) | Subsystem [REC] |
| LLM model/provider | Template-only default; local open model or bank-approved endpoint | **Platform-wide** [MANAGER] [DOC p.73] |
| UI | React + TS + Vite (standalone only) | Subsystem [REC] |
| Messaging | Outbox + CloudEvents schemas; transport deferred | Transport is **platform-wide** [MANAGER] |
| AuthN/Z | OIDC JWT validation via port; `ri_deployment` + `ri_jurisdictions` claims | IdP is **platform-wide** [MANAGER] |
| Packaging | Wheel + OCI image + compose; signed + SBOM; policy packs signed separately | Orchestrator (K8s etc.) **platform-wide** [MANAGER] |
| Observability | JSON logs + OpenTelemetry | Collector/backend **platform-wide** [MANAGER] |

## 5. Subsystem vs platform-wide decisions

| Decision | Scope | Status |
|---|---|---|
| Internal language, libraries, internal module structure | Subsystem | Decide now [REC] |
| Public contract format (REST/OpenAPI, JSON, CloudEvents) | Subsystem proposes, platform ratifies | [MANAGER] |
| Identity provider, token claims (roles, `ri_deployment`, `ri_jurisdictions`) | Platform-wide | [MANAGER] [UNKNOWN] |
| Event bus technology | Platform-wide | [MANAGER] [UNKNOWN] |
| LLM model, hosting location, origin | Platform-wide | [MANAGER] [DOC p.73]; legal questions file 13 Q-P8 [LEGAL-CONFIRM] |
| Container orchestration, secrets/KMS, key vault | Platform-wide | [MANAGER] [DOC p.8, p.74] |
| Tokenisation service ownership | Platform-wide | [MANAGER] [DOC p.8, p.56] |
| Policy engine technology and pack format | Subsystem proposes | [REC]; revisit at first new jurisdiction |
| Policy pack content, applicability and approval | Legal/compliance | [LEGAL-CONFIRM] (file 13, file 14 §9) |
| Policy pack signing key ownership (us, host compliance, both) | Platform-wide | [MANAGER] (file 14 §16 J7; DEC ID assigned in file 12) |
| Deployment scope and data residency topology | Platform-wide | ~~MVP: single deployment `US-NY-NYC`~~ 2026-11-10: Track A one profile with cells `A-US`/`A-UK` (separate DBs, in-region); Track B separate bank on-prem deployment (DEC-51..DEC-53) [LEGAL-CONFIRM]; developer-delegated, manager countersignature recommended |
| Use of a hosted LLM for the standalone synthetic demo | Manager | [MANAGER] |

## 6. Staying compatible with an unknown host stack

1. **No in-process embedding.** The host never imports our Python; it calls HTTP (or files/events). [REC]
2. **Contract-first.** OpenAPI + JSON Schema committed and versioned; generated clients for any language (file 15). [REC]
3. **Ports and adapters.** Identity, data access, audit sink, event publisher, LLM, **policy store** and clock are interfaces; host-specific code lives only in adapters. [REC]
4. **Canonical data model** inside; host-to-canonical mapping in data adapters. [REC]
5. **Standards only at the boundary:** OIDC/JWT, ISO 4217 currency, ISO 8601 UTC timestamps, ISO 3166 jurisdiction codes (+ custom local codes), decimal strings for money, CloudEvents, OpenTelemetry, W3C trace context. [REC]
6. **One image, config-selected mode and scope** (`MODE=standalone|plugin`, `DEPLOYMENT_ID`, `DEPLOYMENT_SCOPE=US-NY-NYC`; 2026-11-10: replaced by a signed deployment profile selecting track, institution type, cells, packs, catalogue and locales, DEC-51); jurisdiction behaviour comes from signed policy packs, never from a `MARKET` switch. [REC]
7. **Exit path:** if the host mandates JVM/.NET, the contract, policy packs (data) and tests survive; only the core would be reimplemented, which is why the contract tests and policy golden cases are written black-box. [REC]

## 7. Open items

- Host stack, IdP, event bus, scheduler, observability backend: [UNKNOWN] -> ask manager.
- Whether the subsystem is deployed by the same team/pipeline as the credit platform: [UNKNOWN] [MANAGER].
- Allowed LLM for standalone demo: [MANAGER].
- ~~Confirm UK/other markets are future expansion only (file 14 §16 J1): [MANAGER].~~ Superseded 2026-11-10 by DEC-50 (two tracks; UK London in Track A).
- LLM hosting location and vendor terms against file 13 Q-P8, F15, S02: [LEGAL-CONFIRM].
- Whether the bank's governance framework for the LLM orchestrator (given that SR 26-2 leaves GenAI / agentic AI governance to the bank's own risk management practices, file 13 F14) imposes specific tooling: [LEGAL-CONFIRM] / MRM.
- Predicate DSL vs OPA/Cedar: revisit at first new jurisdiction (file 14 §16 J9). [REC]

## Change log (NYC scope update)

- 2026-11-09: Added scope-update banner (NYC first: federal + NY State + NYC; no 50-state rulebooks; jurisdictions as policy packs; UK/other markets future [MANAGER confirm]); added [LEGAL-CONFIRM] label.
- §1/§2: replaced US/UK separate-install framing with deployment scope + jurisdiction packs; added needs for jurisdiction-aware, replayable, fail-safe policy.
- New §3.13 policy engine / policy-as-data choice (in-house PDP, YAML + JSON Schema, closed DSL, signed packs; OPA/Cedar deferred; jurisdiction literals in code rejected) and jurisdiction-aware config (packs vs market reference data).
- §3.6: added separate LLM governance track note, citing file 13 F14 (GenAI/agentic AI "not within the scope" of SR 26-2 / OCC 2026-13; separate track is a recommended control, not a regulatory requirement) as [LEGAL-CONFIRM].
- New §3.14 hosting/vendor considerations mapped to file 13 questions and rows (Q-P8, F01, F03, F15, S01, S02).
- §3.3, §3.9, §3.10-3.12, §4-§7: DB per deployment, `ri_deployment`/`ri_jurisdictions` claims, separately signed packs, policy metrics and tests, updated decision ownership and open items.

## Change log (validation pass 2026-11-09)

- Shared ID scheme (D-06, R-03, R-04): "file 14 G1/G3/G4" -> "file 14 PG-1/PG-3/PG-4"; "file 15 C4" -> "file 15 IC4"; file 14 open items cited as "file 14 §16 Jn" (R-19).
- §3.2: responses also carry `permitted_use` / `not_for_credit_decisioning` (file 15 IC7) (D-11).
- §3.4 model governance: file 13 wording item 3 applied (SR 26-2 supersedes SR 11-7 and SR 21-8; OCC 2026-13 rescinds OCC 2011-12 and 2021-19; FDIC FIL-15-2026; "expected to be most relevant" over $30bn is a relevance statement, not a legal threshold); classified SUPERVISORY GUIDANCE / UNRESOLVED APPLICABILITY.
- §3.6: file 13 wording items 1 and 2 applied (fn. 3 quoted; separate LLM governance track is a RECOMMENDED CONTROL and bank policy decision, not a requirement of the guidance); DFS letters classified SUPERVISORY GUIDANCE.
- §3.9: Part 500.12 classified BINDING LAW for covered entities / UNRESOLVED APPLICABILITY; MFA marked RECOMMENDED CONTROL.
- §3.11: added operational SLO/alerting row and backup/restore tooling row (gaps G-3, observability; file 08 SEC-20/SEC-21) [REC].
- §3.14: classification note added (BINDING LAW / SUPERVISORY GUIDANCE / RECOMMENDED CONTROL / UNRESOLVED APPLICABILITY); "Q-legal 7" -> "file 13 §6 Q7" (D-42).
- §4 table: file 13 wording item 4 applied. §7: wording item 5 applied. NYC change log: wording item 6 applied.

## Change log (two-track scope 2026-11-10)
- Two-track banner added; §3.3 store and §4 store row: PostgreSQL per data cell (DEC-52, DEC-53); §3.6 LLM hosting per track (DEC-58); §3.7 Arabic/RTL and per-cell currency (DEC-59); §5 deployment topology row; §6 item 6 deployment profile (DEC-51); §7 UK item superseded (DEC-50). No new technology added.
