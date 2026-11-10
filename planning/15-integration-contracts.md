# 15 - Integration Contracts (v1)

**Label key:** [REC] recommendation | [ASSUME] assumption | [UNKNOWN] | [MANAGER] needs manager approval | [LEGAL-CONFIRM] legal/compliance must confirm | [OPEN] undefined integration point | [DOC p.N] PDF page.
**Status:** planning only, 2026-11-09. Schemas in this file are sketches. **Source of truth (P-4 done, PH-3):** the normative OpenAPI 3.1 contract `contracts/openapi/retail-intelligence-v1.yaml` (the file 04 / spine name `contracts/openapi/v1.yaml`; request/response schemas live in its `components/schemas`) and the CloudEvents schemas `contracts/events/*.schema.json`; see `contracts/README.md`. Where this file and the contract differ, the contract wins and this file is corrected.
**Related:** 04 (architecture/versioning), 07 (data model), 08 (security), 14 (jurisdiction policy layer), 13 (NYC applicability matrix - legal content).

## 1. Principles

| # | Principle | Label |
|---|---|---|
| IC1 | Contract-first: OpenAPI 3.1 + JSON Schema 2020-12 are the source of truth; same contract serves plugin hosts and the standalone UI/adapters | [REC] |
| IC2 | Advisory only: every response is insight, prediction, suggestion or referral. **No response type represents a credit approval, decline, limit, pricing or adverse action.** No endpoint changes accounts or contacts customers | User; [DOC p.53, p.69] |
| IC3 | Data comes to us: host pushes authorised data; we never reach into host databases | file 04 [REC] |
| IC4 | Every customer-level request carries jurisdiction context and consent purpose; every response carries the policy versions applied and a decision-log id | file 14 [REC] |
| IC5 | Tokenised identifiers only; minimum fields per activity | [DOC p.8, p.16, p.74] |
| IC6 | Tolerant reader for responses; strict validation for requests | file 04 [REC] |
| IC7 | **Not for credit decisioning (purpose limitation):** every customer-level response and customer-outcome event carries `permitted_use: ["RETAIL_INSIGHT","MARKETING_SUGGESTION"]` and `not_for_credit_decisioning: true`. Host integration terms prohibit using our outputs (profile features such as income stability or spend vs income, scores, segments, reason codes) for credit eligibility, underwriting, pricing, limits or any other credit decision, because such use could make them consumer-report or credit-decision inputs (file 13 rows F04, F06, S06; **UNRESOLVED APPLICABILITY**). Labels are set by the shared core's response assembler in both modes; misuse prevention beyond the label is a contractual/host-governance control | [REC] [LEGAL-CONFIRM] |

Principle IDs were renamed C1-C6 -> IC1-IC6 in the validation pass to avoid confusion with file 13 rows C01-C07.

## 2. API surface (v1)

Base path `/v1`. All endpoints except `/health` require OIDC bearer (section 5).

| Resource | Method + path | Purpose | Scope | Activity (file 14) |
|---|---|---|---|---|
| Capabilities | `GET /v1/capabilities` | Manifest: contract version, supported activities, jurisdictions in deployment scope, runtime-bundle pack versions with `pack_state` (`ACTIVE` or `ILLUSTRATIVE`), `demo` flag, serving registry entries (`RULESET`/`ML_MODEL`, status); **read-only `deployment_profile`** (`profile_id`, `profile_version`, `track`, `institution_type` `CONVENTIONAL`/`ISLAMIC`, `cell_id`, `currencies`, `locales`, `catalogue_id`, `institution_overlay`; DEC-61, file 21 §10). No request field can set or override the profile | `ri.meta.read` | - |
| Health | `GET /health`, `GET /ready` | Liveness / readiness (ready = DB up + verified L0 + every configured pack verified and loadable; an `ILLUSTRATIVE` pack outside a standalone demo makes the instance not ready; file 14 §8.4) | none / `ri.meta.read` | - |
| Customer insights | `POST /v1/customers/{customer_token}/insights:evaluate` | Stateless evaluate: profile + features + health indicators from supplied window | `ri.insights.read` | PROFILE_BUILD |
| Customer insights (stored) | `GET /v1/customers/{customer_token}/insights` | Latest stored profile (stateful mode) | `ri.insights.read` | PROFILE_BUILD |
| Customer directory (DEC-62, G-RS-2; purpose-bound since contract 1.2.0, DEC-68) | `GET /v1/customer-directory?access_purpose=&customer_token=&segment_code=&data_quality_status=&cursor=` (replaces `GET /v1/customers`, deprecated and not implemented) | Picker list for the serving cell, with a required approved access purpose, an exact token lookup or a narrowing filter, page <= 25, query total <= 100, per-user quotas and an audit record per call (file 21 §2.3): tokenised ids only (customer token, resolved jurisdiction code, age-status class, data-quality flag, last-updated); no name, contact, address, ZIP or account number; minors listed only as protected; cursor-paged; ROW_FILTER by jurisdiction claim | `ri.customers.list` (analyst, relationship staff) | PROFILE_BUILD |
| Stored transactions (DEC-62, G-RS-3) | `GET /v1/customers/{customer_token}/transactions?cursor=` | Enriched stored transactions (stateful mode): date, amount, currency, txn kind, category code, flags; no raw description, counterparty or merchant name; sensitive or prohibited categories only as neutral category codes; consent purpose `INSIGHT` required; minors blocked except age/routing (file 14 §13.3) | `ri.transactions.read` (purpose-bound; relationship staff and analyst) | TXN_ENRICHMENT |
| Portfolio summary (DEC-62, G-RS-1) | `GET /v1/portfolio/summary` | Aggregate-only counts and distributions by segment, band, need and jurisdiction; no tokens or PII; cells below a configured minimum (default 20 [ASSUME]) suppressed; minors excluded from commercial aggregates; aggregation across Track A cells allowed for these aggregates only, never across tracks (DEC-52) | `ri.portfolio.read` (viewer, analyst) | SEGMENTATION_INSIGHT |
| Transaction enrichment | `POST /v1/transactions:enrich` | Categorise, flag recurring/transfer/refund/reversal/duplicate for a batch (max 5,000) | `ri.transactions.enrich` | TXN_ENRICHMENT |
| Segments | `GET /v1/customers/{customer_token}/segments` | Memberships with reasons, validity, purpose (INSIGHT/COMMERCIAL) | `ri.segments.read` | SEGMENTATION_* |
| Churn score | `GET /v1/customers/{customer_token}/churn-score` | Score, band, drivers, model version | `ri.predictions.read` | CHURN_PREDICTION |
| Needs | `GET /v1/customers/{customer_token}/needs` | Need list with score, drivers, evidence | `ri.predictions.read` | NEEDS_PREDICTION |
| Composite evaluate | `POST /v1/customers/{customer_token}/evaluate` | Aggregate of insights + segments + needs + churn + recommendations (file 04) | union of sub-scopes | per sub-result |
| Recommendations | `POST /v1/customers/{customer_token}/recommendations:generate` | Ranked catalogue-grounded suggestions with gate trace | `ri.recommendations.generate` | PRODUCT_RECOMMENDATION, CREDIT_PRODUCT_REFERRAL, MARKETING_ELIGIBILITY |
| Recommendation feedback | `POST /v1/recommendations/{rec_id}/outcomes` | Host reports SHOWN / ACCEPTED / DISMISSED_BY_CUSTOMER / OVERRIDDEN (+ fixed reason code), and in plugin mode the human-review result (CLEARED_TO_SHOW / REJECTED / ESCALATED / EXPIRED, reviewer token, mandatory reason on REJECTED/ESCALATED) | `ri.recommendations.feedback` | - |
| Retention suggestions | `GET /v1/customers/{customer_token}/retention-actions` | Advisory action suggestions + priority | `ri.retention.read` | RETENTION_SUGGESTION |
| Retention action outcome | `POST /v1/retention-actions/{action_id}/outcomes` | Host (or staff via dashboard) reports `outcome` (NOT_ACTIONED / CONTACTED / RETAINED / CLOSED / UNKNOWN), `outcome_reason` (fixed code), `outcome_at`, `outcome_source` (HOST / STAFF); plugin human-review result as for recommendations; append-only (file 07 `retention_outcome`) | `ri.retention.feedback` | - |
| Human review queue (standalone; optional for hosts) | `GET /v1/review-items?state=&role=`, `POST /v1/review-items/{review_item_id}:transition` | List review items; move QUEUED -> IN_REVIEW -> CLEARED_TO_SHOW / REJECTED / ESCALATED (reason mandatory on REJECTED/ESCALATED; reviewer != requester); EXPIRED/CANCELLED set by the system (file 07 `review_item`) | `ri.review.read`, `ri.review.write` | - |
| Explanations | `GET /v1/explanations/{result_id}` | Plain-language explanation of a prior result (template or verified LLM) | `ri.explanations.read` | EXPLANATION_GENERATION |
| Policy decision query | `POST /v1/policy/decisions` | Host asks "would activity X be permitted for this context?" - returns outcome + obligations, no customer data processing | `ri.policy.query` | any (dry-run) |
| Policy packs (read) | `GET /v1/policy/packs?state=`, `GET /v1/policy/packs/{pack_id}/{version}` | Pack manifests and policy metadata in any `pack_state` (default filter: runtime bundle); the single-pack read is the **pack status** view (§2.1) | `ri.policy.read` | - |
| Host policy overlay | `PUT /v1/policy/overlays/{overlay_id}` | Plugin: host supplies stricter overlay (signed); rejected parts reported | `ri.policy.overlay.write` | - |
| Consent ingest | `POST /v1/consents:batchUpsert` | Host pushes consent records (source, purpose, granted/expiry/withdrawn) | `ri.consent.write` | - |
| Preference ingest | `POST /v1/preferences:batchUpsert` | Marketing opt-outs, channel preferences, contact limits, vulnerability flag refs | `ri.consent.write` | - |
| Bulk data ingest | `POST /v1/ingest/batches` + `PUT /v1/ingest/batches/{id}/parts/{n}` + `POST .../{id}:commit` | Customers, accounts, transactions, goals, catalogue (stateful mode) | `ri.ingest.write` | - |
| Ingest status | `GET /v1/ingest/batches/{id}` | Counts, rejects, validation report | `ri.ingest.read` | - |
| Data subject request | `POST /v1/dsr-requests` | Access/delete/object-to-profiling by token; deletion of chained audit/decision-log data by crypto-shredding (file 07 §8.2). Placeholder in v1; full DSR support post-MVP | `ri.dsr.write` | - [LEGAL-CONFIRM] |
| Audit export | `GET /v1/audit/records?from=&to=&cursor=` | Hash-chained audit + decision log export (NDJSON) | `ri.audit.export` | AUDIT_EXPORT |
| Assistant | `POST /v1/assistant/messages` | Staff Q&A over tool results; read-only | `ri.assistant.use` | ASSISTANT_QA |

Composite convenience (row "Composite evaluate" above): each sub-result carries its own `decision_id`. [REC]

**Tracks and cells (2026-11-10; file 21):** the same contract serves every track and cell. A deployment serves only its own cell's jurisdictions; a customer token from another cell or track is not found, and a jurisdiction from another track resolves to `UNSUPPORTED_JURISDICTION`. Decision and audit records carry `profile_id`, `track`, `institution_type` and `cell_id` (DEC-61). New reference reason codes: `SHARIA_REVIEW_PENDING`, `CATALOGUE_ITEM_NOT_ELIGIBLE`, `TERMINOLOGY_VIOLATION_FALLBACK`, `PROHIBITED_CATEGORY_EXCLUDED`, `CELL_SCOPE_MISMATCH`. Locales per cell (Track B: `ar`, `en`; DEC-59).

**Validation tightenings (2026-11-10; file 21 §2.2, §3.1, §10A; file 12 §H.7.3):** (1) tokens (IC5, CI-4) are not reversible by the subsystem; host tokens are random or keyed per cell and track, so the same person has unrelated tokens in each cell and track, and no record carries a cross-context identifier; (2) the portfolio summary follows aggregate disclosure control AG-1..AG-10 (one fixed dimension, cell-root jurisdiction filter only, published snapshot dates only, floor 20 that config can only raise, rounding, complementary suppression, suppression inside each cell before any Track A combination, per-user quota and query audit); (3) the deployment profile is verified against track-bound keys and cannot be switched by environment, flag or request. Contract 1.1.2 carries these as descriptions.

For a minor or age-unknown subject (age threshold per jurisdiction, DEC-60) every activity follows the two-tier block list in file 14 §13.3 (`CTL-MINOR-PROTECT`); blocked sections return `status: "DENIED"` with reason `MINOR_PROTECTED` (§4).

### 2.1 Governance, simulator, decision-log and model-registry endpoints (DEC-46) [REC]

Used by the standalone dashboard (AD-2: UI reaches the core only through `/v1`) and optionally by host governance tools. Pack states follow file 14 §8.4 (DEC-45); `ILLUSTRATIVE` follows DEC-44. Every call is audited; every state change appends to the pack or registry history. Governance schemas are outside the forbidden-term linter's output scope (§3.3).

| Resource | Method + path | Purpose and rules | Scope (role) |
|---|---|---|---|
| Pack upload | `POST /v1/policy/packs` | Upload a candidate pack (YAML bundle + manifest); creates `DRAFT`; records maker | `ri.policy.author` (`policy_author`) |
| Pack validate | `POST /v1/policy/packs/{pack_id}/{version}:validate` | Runs schema, invariants, golden cases and monotonicity; on pass the build signs the manifest and the pack becomes `VALIDATED` (loaded but unapproved: simulator only); returns the validation report | `ri.policy.author` |
| Pack submit | `POST /v1/policy/packs/{pack_id}/{version}:submit` | `VALIDATED` -> `IN_LEGAL_REVIEW`; refused without a simulator report (`simulation_id`) from the current runtime bundle | `ri.policy.author` |
| Pack approval (maker-checker) | `POST /v1/policy/packs/{pack_id}/{version}/approvals` | Per-policy decision (`APPROVED` with `applicability_status`, or `CHANGES_REQUESTED` with reason); approver != maker (`MAKER_CHECKER_VIOLATION`); `illustrative: true` policies refused (`ILLUSTRATIVE_NOT_APPROVABLE`); pack becomes `APPROVED` when every member is approved. While DEC-11 is unresolved no user holds the scope, so every call is 403 and the pack status shows `approver_assigned: false` | `ri.policy.approve` (`policy_approver`) |
| Pack activation | `POST /v1/policy/packs/{pack_id}/{version}/activations` | Body `{target_state: ACTIVE or ILLUSTRATIVE, signoff_refs[]}`. `ACTIVE` needs pack state `APPROVED` + recorded MGR/LEG sign-off refs. `ILLUSTRATIVE` needs a `VALIDATED` all-illustrative pack in a deployment with `mode = STANDALONE`, `demo = true` and the synthetic-only guard (else `ILLUSTRATIVE_NOT_PERMITTED`). Activator != maker. For `ILLUSTRATIVE`, the body also lists each `illustrative_assumption` confirmed by the activator (the DEC-44 second-user check), stored on the activation record. Rollback = activating an earlier version; the replaced version becomes `RETIRED`. Emits `policy.pack.activated` with `pack_state` | `ri.policy.release` (`policy_release`) |
| Pack status | `GET /v1/policy/packs/{pack_id}/{version}` | `pack_state`, per-policy `approval_status` / `applicability_status` / `illustrative_assumption`, validation and simulator report refs, `approver_assigned`, sign-off refs, activation history | `ri.policy.read` |
| Simulation run | `POST /v1/policy/simulations` | Async. Candidate pack (any state except `RETIRED`) vs the current runtime bundle over golden cases + the synthetic population (synthetic only); returns `simulation_id`. A run whose candidate hits PENDING_LEGAL_REVIEW policies writes an audit record and emits `policy.review.required` with `source: SIMULATION` (file 14 §8.4) | `ri.policy.simulate` |
| Simulation result | `GET /v1/policy/simulations/{simulation_id}` | Status and diff report: decisions changed by activity and jurisdiction, golden-case pass/fail; attachable to `:submit` | `ri.policy.simulate` or `ri.policy.read` |
| Decision read | `GET /v1/policy/decisions/{decision_id}` | Decision-log record (file 14 §7): packs, `bundle_state`, policy hits, outcome, obligations, reason codes; no PII | `ri.audit.export` |
| Decision replay | `POST /v1/policy/decisions/{decision_id}:replay` | Re-runs the decision with recorded pack versions and `as_of`; returns `{identical: true or false, diff}`; writes nothing but an audit record | `ri.audit.export` |
| Model registry list | `GET /v1/models`, `GET /v1/models/{model_id}/versions/{version}` | Entries with `kind` (`RULESET`/`ML_MODEL`), status, feature list, aggregate metrics; never customer-level SHADOW outputs | `ri.models.read` |
| Model status transition | `POST /v1/models/{model_id}/versions/{version}:transition` | `RULESET`: `DRAFT` -> `ACTIVE` -> `RETIRED`. `ML_MODEL`: `DRAFT` -> `SHADOW` -> `RETIRED`; -> `APPROVED` needs an MRM approval reference and is refused while DEC-19 is unresolved (`MODEL_APPROVAL_UNAVAILABLE`) (DEC-47). Emits `model.status_changed` | `ri.models.write` (`model_admin`) |

## 3. Canonical envelope

### 3.1 Request (customer-level) [REC]

```json
{
  "contract_version": "1.0.0",
  "correlation_id": "c0a8...-host",
  "as_of": "2026-11-09T00:00:00Z",
  "jurisdiction_context": {
    "residence_jurisdiction": "US-NY-NYC",
    "booking_entity_jurisdiction": "US-NY",
    "servicing_location": "US-NY-NYC",
    "channel": "MOBILE_APP"
  },
  "consent": {
    "purpose": "PERSONALISATION",
    "consent_refs": ["cns_tok_123"]
  },
  "subject": {"customer_token": "cust_tok_A", "age_status": "ADULT"},
  "host_policy_decision_ref": null,
  "data": {
    "accounts": [{"account_token": "acc_tok_1", "type": "CURRENT", "currency": "USD"}],
    "transactions": [{"txn_ref": "t-1", "account_token": "acc_tok_1", "booking_ts": "2026-10-01T10:00:00Z",
                      "amount": "-42.50", "currency": "USD", "status": "BOOKED",
                      "merchant_ref": "m-889", "mcc": "5411"}]
  },
  "options": {"explanation": {"language": "en", "allow_llm": true}}
}
```

Headers: `Authorization: Bearer <JWT>`, `Idempotency-Key` (for POST), `traceparent` (W3C), `X-Contract-Version: 1.x` (optional pin). Money as **decimal strings** with ISO 4217 currency; timestamps ISO 8601 UTC. [REC]

### 3.2 Response (customer-level) [REC]

```json
{
  "contract_version": "1.0.0",
  "correlation_id": "c0a8...-host",
  "result_id": "res_01J...",
  "advisory_only": true,
  "permitted_use": ["RETAIL_INSIGHT", "MARKETING_SUGGESTION"],
  "not_for_credit_decisioning": true,
  "illustrative": false,
  "watermark": null,
  "subject": {"customer_token": "cust_tok_A", "protection_status": "ADULT"},
  "policy": {
    "decision_id": "dec_01J...",
    "bundle_state": "ACTIVE",
    "outcome": "PERMIT_WITH_OBLIGATIONS",
    "jurisdiction_set": ["*", "US", "US-NY", "US-NY-NYC"],
    "packs_applied": [{"pack_id": "PACK-US-NYC", "pack_version": "0.3.0"}],
    "effective_bundle_sha256": "...",
    "obligations": [{"control_id": "CTL-HUMAN-REVIEW", "params": {"role": "relationship_staff"}}],
    "flags": []
  },
  "versions": {"model": "churn-nyc-0.4.0", "feature_set": "fs-1.2.0", "catalogue": "cat-demo-2026-10", "config": "cfg-0.9.1"},
  "recommendations": [{
    "rec_id": "rec_01J...",
    "kind": "PRODUCT_SUGGESTION",
    "product_code": "SAV-DEMO-01",
    "rank": 1,
    "status": "PROPOSED",
    "reason_codes": ["NEED_EMERGENCY_BUFFER", "SALARY_REGULAR"],
    "gate_trace": [{"control_id": "CTL-CHANNEL-ALLOW", "result": "PASS", "params": {"channels": ["MOBILE_APP", "EMAIL"]}}],
    "requires_human_review": true,
    "review_item_id": "rvw_01J...",
    "allowed_channels": ["MOBILE_APP", "EMAIL"],
    "expires_at": "2026-11-23T00:00:00Z"
  }],
  "explanation": {"text": "...", "language": "en", "generated_by": "TEMPLATE", "numbers_verified": true, "ai_assisted": false},
  "suppressed": [{"product_code": "CARD-DEMO-02", "reason_codes": ["MARKETING_OPT_OUT"]}]
}
```

Enum notes: `subject.protection_status` = `ADULT` | `MINOR_PROTECTED` | `AGE_UNKNOWN` (age unknown is treated as minor for commercial activities, file 14 §10). Channel values follow file 14 §3.4 (`MOBILE_APP`, `WEB`, `BRANCH_STAFF`, `EMAIL`, `SMS`, `PUSH`, `CALL_CENTRE`). `recommendations[].status` = `PROPOSED` | `SUPPRESSED` | `PENDING_REVIEW` | `SHOWN` | `ACCEPTED` | `DISMISSED_BY_CUSTOMER` | `OVERRIDDEN` | `EXPIRED` (file 07). `permitted_use` and `not_for_credit_decisioning` are required on every customer-level response and customer-outcome event payload (IC7). `illustrative` and `watermark` are also required (DEC-44): when `policy.bundle_state` is `ILLUSTRATIVE` (standalone demo only) they are `true` and `"illustrative - not legal advice - not for real customers"`, the decision flags include `ILLUSTRATIVE`, and the same pair is set on events, exports and explanations; in any other deployment they are `false` / `null`. Set by the shared-core response assembler.

### 3.3 No credit-decision type (hard rule) [REC]

- Allowed `recommendations[].kind`: `PRODUCT_SUGGESTION`, `SERVICE_SUGGESTION`, `EDUCATIONAL_CONTENT`, `CREDIT_REFERRAL`. A `CREDIT_REFERRAL` means only "refer to the credit platform/bank process for assessment"; it carries no eligibility, limit, rate or likelihood. [DOC p.4, p.31]
- No customer-outcome schema contains fields or enum values such as `approved`, `declined`, `credit_decision`, `credit_limit`, `apr`, `adverse_action`, `pre_approved`, `eligibility_decision`. A CI schema linter fails the build if any appear (file 14 §13.1). **Linter scope:** customer-outcome output types - response and event payload schemas for insights, segments, predictions, recommendations, retention actions/outcomes, review items, explanations and assistant answers. Our own values there are named to pass (`DISMISSED_BY_CUSTOMER`, `CLEARED_TO_SHOW`). **Excluded (explicit allow-list of schema paths):** governance and lifecycle schemas - policy pack/policy reads (`approval_status: APPROVED`, `approver`, `approved_at`), model registry and the `model.status_changed` event (`APPROVED`/`RETIRED`), `MINOR-APPROVAL-*` record refs. Same choice as files 07 and 14.
- Assistant intents requesting credit approval/decline return `422 OUT_OF_SCOPE_REQUEST` with a handoff message template (`CTL-ADVERSE-ACTION-HANDOFF`).

### 3.4 Policy decision query

```json
// POST /v1/policy/decisions
{"activity": "PRODUCT_RECOMMENDATION", "purpose": "PERSONALISATION",
 "jurisdiction_context": {"residence_jurisdiction": "US-NY-NYC", "channel": "PUSH"},
 "subject": {"age_status": "ADULT"}, "as_of": "2026-11-09"}
// 200
{"decision_id": "dec_...", "outcome": "DENY", "reason_codes": ["POLICY_PENDING_LEGAL_REVIEW"],
 "flags": ["PENDING_LEGAL_REVIEW"], "obligations": [], "packs_applied": [...], "effective_bundle_sha256": "..."}
```

Outcomes: `PERMIT`, `PERMIT_WITH_OBLIGATIONS`, `DENY` (`INDETERMINATE` is never returned; it is enforced as `DENY` with reason `POLICY_INDETERMINATE`). Responses never claim a legal conclusion; reason codes reference policy IDs/statuses only. [REC]

## 4. Error model

RFC 9457 `application/problem+json`, extended. [REC]

```json
{"type": "https://errors.retail-intel.invalid/consent-missing", "title": "Consent missing for purpose",
 "status": 403, "code": "CONSENT_MISSING", "correlation_id": "...", "decision_id": "dec_...",
 "retryable": false, "details": [{"field": "consent.purpose", "issue": "no valid consent for PERSONALISATION"}]}
```

| HTTP | code | When |
|---|---|---|
| 400 | `VALIDATION_FAILED` | Schema violation, unknown request field (strict mode), bad currency |
| 401 | `UNAUTHENTICATED` | Missing/invalid/expired token |
| 403 | `FORBIDDEN_SCOPE`, `JURISDICTION_NOT_PERMITTED` | Scope missing; token jurisdiction claims don't cover request context |
| 403 | `CONSENT_MISSING`, `POLICY_DENIED`, `MINOR_PROTECTED` | Policy layer denies (decision_id included) |
| 404 | `NOT_FOUND` | Unknown token/result (no existence leak across jurisdictions: same 404) |
| 409 | `IDEMPOTENCY_CONFLICT` | Same key, different payload hash |
| 409 | `PACK_STATE_INVALID`, `REVIEW_STATE_INVALID`, `MAKER_CHECKER_VIOLATION`, `ILLUSTRATIVE_NOT_PERMITTED`, `MODEL_APPROVAL_UNAVAILABLE` | Governance or review-item transition not allowed from the current state; same person as maker (or reviewer = requester); illustrative activation outside a standalone demo; ML approval while DEC-19 is unresolved (§2.1) |
| 422 | `ILLUSTRATIVE_NOT_APPROVABLE` | Approval requested for an `illustrative: true` policy (§2.1) |
| 412 | `CONTRACT_VERSION_UNSUPPORTED` | Pinned version not served |
| 413 | `PAYLOAD_TOO_LARGE` | Limits exceeded |
| 422 | `OUT_OF_SCOPE_REQUEST`, `UNSUPPORTED_JURISDICTION`, `DATA_QUALITY_INSUFFICIENT` | Credit-decision ask; jurisdiction outside deployment; data gaps (never filled by guess; rules DQ-01..DQ-10 in file 07 §4.1, failing `rule_id`s listed in `details`) |
| 429 | `RATE_LIMITED` | With `Retry-After` |
| 503 | `POLICY_UNAVAILABLE`, `DEPENDENCY_UNAVAILABLE` | Policy bundle not verified; fail closed |

Partial results: a composite evaluate may return 200 with some sections `{"status": "DENIED", "reason_codes": [...], "decision_id": "..."}`; denied sections are never silently omitted. [REC]

## 5. Authentication and authorisation

| Item | Design | Label |
|---|---|---|
| Protocol | OIDC / OAuth2 bearer JWT validated against host JWKS (plugin) or local issuer (standalone); mTLS optional where host requires | file 08 SEC-01, SEC-05 |
| Service identity | Host calls with client-credentials tokens (`sub` = service id); staff-originated calls carry `act` (actor) claim or `X-On-Behalf-Of` signed token for audit | [REC] [UNKNOWN host IdP] |
| Scopes | `ri.meta.read`, `ri.insights.read`, `ri.transactions.enrich`, `ri.segments.read`, `ri.predictions.read`, `ri.recommendations.generate`, `ri.recommendations.feedback`, `ri.retention.read`, `ri.retention.feedback`, `ri.review.read`, `ri.review.write`, `ri.explanations.read`, `ri.policy.query`, `ri.policy.read`, `ri.policy.author`, `ri.policy.approve`, `ri.policy.release`, `ri.policy.simulate`, `ri.models.read`, `ri.models.write`, `ri.policy.overlay.write`, `ri.consent.write`, `ri.ingest.write`, `ri.ingest.read`, `ri.dsr.write`, `ri.audit.export`, `ri.assistant.use` | [REC] |
| Jurisdiction claims | Replace the old `market` claim: `ri_deployment` (must equal instance `deployment_id`) and `ri_jurisdictions` (list of subtrees caller may act for, e.g. `["US-NY-NYC"]`). Request context outside the claimed subtrees -> 403 `JURISDICTION_NOT_PERMITTED` | file 14; [REC] |
| Roles (staff) | viewer, analyst, relationship_staff, model_admin, auditor, policy_author, policy_approver, policy_release, service_host | file 08 SEC-02 + file 14 [MANAGER] |
| Token lifetime | Short-lived (proposal 5-15 min); no refresh tokens accepted by us | [REC] |
| Audit | Every authenticated call logged with `sub`, `act`, scopes, correlation id | file 08 SEC-07 |

### 5.1 Role-to-scope and screen matrix (gap G-2; file 08 SEC-19) [REC] [MANAGER]

**Normative version: `contracts/role-scope-matrix.md` / `.json`** (updated 2026-11-10 for DEC-62 and DEC-55); this table is the planning summary and the contract files win where they differ. Additions on 2026-11-10: scope `ri.transactions.read` (stored transactions; relationship_staff and analyst R), scope `ri.portfolio.read` (portfolio summary; viewer and analyst R), and role `sharia_approver` (Track B only; records `sharia_approval_status` on packs, catalogue items, templates and reason-code texts; never also `policy_author`; DEC-55). Interim approver roles per track (DEC-63): Track A Head of Compliance -> `policy_approver`; Track B Head of Compliance -> `policy_approver` plus Sharia Compliance/ISSC -> `sharia_approver`; no user holds either scope until a named person is recorded.

Deny by default: a role has only the scopes marked. R = read, W = write/act, - = none. Screens are standalone-dashboard screens; in plugin mode the host maps its own roles to these scopes.

| Scope / screen | viewer | analyst | relationship_staff | model_admin | auditor | policy_author | policy_approver | policy_release | service_host |
|---|---|---|---|---|---|---|---|---|---|
| `ri.meta.read` | R | R | R | R | R | R | R | R | R |
| `ri.insights.read`, `ri.segments.read`, `ri.predictions.read` (customer view screen) | - (portfolio summaries only) | R | R | - | R | - | - | - | R |
| `ri.transactions.enrich`, `ri.ingest.write`, `ri.ingest.read` | - | - | - | - | - | - | - | - | W |
| `ri.recommendations.generate`, `ri.retention.read`, `ri.explanations.read` | - | R | R | - | R | - | - | - | W |
| `ri.recommendations.feedback`, `ri.retention.feedback` | - | - | W | - | - | - | - | - | W |
| `ri.review.read` / `ri.review.write` (review queue screen) | - | R | R / W | - | R | - | - | - | R / W (host-run queue) |
| `ri.assistant.use` | - | W | W | - | - | - | - | - | - |
| `ri.policy.query`, `ri.policy.read` (policy pack viewer, simulator screen) | - | R | - | R | R | R / run simulator | R / run simulator | R | R |
| `ri.policy.simulate` (run simulator) | - | - | - | - | - | W | W | - | - |
| `ri.policy.author`: policy authoring, pack upload/validate/submit (maker) | - | - | - | - | - | W | - | - | - |
| `ri.policy.approve`: policy approval (checker; sets `applicability_status`) | - | - | - | - | - | - | W | - | - |
| `ri.policy.release`: pack activation `ACTIVE` / `ILLUSTRATIVE` (demo only) | - | - | - | - | - | - | - | W (`ACTIVE` with MGR/LEG sign-off) | - |
| `ri.policy.overlay.write` | - | - | - | - | - | - | - | - | W (signed overlay) |
| `ri.models.read` / `ri.models.write` (model registry screen; status change) | - | R / - | - | R / W (`ML_MODEL` `APPROVED` only with MRM approval ref; DEC-47) | R / - | - | - | - | - |
| `ri.consent.write`, `ri.dsr.write` | - | - | - | - | - | - | - | - | W |
| `ri.audit.export` (audit / decision log screen) | - | - | - | - | R | - | R | - | R (plugin export) |
| Name "reveal" | Never in this subsystem - host reveal function only | | | | | | | | |

Standalone demo issuer (DEC-11, DEC-44): it issues one demo user per role except `policy_approver`, which stays unassigned until DEC-11 names an approver, so the approval step shows as unavailable. The manager drives the demo through those role users (for example `analyst` for views and `policy_author` for the simulator); "manager" is not a role.

Separation of duties: maker != approver; `policy_author` cannot approve or activate; `policy_release` cannot author or approve; auditors are read-only; nobody can approve their own item. The matrix is versioned with the OpenAPI spec and tested in CI (negative test per cell). [REC]

## 6. Versioning policy

| Rule | Detail | Label |
|---|---|---|
| SemVer | Contract `MAJOR.MINOR.PATCH`; major in URL (`/v1`); `contract_version` echoed in every response and event | file 04 |
| Compatible (minor) | Add optional request fields, response fields, endpoints, enum values (consumers must tolerate unknown enum values; schemas mark extensible enums) | [REC] |
| Breaking (major) | Remove/rename fields, change types/semantics, make optional required, tighten validation, change error codes | [REC] |
| Parallel majors | `/v2` served alongside `/v1` during deprecation window | [REC] |
| Deprecation | `Deprecation` + `Sunset` headers (RFC 8594/9745); window proposal: 2 host release cycles or 6 months, whichever longer | [MANAGER] |
| Separate version axes | Contract, policy packs, models, catalogue, config are versioned independently and reported in `versions` / `policy` | [REC] |
| Policy pack change | Never a contract change; may change outcomes - announced via `policy.pack.activated` event | file 14 |
| CI gates | OpenAPI breaking-change diff (e.g. oasdiff) blocks merges; schema linter for forbidden credit-decision terms; examples validated against schemas | [REC] |
| Consumer-driven contract tests | Pact (or equivalent) once a real host consumer exists; until then host-simulator expectations act as the consumer contract | file 03/04 [REC] |

## 7. Idempotency, pagination, rate limiting

- **Idempotency:** `Idempotency-Key` required on all POST/PUT that create or ingest; stored 24 h [REC] with request hash; replay returns original response; different hash -> 409. Ingest rows also dedupe by `(account_token, source_txn_id)` (file 07). Evaluate calls are deterministic for identical inputs + versions, so retries are safe.
- **Pagination:** cursor-based (`?limit=` max 500, `cursor=` opaque, response `next_cursor`); stable ordering by `(created_at, id)`; audit export streams NDJSON with cursor. No offset pagination. [REC]
- **Rate limiting:** per client id token bucket; defaults proposal 50 rps evaluate, 5 concurrent bulk ingests; headers `RateLimit-Limit/Remaining/Reset` (IETF draft) + `Retry-After` on 429; limits configurable per host [MANAGER].
- **Size limits:** 5 MB JSON per request, 5,000 transactions per evaluate, bulk parts 100 MB [REC] [ASSUME].

## 8. PII handling at the boundary

| Rule | Detail | Label |
|---|---|---|
| Tokens only | `customer_token`, `account_token`, consent refs - host tokenisation service owns mapping; no names, SSN/ITIN, full address, phone, email, DOB (birth year-month or host-computed `age_status` only) | [DOC p.8, p.16, p.74] |
| Coarse location | Jurisdiction codes only (e.g. `US-NY-NYC`), never street/ZIP unless legal approves ZIP for a specific use (ZIP is on the proxy list, file 14 §13.2) | [REC] [LEGAL-CONFIRM] |
| Transaction descriptions | Host sends `merchant_ref`/MCC; free-text description optional, redacted, treated as untrusted (prompt injection) | file 08 PRV-05, SEC-13 |
| Minimisation | Per-activity field allow-list (`CTL-DATA-MINIMISE`); extra fields rejected in strict mode or dropped and counted in lenient mode | [REC] |
| Sensitive attributes | Request schemas have no fields for excluded attributes; if detected in extension bags -> 400 | file 14 |
| Logs | Correlation ids only; no payloads | file 08 SEC-08 |
| Responses | Return tokens; never re-identify | [REC] |

## 9. Outbound events (CloudEvents 1.0)

Transport deferred (outbox table; host bus later [UNKNOWN]). `source = /retail-intel/{deployment_id}`, `subject = customer_token` where applicable, `datacontenttype = application/json`, extension attributes `contractversion`, `correlationid`, `decisionid`, `jurisdiction`. Payloads carry ids and summaries; consumers fetch details via API. [REC]

| type | When | Payload (summary) |
|---|---|---|
| `com.retailintel.insights.updated.v1` | Profile/features recomputed | customer_token, result_id, as_of |
| `com.retailintel.segments.changed.v1` | Membership added/ended | segment_code, change, purpose |
| `com.retailintel.churn.band_changed.v1` | Churn band crosses threshold | old/new band, model_version |
| `com.retailintel.needs.detected.v1` | New need above threshold | need_code, score band |
| `com.retailintel.recommendation.proposed.v1` | New suggestion (incl. CREDIT_REFERRAL) | rec_id, kind, product_code, requires_human_review, allowed_channels |
| `com.retailintel.recommendation.suppressed.v1` | Suppressed by policy | rec_id, reason_codes |
| `com.retailintel.retention.action_suggested.v1` | Action suggested | action_id, priority |
| `com.retailintel.retention.outcome_recorded.v1` | Retention outcome reported | action_id, outcome, outcome_source |
| `com.retailintel.review.item_changed.v1` | Human review item changed state | review_item_id, item_type, item_id, state, reason_code |
| `com.retailintel.consent.applied.v1` | Consent withdrawal processed, results recomputed | consent_ref, affected result ids |
| `com.retailintel.policy.pack.activated.v1` | New pack version live (`ACTIVE`, or `ILLUSTRATIVE` in a standalone demo) | pack ids/versions, `pack_state`, activation_id |
| `com.retailintel.policy.review.required.v1` | PENDING_LEGAL_REVIEW hit or review_due reached (internal/compliance consumers) | policy_id, version, count |
| `com.retailintel.policy.overlay.rejected.v1` | Host overlay loosening rejected | overlay_id, rejected controls |
| `com.retailintel.model.status_changed.v1` | Registry entry status change: `RULESET` ACTIVE/RETIRED, `ML_MODEL` SHADOW/APPROVED/RETIRED, fallback (governance event; outside the linter's output scope, §3.3) | model_version, kind, status |
| `com.retailintel.ingest.completed.v1` | Bulk batch committed | batch_id, counts, rejects |

No event type for credit decisions exists (same linter as §3.3). Customer-outcome event payloads carry `permitted_use` and `not_for_credit_decisioning: true` (IC7).

## 10. Inbound data adapters (host -> us)

Host provides; `host_data_mapper` translates host shapes to canonical model, validates, and records lineage (`ingest_batch`). [REC]

| Feed | Canonical entity | Key validations | Mode |
|---|---|---|---|
| Customers | `customer` (token, birth year-month or age_status, jurisdiction context, relationship start, vulnerability flag ref) | Token format; jurisdiction code exists in reference data; no PII fields | Payload / bulk |
| Accounts | `account` | Type enum, ISO 4217, ownership link | Payload / bulk |
| Transactions | `transaction` | Decimal amount, currency, status enum, dedupe key, account exists, booking_ts within window | Payload / bulk / events later |
| Consent | `consent_record` | Purpose in taxonomy, source, timestamps ordering; withdrawal triggers recompute | API push |
| Preferences / contact limits | `contact_preference` | Channel enum; limits numeric | API push |
| Product catalogue | `product` (code, jurisdictions offered, is_credit, min_age, eligibility_rule_ref, terms_ref, approved_by) | `jurisdictions` codes valid; credit products flagged; terms never generated by us | Bulk / API |
| Host policy overlay | `HOST_OVERLAY` (file 14) | Signature, schema, controls exist, stricter-only merge | API |
| Credit signals | see §11 | [OPEN] | [OPEN] |

Mapping specs are versioned YAML (`mappings/host-x/v1.yaml`: source field -> canonical field, transforms, required flag). Rejects go to a validation report, never silently corrected; data gaps are flagged, not imputed by guess (file 06 M1). [REC]

## 11. Candidate credit-platform integration points (all [OPEN])

**This is the canonical list (CI-1..CI-9).** File 07 §9 and file 12 reference it; former IDs: 07 O1-O9 = CI-1..CI-9 in the same order; the former 15 O-numbering is superseded.

| # | Candidate | Direction | Contract placeholder | PDF ref | Status |
|---|---|---|---|---|---|
| CI-1 | Income segments S0-S5 + overlay tags (GENZ, THIN_FILE, GIG_SELF_EMPLOYED, ...) | Credit -> us, read-only attribute | `subject.host_segments[]` `{code, source:"HOST", origin:"CREDIT_PLATFORM", as_of}` (file 07 `segment_membership.origin`) | [DOC p.60-61, p.78-81] | [OPEN] interface undefined |
| CI-2 | Early-warning / watch-list status; "pause offers" flag | Credit -> us (suppress offers only) | `preferences.suppressions[]` `{type:"PAUSE_OFFERS", source:"HOST", origin:"CREDIT_PLATFORM"}` -> `CTL-MARKETING-SUPPRESS`; never a feature | [DOC p.32, p.68-69] | [OPEN] |
| CI-3 | Shared bank-account transaction feed / event stream | Platform -> both | Same transaction schema via events | [DOC p.53] | [OPEN] |
| CI-4 | Consent records and tokenisation service | Platform -> us | Consent ingest (§2) | [DOC p.8, p.56, p.74] | [OPEN] |
| CI-5 | Vulnerability flags and contact limits | Platform -> us | Preference ingest | [DOC p.17, p.41, p.54] | [OPEN] |
| CI-6 | Under-18 readiness record | Unclear ownership | None; we do not build or consume it in MVP | [DOC p.14, p.66-67] | [OPEN] [MANAGER] - not built in MVP |
| CI-7 | Next-best offers / Gen Z loyalty overlap (platform Release 3) | Both (overlap risk) | Ownership decision first | [DOC p.46, p.70] | [OPEN] [MANAGER] |
| CI-8 | Credit product eligibility pre-check for recommendations | Credit -> us | Not defined; until it is, credit products appear only as `CREDIT_REFERRAL` ("refer to the credit platform for assessment"), never as pre-approved | none | [OPEN] [MANAGER] |
| CI-9 | Daily/steady PD or grade | Credit -> us | **Not consumed in v1**; any future use requires manager approval, legal confirmation and a fairness review, and never as a ranking feature for credit products | [DOC p.68] | [OPEN] [MANAGER] [LEGAL-CONFIRM] |

Not a candidate (already defined outbound path): **credit referral handoff** (us -> credit platform/host) = the `CREDIT_REFERRAL` recommendation kind (§3.3) plus the `recommendation.proposed` event; the credit platform alone assesses and decides. Only the target system is [UNKNOWN].

## 12. Data-exchange boundary

| Crosses the boundary (host <-> us) | Direction | Never crosses |
|---|---|---|
| Customer/account tokens, age_status or birth year-month, coarse jurisdiction codes | In | Names, SSN/ITIN, full address, phone, email, full DOB |
| Transactions (amount, currency, status, merchant_ref, MCC, redacted description) | In | Card PAN, account numbers (only tokens), raw free text without redaction |
| Consent records, preferences, contact limits, vulnerability flag refs | In | Vulnerability details/notes; our inferred vulnerability (we never infer) |
| Product catalogue with bank-approved eligibility refs and terms refs | In | Terms or eligibility authored by us or the LLM |
| Host policy overlay (signed) | In | Unsigned/looser overrides (rejected) |
| Insights, segments (with purpose), needs, churn score/band, recommendations (suggestion/referral), retention suggestions, explanations, reason codes - all labelled `permitted_use` + `not_for_credit_decisioning: true` (IC7) | Out | Any credit approval/decline, limit, pricing, adverse-action content; any licence to use outputs for credit decisions |
| Recommendation, retention and human-review outcomes | In | Customer free-text complaints or notes (fixed reason codes only) |
| Policy decision ids, pack versions, obligations | Out | Sensitive attributes, proxy features used for fairness testing, raw feature vectors (unless host is approved [MANAGER]) |
| Audit/decision-log export (hash-chained, tokenised) | Out | Prompts containing PII (none exist), model artefacts, training data |
| CloudEvents (ids + summaries) | Out | Full customer payloads in events |
| - | - | Cross-deployment data (e.g. future international deployment) [LEGAL-CONFIRM] |

## 13. Host simulator and contract testing

| Component | What it does | Label |
|---|---|---|
| Host simulator (`host-simulator/`) | Separate app acting as the host: test JWKS issuing tokens with scopes + `ri_deployment` + `ri_jurisdictions`; sends host-shaped payloads through the mapper; pushes consent/preference changes; supplies a signed (and a deliberately looser) policy overlay; consumes outbox events | file 04 [REC] |
| Scenario library | NYC adult happy path; consent withdrawn mid-session; minor (tier-1 and tier-2 activities per file 14 §13.3, with and without a `MINOR-APPROVAL-*` record); age unknown; retention outcome reported; human review cleared / rejected / expired; response without IC7 labels rejected by the contract test; jurisdiction unknown; `US-CA` out of scope; channel not allowed; PENDING_LEGAL_REVIEW policy; looser overlay; credit approval request to assistant; `ILLUSTRATIVE` pack offered to the plugin deployment (refused at load, not ready); responses carry `illustrative: false`; ML model in SHADOW never appears in any response; duplicate idempotency key; expired token; wrong `ri_deployment`; oversized payload; malformed transactions | [REC] |
| Contract tests | Schemathesis property tests against OpenAPI in **both** modes; golden request/response examples in `contracts/examples/` validated in CI; event payloads validated against CloudEvents + data schemas | [REC] |
| Equivalence test | Same inputs via standalone adapter and via simulator plugin path -> identical results and decision outcomes (file 09 success criterion). Both wirings receive the same signed fixture bundle through a test-only `PolicyStorePort` double (CI seam, not a deployment adapter), because a plugin deployment refuses `ILLUSTRATIVE` packs (DEC-44). The fixture is shaped as `ACTIVE` (so `illustrative: false`), signed with a `TEST`-class key that only the test double trusts, and contains synthetic placeholder policies only | [REC] |
| Consumer-driven contracts | Simulator expectations exported as Pact-style files now; replaced by real host consumer pacts later | [REC] |
| Breaking-change gate | OpenAPI diff + forbidden-term linter + example validation | [REC] |
| Security tests | Scope/jurisdiction claim mismatch, PII field injection, prompt injection in descriptions | file 08 [REC] |

## 14. Open items

File 12 is the single decision register; each item below is cross-referenced there (DEC ID assigned in file 12).

| # | Item | Owner | Register |
|---|---|---|---|
| I1 | Host IdP, claim names, mTLS requirement | [UNKNOWN] HOST | see file 12 (formerly 15-I1); DEC ID assigned in file 12 |
| I2 | Deprecation window and rate limit defaults | [MANAGER] | see file 12 (formerly 15-I2); DEC ID assigned in file 12 |
| I3 | Stateless vs stateful first (proposal: stateless evaluate) | [MANAGER] | see file 12 (formerly 15-I3); DEC ID assigned in file 12 |
| I4 | Event transport and whether host wants events in MVP | [UNKNOWN] | see file 12 (formerly 15-I4); DEC ID assigned in file 12 |
| I5 | Whether ZIP/fine location may ever cross the boundary | [LEGAL-CONFIRM] | see file 12 (formerly 15-I5); DEC ID assigned in file 12 |
| I6 | DSR handling scope per jurisdiction; whether crypto-shredding counts as deletion | [LEGAL-CONFIRM] | see file 12 (formerly 15-I6); DEC ID assigned in file 12 |
| I7 | All credit-platform points CI-1..CI-9 (§11) | [OPEN] [MANAGER] | see file 12 (formerly 15-I7); DEC ID assigned in file 12 |
| I8 | Wording of the host integration terms prohibiting credit use of outputs (IC7) | [LEGAL-CONFIRM] [MANAGER] | new in validation pass (formerly 15-I8); see file 12 DEC-42 |
| I9 | GATE-3 contract gaps G-RS-1 (portfolio), G-RS-2 (customer list), G-RS-3 (stored transactions) | DEV | **DECIDED 2026-11-10** (DEC-62): endpoints in §2; Story 9.7 stays Could |
| I10 | GATE-3 integration owner (file 11 [UNKNOWN]) | MGR | **DECIDED 2026-11-10, interim** (DEC-63): the developer, until a host platform owner exists |
| I11 | Deployment profile in the contract (track, institution type, cell) | DEV | **DECIDED 2026-11-10** (DEC-61): read-only in `/v1/capabilities`; never a request field |

## Change log (validation pass 2026-11-09)

- §1: principles C1-C6 renamed IC1-IC6 (avoids clash with file 13 rows C01-C07) (D-06); new **IC7 not for credit decisioning** (purpose limitation, `permitted_use`, `not_for_credit_decisioning`) (D-11).
- §2: composite evaluate row added and Recommendations activity cell completed (D-23); feedback value `DECLINED` -> `DISMISSED_BY_CUSTOMER` (D-05); new retention outcome endpoint with scope `ri.retention.feedback` (D-09, G-9); human review queue endpoints and plugin review outcomes (G-1, D-29); DSR row marked placeholder with crypto-shredding method (G-4); minors note referencing the two-tier list in file 14 §13.3 (D-01).
- §3.2: IC7 fields added; `APP` -> `MOBILE_APP` (D-19); `review_item_id`; enum notes incl. `protection_status` `ADULT|MINOR_PROTECTED|AGE_UNKNOWN` (D-20).
- §3.3: linter scoped to customer-outcome output types with an explicit governance-schema allow-list - same choice as files 07 and 14 (D-05).
- §4: `DATA_QUALITY_INSUFFICIENT` linked to file 07 §4.1 rules (G-6).
- §5: scopes `ri.retention.feedback`, `ri.review.read`, `ri.review.write`; role `policy_release` (D-24); new §5.1 role-to-scope and screen matrix (G-2, D-30).
- §9: retention outcome and review events; model status event marked as governance; IC7 labels on events.
- §11: canonical credit-integration list **CI-1..CI-9** incl. the under-18 readiness record; source `HOST` + `origin: CREDIT_PLATFORM` (D-44); PD/grade wording per D-37; referral handoff moved out of the candidate list as an already-defined outbound path (D-04).
- §12, §13: IC7 labels and outcome flows at the boundary; scenario library extended (minors two-tier, outcomes, review states, IC7 contract test).
- §14: I1-I7 cross-referenced "(formerly 15-In); DEC ID assigned in file 12" (D-03); I7 now CI-1..CI-9; new I8 (host terms wording).

## Change log (readiness fixes 2026-11-09)

- §2: capabilities, `/ready` and pack read rows follow the pack lifecycle (file 14 §8.4; DEC-45). New §2.1 governance, simulator, decision-log and model-registry endpoints (DEC-46; closes BMAD readiness R-3 at sketch level; normative at P-4/GATE-3).
- §3.2: `illustrative`, `watermark`, `policy.bundle_state` (DEC-44). §4: governance 409/422 codes. §5: scopes `ri.policy.author`, `ri.policy.approve`, `ri.policy.release`, `ri.policy.simulate`, `ri.models.read`, `ri.models.write`; §5.1 rows mapped to them.
- §9: `pack_state` on `policy.pack.activated`; `model.status_changed` covers `RULESET`/`ML_MODEL` and SHADOW (DEC-47). §13: illustrative-refusal and shadow scenarios.
- After the readiness re-run: §2.1 activation records confirmed assumptions; simulation emits `policy.review.required`; §5.1 demo-issuer note (no `policy_approver` user; manager acts through role users); §13 parity fixture bundle (ACTIVE-shaped, TEST-class key, test-port double).

## Change log (PH-3 contract specification 2026-11-09)

- Header: the normative contract is now `contracts/openapi/retail-intelligence-v1.yaml` (OpenAPI 3.1, 42 operations covering §2 and §2.1) plus `contracts/events/*.schema.json` (CloudEvents 1.0, 15 event types); it is the source of truth and wins over the sketches here (closes BMAD readiness P-4 for the contract part). Lint: Redocly CLI 1.34.5, 0 errors / 0 warnings.
- §4: `REVIEW_STATE_INVALID` added for invalid review-item transitions (409).
- Contract decisions not visible in the sketches: `X-Not-For-Credit-Decisioning: true` header on every response and the label on error bodies; `ReviewerRole` (`analyst`, `relationship_staff`) in customer-outcome schemas so no governance role name reaches the forbidden-term linter's scope; consent purposes, activities and jurisdiction codes are pattern-checked reference data (market-neutral, DEC-50); `X-Content-SHA256` on bulk parts; review results and customer outcomes on the outcome endpoints are a `report_type` discriminated union.

## Change log (two-track scope 2026-11-10)

- Two-track scope (file 21; file 12 §H.7): §2 capabilities row adds read-only `deployment_profile` (DEC-61); three new rows `GET /v1/customers`, `GET /v1/customers/{customer_token}/transactions`, `GET /v1/portfolio/summary` (DEC-62); tracks-and-cells note with new reason codes and locales. §5.1 points to `contracts/role-scope-matrix.*` as normative, adds `ri.transactions.read`, `ri.portfolio.read`, role `sharia_approver` and the interim role approvers (DEC-55, DEC-63). §14 I9-I11 recorded as decided (G-RS-1..3, interim integration owner = the developer, deployment profile).
- 2026-11-10 (independent two-track validation): tracks-and-cells note extended with the token scheme, portfolio disclosure control and profile integrity tightenings (file 21 §2.2, §3.1, §10A; file 12 §H.7.3); contract 1.1.2.
- 2026-11-10 (contract 1.2.0, V-8, V-9): §2 customer-list row now the purpose-bound `GET /v1/customer-directory` under `ri.customers.list` (DEC-68; file 21 §2.3); `listCustomers` deprecated and not implemented. Recommendations in ISLAMIC deployments carry the SDC-11 `sharia_disclosure` block (file 21 §4.4); `contracts/` is normative.
