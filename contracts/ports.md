# Adapter port list (PH-3)

**Status:** normative port list, PH-3 (readiness P-4; planning/10 S3.3 adapter interfaces and S3.4 credit-boundary placeholder), 2026-11-10. It is a planning artefact: interfaces are named and specified here, but no code exists.

**Sources:**
- planning/04 §4 (layout and ports, labelled "illustrative" there) and §7 (host simulator).
- Spine AD-1, AD-2, AD-10, AD-12, AD-14..AD-20 and the Structural Seed.
- planning/15 §10, §11 and §13.
- `contracts/openapi/retail-intelligence-v1.yaml` (47 operations, contract 1.2.0).
- `contracts/data-model/erd.md` (67 entities).
- planning/21 (two-track deployment design) and planning/12 §H.7 (DEC-51..DEC-64).

**Authority.** This file supersedes the illustrative table in planning/04 §4, which is read-only. It extends the spine's 12 ports with 6 ports that the contract and the ERD need; see §4.

## 1. Rules

- `core/` imports only `core/` and `ports/`. Adapters implement ports, and only `bootstrap.py` wires them (AD-2). Mode differences are wiring and YAML only; no `if MODE` appears in core (AD-1).
- **Direction.**
  - *Driving (inbound)* ports are the use-case interfaces in `core/services`. Inbound adapters call them: `http_api` (FastAPI), `cli_batch`, and later `event_consumer`.
  - *Driven (outbound)* ports are interfaces that core calls and outbound adapters implement.
- Every driven port has one adapter test suite. Both the standalone adapter and the plugin adapter must pass it (planning/04 §7).
- **Entity ownership.** Each ERD entity is written through exactly one driven port (§3). Other ports and modules read it only through that port's read operations.
- **Market-neutral (DEC-50, DEC-51).** No port operation takes or returns a jurisdiction, track or institution literal. Jurisdiction codes, track and cell codes, purposes and reason codes are reference data; `institution_type` is read only from `DeploymentProfilePort`. One set of ports serves every track; each data cell wires its own adapter instances (database, keys, logs) and no adapter reaches another cell or track (DEC-52, DEC-53).

## 2. Driving (inbound) ports - use cases

Every authenticated operation passes through `IdentityPort.verify` and the role-scope matrix (`contracts/role-scope-matrix.json`) in the inbound adapter before it reaches the use case. Idempotency (24 h) and rate limits are inbound-adapter concerns in `http_api`. Their store, `idempotency_record`, is owned by that adapter and has no core port.

| Use-case port (`core/services`) | Operations (OpenAPI `operationId` or CLI) | Owning module | Inbound adapter |
|---|---|---|---|
| `MetaQuery` | `getHealth` (adapter-only liveness), `getReadiness`, `getCapabilities` | M8 | http_api |
| `EvaluateCustomer` | `evaluateCustomerInsights`, `getStoredCustomerInsights`, `evaluateCustomerComposite` | M1 (composite orchestrated in services across M1-M7) | http_api |
| `CustomerDirectory` | `searchCustomerDirectory` (contract 1.2.0, DEC-68: scope `ri.customers.list`, registered access purpose, exact lookup or narrowing filter, page <= 25, query total <= 100, per-user quotas; every call, allowed or denied, appends an `audit_record` with an access block through `AuditSinkPort`; tokens only; ROW_FILTER). `listCustomers` (DEC-62) is deprecated and not implemented: no adapter routes it | M1 | http_api |
| `EnrichTransactions` | `enrichTransactions`, `getStoredCustomerTransactions` (purpose `INSIGHT`, enriched view only; DEC-62) | M2 | http_api |
| `QuerySegments` | `getCustomerSegments` | M3 | http_api |
| `QueryPredictions` | `getCustomerChurnScore` (M5), `getCustomerNeeds` (M4) | M4, M5 | http_api |
| `GenerateRecommendations` | `generateRecommendations` | M6 | http_api |
| `QueryRetentionActions` | `getRetentionActions` | M7 | http_api |
| `RecordOutcome` | `reportRecommendationOutcome` (M6), `reportRetentionOutcome` (M7) | M6, M7 | http_api |
| `ReviewQueue` | `listReviewItems` (access audit record per call, contract 1.2.0), `transitionReviewItem` | M6 / M7 (CTL-HUMAN-REVIEW) | http_api |
| `PortfolioSummary` | `getPortfolioSummary` (aggregate-only, small-cell suppression; DEC-62) | services (read-only aggregation over `ResultStorePort`) | http_api |
| `ExplainResult` | `getExplanation` | M9 | http_api |
| `AssistantQA` | `postAssistantMessage` | M9 | http_api |
| `DecidePolicy` | `queryPolicyDecision` | M8 | http_api; also called in-process by every module PEP (`decide`, `post_check`) |
| `DecisionLogQuery` | `getDecisionLogRecord`, `replayDecision`, `exportAuditRecords` (auditor read path; the export itself is audited with an access block, contract 1.2.0) | M8 | http_api |
| `PackGovernance` | `listPolicyPacks`, `uploadPolicyPack`, `getPolicyPackStatus`, `validatePolicyPack`, `submitPolicyPack`, `recordPackApproval`, `recordPackShariaApproval` (ISLAMIC deployments only; DEC-55), `activatePolicyPack` | M8 | http_api |
| `PolicySimulation` | `startPolicySimulation`, `getPolicySimulation` | M8 | http_api |
| `HostOverlay` | `putHostOverlay` | M8 | http_api (plugin wiring only) |
| `ModelRegistryAdmin` | `listModels`, `getModelVersion`, `transitionModelVersion` | M5 | http_api |
| `IngestConsentPreferences` | `batchUpsertConsents`, `batchUpsertPreferences` | M8 | http_api |
| `BulkIngest` | `createIngestBatch`, `getIngestBatch`, `putIngestBatchPart`, `commitIngestBatch` | M2 | http_api |
| `DataSubjectRequest` | `createDsrRequest` (placeholder in v1) | M8 | http_api |
| `RunNightlyBatch` | CLI `batch run --as-of` | services (M1-M7 in order M2, M1, M3, M4/M5, M6, M7) | cli_batch |
| `RetentionPurge` | CLI `retention purge --report-only` (report-only until DEC-20) | M8 | cli_batch |
| `GenerateSyntheticData` | CLI `synthetic generate --seed` (standalone only) | Epic 1 generator (outside core modules) | cli_batch |

## 3. Driven (outbound) ports

Test doubles: an in-memory fake exists for every port for unit tests. The column below names the double that matters for contract and parity tests (planning/15 §13).

| # | Port | Operations (logical) | Standalone adapter | Plugin adapter | Test double / host simulator | Owning module | Entities written (ERD) |
|---|---|---|---|---|---|---|---|
| 1 | `IdentityPort` | `verify(token) -> Principal{sub, act, roles, scopes, ri_deployment, ri_jurisdictions}`; `issuer_metadata()` | `jwt_local_issuer` (demo issuer, one user per role except `policy_approver`, claims per the role-scope matrix §3.2) | `oidc_validator` (host JWKS, claim mapping as config; mTLS optional; DEC-30 [UNKNOWN]) | Host simulator test JWKS: valid, expired, wrong `ri_deployment`, out-of-claim and forged-scope tokens | M8 | none |
| 2 | `ReferenceDataPort` | `jurisdiction_tree()`; `deployment()` (scope, mode, `demo`); `consent_purposes()`; `dq_thresholds()`; `reason_codes()`; `segment_definitions()`; `fx_rate(date, base, quote)`; `config_version(kind)`; `templates(kind, locale)` (contract 1.2.0: `message_template`, both language twins, Sharia state in ISLAMIC deployments); `access_purposes(role, client_id)` (DEC-68 registry; deny when absent) | `postgres_repo` over `config/reference/*`, maker-checker `config_version` | Same, or a host reference feed for catalogue and FX [UNKNOWN] | Fixture config sets: DEC-16 test jurisdictions as data (extended in planning/12 §H.7), empty pack set | M8 (config governance) | deployment, jurisdiction, jurisdiction_membership, config_version, segment_definition, fx_rate, message_template |
| 3 | `CustomerDataPort` | `get_customer(token, as_of)`; `upsert_customers(rows)`; `accounts(token)`; `goals(token)`; `delete_subject(token)` | `postgres_repo` loaded by `synthetic_source` (rows must have `is_synthetic=true`, AD-19) | `host_data_mapper`: evaluate payload or bulk ingest, mapped to canonical rows; host pushes, we never pull (AD-12) | Host simulator sends host-shaped payloads through the mapper | M1 | customer, account, account_holder, financial_goal |
| 4 | `TransactionDataPort` | `window(token, from, to)`; `upsert(rows)` (idempotent on `dedupe_key`); `link(refund, reversal, transfer)`; `categorise(rows)`; `delete_subject(token)` | `postgres_repo` + `duckdb_batch` (Parquet working sets) | `host_data_mapper` (payload or bulk); event feed later (CI-3 [OPEN]) | Edge-case fixtures (duplicates, pending, reversals, FX); malformed rows | M2 | transaction, transaction_category |
| 5 | `BulkStagingPort` | `open(batch)`; `put_part(batch, n, sha256, stream)`; `commit(batch)`; `status(batch)`; `purge_staging(batch)` | Local staging directory + `postgres_repo` | Same (the batch API is ours in both modes) | Oversized and corrupt parts; digest mismatch on a committed part (409) | M2 | ingest_batch, ingest_batch_part |
| 6 | `ProductCatalogPort` | `catalogue(version)`; `products_offered(jurisdiction_set, as_of)`; `upsert(products)` (ISLAMIC: rejects `CONVENTIONAL_*` structures and incomplete SDC-11 disclosure references; CONVENTIONAL: rejects any Sharia column); `disclosure(product_code, catalogue_version)` (contract 1.2.0 `ShariaDisclosure` refs, ISLAMIC only) | Synthetic demo catalogue ("SYNTHETIC - DEMO", `jurisdictions_offered`) | Host catalogue via bulk ingest `PRODUCT_CATALOGUE` or a feed [UNKNOWN] | Catalogue fixtures, including credit products (referral only) | M6 | product |
| 7 | `ConsentPort` | `consent(token, purpose, source, as_of)`; `upsert_consents(rows)`; `preferences(token)`; `upsert_preferences(rows)`; `suppressions(token)`; `on_withdrawal(callback)` | Synthetic consent and preference records | Host consent service via `POST /v1/consents:batchUpsert` and `/v1/preferences:batchUpsert` (push; CI-4, CI-5) | Host simulator withdraws consent mid-session; unknown purpose; expired consent | M8 | consent_record, contact_preference, contact_suppression |
| 8 | `CreditBoundaryPort` (S3.4 placeholder, [UNKNOWN], optional in v1) | `host_segments(token, as_of) -> [HostSegment]` (CI-1); `offer_suppressions(token) -> [Suppression]` (CI-2). Read-only. **Never** PD/grade (CI-9) and never an eligibility check (CI-8) | Synthetic stub values only (DEC-03 DECIDED-DEFAULT, GATE-3) | Fields carried in the payload (`subject.host_segments[]`, `preferences.suppressions[]` with `origin: CREDIT_PLATFORM`) through `host_data_mapper`; no live feed in the MVP | Stub returning fixed tags and a PAUSE_OFFERS flag; empty stub | M3 (segments) with M6/M7 consumers | none (segments are stored through `ResultStorePort`, suppressions through `ConsentPort`) |
| 9 | `ResultStorePort` | `save(module_result)` (requires `decision_id`, AD-4); `latest(token, kind)`; `record_outcome(...)` (append-only); `review_queue(filter)`; `transition_review(...)`; `delete_subject(token)` | `postgres_repo` | `postgres_repo` (same; in plugin mode the host may run the review queue and report results through the outcome endpoints) | In-memory store; review SLA clock driven by `ClockPort` | M1-M7 (each module writes only its own rows; planning/06) | feature_snapshot, segment_membership, prediction, recommendation, recommendation_prediction, recommendation_outcome, retention_action, retention_outcome, review_item, review_transition, explanation, data_quality_result |
| 10 | `ModelRegistryPort` | `serving(activity) -> entry` (RULESET ACTIVE or ML_MODEL APPROVED only); `list()`; `get(id, version)`; `transition(id, version, target, mrm_ref)`; `write_shadow(prediction)` (synthetic only) | Local registry (`postgres_repo`) + artefact store | Local (same) | Registry fixtures: RULESET ACTIVE baseline, ML_MODEL in SHADOW, APPROVED refused (DEC-19) | M5 (M4 consumer) | model_version, model_status_change, shadow_prediction |
| 11 | `PolicyStorePort` | `runtime_bundle()` (verified L0 + ACTIVE, or L0 + ILLUSTRATIVE; never mixed); `verify(pack)`; `put_pack`, `validate`, `submit`, `approve`, `sharia_approve` (ISLAMIC only; legal and Sharia approval both needed for APPROVED), `activate`; `pack_status`; `simulate(candidates)`; `put_overlay(overlay)` | `policy_store_local`: signed packs; DEMO key only where `demo=true`; ILLUSTRATIVE only when `mode=STANDALONE` and `demo=true` (AD-18) | `policy_store_local` + `policy_store_host_overlay` (host-signed, tighten-only merge, AD-8); refuses ILLUSTRATIVE at load | **Test-only double** with an ACTIVE-shaped fixture bundle signed by a TEST-class key that only the double trusts (parity seam, planning/15 §13); a looser overlay for rejection tests | M8 | policy, policy_version, legal_source, policy_source, control, policy_control, policy_approval, policy_change_event, sharia_approval, applicability_record, minor_approval_record, policy_pack, pack_member, pack_state_change, policy_test_case, exclusion_list, exclusion_entry, pack_activation, host_overlay, policy_simulation |
| 12 | `DecisionLogPort` | `append(decision)` (hash-chained, insert-only); `get(decision_id)`; `replay_inputs(decision_id)` (via `SubjectKeyPort`); `export(cursor)`; `raise_conflict(...)`; `verify_chain()` | Local append-only table | Local table + export to the host audit sink (`audit_host_forwarder`) | Chain-tamper fixture; erased-subject replay (`SUBJECT_ERASED`) | M8 | policy_decision_log, decision_policy_hit, decision_obligation, policy_conflict, replay_snapshot |
| 13 | `AuditSinkPort` | `append(audit_record)` (incl. the contract 1.2.0 access block for directory, queue and export calls, allowed or denied); `append_llm(interaction)`; `export(cursor)`; `verify_chain()` | `audit_pg` (append-only table) | `audit_pg` + `audit_host_forwarder` [UNKNOWN host sink] | Chain verification after restore (SEC-20) | M8 | audit_record, llm_interaction |
| 14 | `EventPublisherPort` | `enqueue(cloud_event)` in the business transaction (AD-20); `dispatch()` | Outbox table; the UI polls | Outbox -> host bus [UNKNOWN] (DEC-33); no broker in the MVP | Host simulator consumes the outbox; schema validation against `contracts/events/` | services (shared) | event_outbox |
| 15 | `LLMPort` | `render(template_id, tool_results)`; `generate(prompt, tools)` (disabled); `validate(output, tool_results)` | `llm_template` (only adapter enabled, AD-14, DEC-06) | `llm_template`; a bank-approved endpoint only after a separate MGR/SEC decision | Template adapter; number-mismatch fixture forces the template fallback | M9 | none (interactions are written through `AuditSinkPort`) |
| 16 | `SubjectKeyPort` | `key_for(subject_ref)`; `encrypt(subject_ref, bytes)`; `decrypt(subject_ref, bytes)`; `destroy(subject_ref, dsr_request_id)`; `state(subject_ref)`; `register_dsr(...)` | Dev KMS (local key table; demo only, no backup per DEC-36) | Bank KMS / HSM [UNKNOWN; platform-wide, spine Deferred] | In-memory KMS; destroyed-key fixture | M8 | subject_key, dsr_request |
| 17 | `ClockPort` | `now()`; `today(tz)` | System clock | System clock | Injectable fixed clock for replay, SLA expiry and `review_due` tests | services (shared) | none |
| 18 | `DeploymentProfilePort` (DEC-51, DEC-61) | `profile()` (signed, verified at bootstrap; read-only); `cells()`; `cell_for(jurisdiction_set)` (else `UNSUPPORTED_JURISDICTION` / `UNKNOWN`); `institution_type()`; `verify_artefact_track(artefact)` (packs, catalogue, models, templates) | `profile_file_signed` (config/profile, signature checked with the deployment or demo-class key) | Same; the host cannot override any field (no request input reaches this port) | Fixtures: Track A two-cell profile, Track B ISLAMIC profile, tampered signature, ISLAMIC without overlay pack, mismatched artefact track (all must fail closed) | M8 | deployment_profile, data_cell |

**Coverage.**
- All 67 ERD entities are owned: 66 by the 18 driven ports above and `idempotency_record` by the `http_api` inbound adapter.
- All 47 OpenAPI operations (contract 1.2.0) map to a use-case port in §2 (`listCustomers` is listed as deprecated and not implemented).
- The 12 spine ports are all present.

## 4. Differences from planning/04 §4 and the spine Structural Seed

| Port | Change | Why |
|---|---|---|
| `ReferenceDataPort` | New | AD-1 and AD-17 say config (the `demo` flag, consent purposes, DQ thresholds) is "read through ports", but no port was named |
| `ResultStorePort` | New | `postgres_repo` wrote derived rows with no port; AD-4's required `decision_id` is enforced at `save` |
| `BulkStagingPort` | New | Bulk ingest parts (Story 3.6, contract `putIngestBatchPart`) need staged storage; `duckdb_batch` had no port |
| `SubjectKeyPort` | New | Crypto-shredding (AD-10, DEC-35, planning/07 §8.2) needs key create, encrypt and destroy behind a port, so the bank KMS can replace the dev KMS |
| `CreditBoundaryPort` | New, optional, [UNKNOWN] | planning/10 S3.4 asks for a documented placeholder for reading CI-1 segments and tags and CI-2 suppressions (DEC-03: synthetic stubs only) |
| `ConsentPort` | Scope widened | It also carries contact preferences and suppressions (contract `/v1/preferences:batchUpsert`, Story 2.10) |
| `DeploymentProfilePort` | New (two-track update, 2026-11-10) | One signed profile per deployment decides track, institution type, cells and overlay (planning/21 §3); bootstrap must read it through a port so tests can inject each track's profile |
| `LLMPort` | Adapter set narrowed | `llm_local` and `llm_bank_endpoint` from planning/04 §4 stay unwired until the DEC-06 follow-up decision (AD-14) |

These are recorded in the spine change log (2026-11-10) and in implementation-readiness.md.

## Change log

- 2026-11-10: created (PH-3, readiness P-4; planning/10 S3.3, S3.4). 23 driving use-case ports covering all 42 operations; 17 driven ports (spine's 12 + 5 new), each with its standalone adapter, plugin adapter, test double, owning module and owned entities. The cross-check (`crosscheck.py`, scratch) passed: every operation is mapped, every entity has exactly one owner, and every spine port is present.
- 2026-11-10 (two-track scope, planning/21; planning/12 §H.7): new use-case ports `CustomerDirectory` (`listCustomers`) and `PortfolioSummary` (`getPortfolioSummary`); `getStoredCustomerTransactions` on `EnrichTransactions`; `recordPackShariaApproval` on `PackGovernance`; new driven port 18 `DeploymentProfilePort` owning `deployment_profile` and `data_cell`; `PolicyStorePort` owns `sharia_approval`. Now 25 use-case ports for 46 operations and 18 driven ports. `crosscheck.py` re-run: PASS.
- 2026-11-10 (contract 1.2.0; V-8, V-9; planning/12 DEC-68, §H.7.4): `CustomerDirectory` serves `searchCustomerDirectory` (purpose-bound, audited) and lists `listCustomers` as deprecated and not implemented; `ReviewQueue` and `DecisionLogQuery` note the access audit; `ReferenceDataPort` gains `templates(kind, locale)` and `access_purposes(role, client_id)` and owns the new `message_template`; `ProductCatalogPort` gains `disclosure(...)` and the SDC-11 load checks; `AuditSinkPort.append` carries the access block. Now 25 use-case ports for 47 operations, 18 driven ports, 67 entities. `crosscheck.py` re-run: PASS.
