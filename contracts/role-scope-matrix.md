# Role-scope matrix (draft for GATE-3)

**Status:** DRAFT 0.3.0, PH-3 (readiness P-4), 2026-11-09; two-track update 2026-11-10 (planning/21; planning/12 §H.7); purpose-bound customer directory 2026-11-10 (contract 1.2.0, V-8, DEC-68). DEC-38 is DECIDED (developer-delegated, 2026-11-10; manager countersignature recommended) for this GATE-3 draft, and SEC/MGR approve the final version at GATE-12. This is a planning artefact; no identity code exists.

**Machine-readable twin:** `contracts/role-scope-matrix.json`. Both files are generated together from one source, so the tables below and the JSON always agree. A check script confirms this and compares the scopes with the OpenAPI file.

**Sources:**
- planning/15 §5 (scope catalogue) and §5.1 (proposed matrix).
- planning/08 SEC-02 (roles), SEC-03 (claims) and SEC-19 (screens).
- Spine AD-16 (deployment scope) and AD-21 (deny by default, one negative test per cell).
- EXPERIENCE.md information architecture.
- planning/21 §2.3 and planning/12 DEC-68 (purpose-bound customer directory).
- Normative scopes: `contracts/openapi/retail-intelligence-v1.yaml` (`components.securitySchemes.oauth2`; 31 scopes, 47 operations, contract 1.2.0; 30 scopes and 46 operations in 1.1.0-1.1.2).

**Market-neutral (two tracks, DEC-50/DEC-51).** No jurisdiction or track code is hard-wired here. The same matrix applies to every deployment profile: Track A (`CONVENTIONAL`, cells `A-US` and `A-UK`) and Track B (`ISLAMIC`, cell `B-AE`). Each deployment has its own issuer realm (DEC-53), so a token from one track is never valid in the other (JX-1). Claim profiles refer to the deployment's reference data (`deployment_scope` and the DEC-16 test-jurisdiction set, extended in planning/12 §H.7). The role `sharia_approver` (DEC-55) exists only in `ISLAMIC` deployments; in `CONVENTIONAL` deployments the issuer creates no user with it, and `recordPackShariaApproval` returns 422 `OUT_OF_SCOPE_REQUEST` for any caller.

## 1. Rules

1. **Deny by default.** A role has only the cells marked `allow`. An endpoint without a declared scope does not exist, except the unauthenticated `GET /health`.
2. **Three checks, in this order:**
   - The token is valid: 401 `UNAUTHENTICATED`.
   - The scope is held: 403 `FORBIDDEN_SCOPE`.
   - The jurisdiction claims pass: 403 `JURISDICTION_NOT_PERMITTED`, or the row is rejected or filtered (§3).

   Only then does the PDP decide (consent, minors, policy). The order is itself tested; see the precedence tests in family ROLE_JURISDICTION.
3. **Screens follow scopes.** A screen is shown only when the role holds every scope that the screen requires. Each action on a screen also needs the scope of its operation. The dashboard is a plain `/v1` client, so a hidden screen is never the only control: the API refuses the call as well. `service_host` is a machine client and cannot open a dashboard session.
4. **Plugin mode.** The host maps its own roles to these scopes. The scope column is the contract; the staff role columns apply to the standalone issuer.
5. **Demo issuer** (planning/15 §5.1, DEC-11, DEC-63). It creates one user per role, except `policy_approver` and `sharia_approver`, which stay unassigned until a named person is recorded (interim approver roles per track: DEC-63). "Manager" is not a role: the manager drives the demo through the role users.
6. **Purpose-bound customer directory** (contract 1.2.0, DEC-68; planning/21 §2.3). `searchCustomerDirectory` needs the scope `ri.customers.list` (analyst and relationship_staff only) **and** an `access_purpose` registered for the role (Table 5) or, for a machine client, for its `client_id` in the issuer client registry (none registered). Check order after the scope: purpose (400 `ACCESS_PURPOSE_INVALID`, 403 `ACCESS_PURPOSE_NOT_REGISTERED`), narrowing filter (400 `NARROWING_FILTER_REQUIRED`), quota (429 `ACCESS_QUOTA_EXCEEDED`), then ROW_FILTER. Every call, allowed or denied, writes an `audit_record` with an `access` block. The auditor reads those records through `exportAuditRecords` and holds no directory scope; the deprecated `listCustomers` is not implemented (404 for every role; `deprecated_operation_tests` in the JSON).

## 2. Matrices

Cells are `allow` or `deny`. Every `deny` cell has exactly one negative test (§5).

### Table 1 - Roles x OAuth scopes

| Scope (jurisdiction check) | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host |
|---|---|---|---|---|---|---|---|---|---|---|
| `ri.meta.read` (DEPLOYMENT_ONLY) | allow | allow | allow | allow | allow | allow | allow | allow | allow | allow |
| `ri.insights.read` (REQUEST_CONTEXT) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.customers.list` (ROW_FILTER) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| `ri.transactions.enrich` (REQUEST_CONTEXT) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.transactions.read` (REQUEST_CONTEXT) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| `ri.segments.read` (REQUEST_CONTEXT) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.predictions.read` (REQUEST_CONTEXT) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.recommendations.generate` (REQUEST_CONTEXT) | deny | allow | allow | deny | deny | deny | deny | deny | deny | allow |
| `ri.recommendations.feedback` (STORED_SUBJECT) | deny | deny | allow | deny | deny | deny | deny | deny | deny | allow |
| `ri.retention.read` (REQUEST_CONTEXT) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.retention.feedback` (STORED_SUBJECT) | deny | deny | allow | deny | deny | deny | deny | deny | deny | allow |
| `ri.review.read` (ROW_FILTER) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.review.write` (STORED_SUBJECT) | deny | deny | allow | deny | deny | deny | deny | deny | deny | allow |
| `ri.explanations.read` (STORED_SUBJECT) | deny | allow | allow | deny | allow | deny | deny | deny | deny | allow |
| `ri.policy.query` (REQUEST_CONTEXT) | deny | allow | deny | allow | allow | allow | allow | allow | deny | allow |
| `ri.policy.read` (DEPLOYMENT_ONLY) | deny | allow | deny | allow | allow | allow | allow | allow | allow | allow |
| `ri.policy.author` (PACK_SCOPE) | deny | deny | deny | deny | deny | allow | deny | deny | deny | deny |
| `ri.policy.approve` (PACK_SCOPE) | deny | deny | deny | deny | deny | deny | allow | deny | deny | deny |
| `ri.sharia.approve` (PACK_SCOPE) | deny | deny | deny | deny | deny | deny | deny | deny | allow | deny |
| `ri.policy.release` (PACK_SCOPE) | deny | deny | deny | deny | deny | deny | deny | allow | deny | deny |
| `ri.policy.simulate` (PACK_SCOPE) | deny | deny | deny | deny | deny | allow | allow | deny | deny | deny |
| `ri.policy.overlay.write` (PACK_SCOPE) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.models.read` (DEPLOYMENT_ONLY) | deny | allow | deny | allow | allow | deny | deny | deny | deny | deny |
| `ri.models.write` (DEPLOYMENT_ONLY) | deny | deny | deny | allow | deny | deny | deny | deny | deny | deny |
| `ri.consent.write` (ROW_REJECT) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.ingest.write` (ROW_REJECT) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.ingest.read` (DEPLOYMENT_ONLY) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.dsr.write` (STORED_SUBJECT) | deny | deny | deny | deny | deny | deny | deny | deny | deny | allow |
| `ri.audit.export` (ROW_FILTER) | deny | deny | deny | deny | allow | deny | allow | deny | deny | allow |
| `ri.assistant.use` (REQUEST_CONTEXT) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| `ri.portfolio.read` (ROW_FILTER) | allow | allow | deny | deny | deny | deny | deny | deny | deny | deny |

### Table 2 - Roles x dashboard screens

| Screen (required scopes) | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host |
|---|---|---|---|---|---|---|---|---|---|---|
| S00 App shell, not-ready state, banners (`ri.meta.read`) | allow | allow | allow | allow | allow | allow | allow | allow | allow | deny |
| S01 Customer list / picker (`ri.customers.list`) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| S02 Customer view (sections gated by their own scopes) (`ri.insights.read`) | deny | allow | allow | deny | allow | deny | deny | deny | deny | deny |
| S03 Retention outcome capture (`ri.retention.read`, `ri.retention.feedback`) | deny | deny | allow | deny | deny | deny | deny | deny | deny | deny |
| S04 Human review queue (`ri.review.read`) | deny | allow | allow | deny | allow | deny | deny | deny | deny | deny |
| S05 Audit and decision log, replay (`ri.audit.export`) | deny | deny | deny | deny | allow | deny | allow | deny | deny | deny |
| S06 Policy pack viewer (`ri.policy.read`) | deny | allow | deny | allow | allow | allow | allow | allow | allow | deny |
| S07 Policy simulator (`ri.policy.read`, `ri.policy.simulate`) | deny | deny | deny | deny | deny | allow | allow | deny | deny | deny |
| S08 Policy approval workflow (submit, approve, Sharia approve, activate) (any of `ri.policy.author` / `ri.policy.approve` / `ri.sharia.approve` / `ri.policy.release`) | deny | deny | deny | deny | deny | allow | allow | allow | allow | deny |
| S09 Model registry (`ri.models.read`) | deny | allow | deny | allow | allow | deny | deny | deny | deny | deny |
| S10 Assistant panel (`ri.assistant.use`) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| S11 Portfolio view (Could) (`ri.portfolio.read`) | allow | allow | deny | deny | deny | deny | deny | deny | deny | deny |
| S12 Name reveal (never) | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |

### Table 3 - Roles x screen actions

| Action (screen, scope, operation) | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host |
|---|---|---|---|---|---|---|---|---|---|---|
| A01 Generate suggestions for the customer (S02, `ri.recommendations.generate`, `generateRecommendations`) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| A02 Request an explanation (S02, `ri.explanations.read`, `getExplanation`) | deny | allow | allow | deny | allow | deny | deny | deny | deny | deny |
| A03 Record a recommendation customer outcome (S02, `ri.recommendations.feedback`, `reportRecommendationOutcome`) | deny | deny | allow | deny | deny | deny | deny | deny | deny | deny |
| A04 Record a retention outcome (S03, `ri.retention.feedback`, `reportRetentionOutcome`) | deny | deny | allow | deny | deny | deny | deny | deny | deny | deny |
| A05 Transition a review item (start, clear, reject, escalate) (S04, `ri.review.write`, `transitionReviewItem`) | deny | deny | allow | deny | deny | deny | deny | deny | deny | deny |
| A06 Read and replay a decision (S05, `ri.audit.export`, `replayDecision`) | deny | deny | deny | deny | allow | deny | allow | deny | deny | deny |
| A07 Run a simulation (S07, `ri.policy.simulate`, `startPolicySimulation`) | deny | deny | deny | deny | deny | allow | allow | deny | deny | deny |
| A08 Upload, validate, submit a pack (maker) (S08, `ri.policy.author`, `submitPolicyPack`) | deny | deny | deny | deny | deny | allow | deny | deny | deny | deny |
| A09 Record per-policy approval (checker) (S08, `ri.policy.approve`, `recordPackApproval`) | deny | deny | deny | deny | deny | deny | allow | deny | deny | deny |
| A10 Activate a pack (ACTIVE, or ILLUSTRATIVE in the demo) (S08, `ri.policy.release`, `activatePolicyPack`) | deny | deny | deny | deny | deny | deny | deny | allow | deny | deny |
| A11 Transition a registry entry (S09, `ri.models.write`, `transitionModelVersion`) | deny | deny | deny | allow | deny | deny | deny | deny | deny | deny |
| A12 Ask the assistant (S10, `ri.assistant.use`, `postAssistantMessage`) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| A13 View stored enriched transactions (S02, `ri.transactions.read`, `getStoredCustomerTransactions`) | deny | allow | allow | deny | deny | deny | deny | deny | deny | deny |
| A14 Record a Sharia approval (ISLAMIC deployments only) (S08, `ri.sharia.approve`, `recordPackShariaApproval`) | deny | deny | deny | deny | deny | deny | deny | deny | allow | deny |

### Table 4 - Roles x jurisdiction conditions

| Condition | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host |
|---|---|---|---|---|---|---|---|---|---|---|
| JX-1 Token `ri_deployment` differs from the instance `deployment_id` (any scope) | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-2 REQUEST_CONTEXT scope: a context attribute lies outside `ri_jurisdictions` | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-3 STORED_SUBJECT scope: the stored item's jurisdiction lies outside `ri_jurisdictions` | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-4 PACK_SCOPE scope: pack `scope_root` or overlay scope lies outside `ri_jurisdictions` | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-5 ROW_REJECT scope: batch row for a customer outside `ri_jurisdictions` (row rejected) | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-6 ROW_FILTER scope: list/export row outside `ri_jurisdictions` (row omitted) | deny | deny | deny | deny | deny | deny | deny | deny | deny | deny |
| JX-7 REQUEST_CONTEXT scope: context inside the claim but outside the deployment scope (or `UNKNOWN`): call allowed, PDP returns the restrictive default (`UNSUPPORTED_JURISDICTION` / DEC-24) | deny | allow | allow | allow | allow | allow | allow | allow | deny | allow |
| JX-8 Any held scope: claim and deployment match and context inside the claim and the deployment scope | allow | allow | allow | allow | allow | allow | allow | allow | allow | allow |

### Table 5 - Roles x customer-directory access purposes (contract 1.2.0, DEC-68)

| Purpose (use case) | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host |
|---|---|---|---|---|---|---|---|---|---|---|
| `CUSTOMER_SERVICING` (Relationship staff open a customer they are serving (S01 -> S02, UJ-1)) | deny | deny | allow | deny | deny | deny | deny | deny | deny | deny |
| `INSIGHT_REVIEW` (Analyst reviews insight or data-quality outcomes for a narrowed set (S01 -> S02)) | deny | allow | deny | deny | deny | deny | deny | deny | deny | deny |

### Negative-test counts (one test per deny cell)

| Family | viewer | analyst | rel_staff | model_admin | auditor | pol_author | pol_approver | pol_release | sharia_appr | service_host | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ROLE_SCOPE | 29 | 16 | 17 | 26 | 20 | 26 | 25 | 27 | 28 | 11 | 225 |
| ROLE_SCREEN | 11 | 5 | 7 | 10 | 7 | 9 | 8 | 10 | 10 | 13 | 90 |
| ROLE_ACTION | 14 | 10 | 7 | 13 | 12 | 12 | 11 | 13 | 13 | 14 | 119 |
| ROLE_JURISDICTION | 7 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 6 | 62 |
| ROLE_PURPOSE | 2 | 1 | 1 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 18 |
| **All** | 63 | 38 | 38 | 57 | 47 | 55 | 52 | 58 | 60 | 46 | **514** |

## 3. Jurisdiction restriction

The token carries two claims (SEC-03, AD-16):
- `ri_deployment` must equal the instance `deployment_id`.
- `ri_jurisdictions` lists the subtrees the caller may act for.

**Data cells (validation 2026-11-10; planning/21 §2.1).** Each data cell runs its own instance with its own `deployment_id`, so JX-1 binds every token to one cell: a session never reaches personal data in two cells. In Track A the issuer grants `ri_jurisdictions` for personal-data scopes from one cell's roots only. Only the aggregate-only `ri.portfolio.read` result may be combined across the cells of one track, and only from each cell's own suppressed output. No matrix cell changes.

A code is inside a claim when it equals a listed code or descends from one in the deployment's jurisdiction tree. `*` covers the whole tree. The special code `UNKNOWN` is covered only when `UNKNOWN` or `*` is listed.

### 3.1 Check class per scope

Each scope declares one class. Some operations override it, as recorded in the JSON:
- `getDecisionLogRecord` and `replayDecision` use STORED_SUBJECT.
- Pack reads, simulation reads and ingest-batch reads use DEPLOYMENT_ONLY.

| Class | Rule |
|---|---|
| DEPLOYMENT_ONLY | Only `ri_deployment` must equal the instance `deployment_id`. Used for governance or operational metadata with no customer data |
| REQUEST_CONTEXT | Every attribute of the request `jurisdiction_context` (residence, booking entity, servicing location) must lie inside `ri_jurisdictions`. Otherwise 403 `JURISDICTION_NOT_PERMITTED`. Checked before the PDP runs |
| STORED_SUBJECT | The stored item's recorded jurisdiction must lie inside `ri_jurisdictions`. The item is a customer, recommendation, retention action, review item, or decision `jurisdiction_set`. Otherwise 403 `JURISDICTION_NOT_PERMITTED` |
| ROW_FILTER | List and export results contain only rows whose recorded jurisdiction lies inside `ri_jurisdictions`. Governance rows with an empty jurisdiction set are included. No 403 |
| ROW_REJECT | Batch input rows are consent, preference or ingest records. A row whose customer has a supplied or stored residence outside `ri_jurisdictions` is rejected on its own, with issue `JURISDICTION_NOT_PERMITTED`. The other rows proceed |
| PACK_SCOPE | The target pack's `scope_root`, or every overlay control's `jurisdiction_scope`, must lie inside `ri_jurisdictions`. An L0 pack (`*`) needs `*` in the claim. Otherwise 403 `JURISDICTION_NOT_PERMITTED` |

### 3.2 Claim profile per role (what the issuer grants)

| Role | `ri_jurisdictions` granted |
|---|---|
| viewer | Deployment scope roots only (reference data `deployment_scope`) |
| analyst | Demo: the deployment test-jurisdiction set (DEC-16 values as reference data: the in-scope root, one out-of-scope code and `UNKNOWN`). Pilot: subtrees set by access review |
| relationship_staff | As analyst |
| model_admin | Deployment scope roots only |
| auditor | As analyst. The auditor must reach every test context to inspect fail-safe outcomes |
| policy_author | The scope roots of the packs the user maintains. `*` only for the internal L0 maintainer (DEC-45) |
| policy_approver | As policy_author. No user holds the role while DEC-11 is unresolved |
| policy_release | The scope roots of the packs the user activates. `*` only where L0 activation is delegated (DEC-45) |
| sharia_approver | `ISLAMIC` deployments only (DEC-55): `*` for the institution overlay pack plus the deployment's jurisdiction roots. No user holds the role in `CONVENTIONAL` deployments, and none until DEC-63 records a named person |
| service_host | Granted by the host integration config: a subset of the deployment scope, plus `UNKNOWN` if the host sends incomplete context |

**Why the test set is in the staff claim.** UJ-2 must show the restrictive default for an out-of-scope customer and for a customer with no jurisdiction. That happens only if the claim covers those codes (condition JX-7). With an in-scope-only claim, the request stops earlier at 403 `JURISDICTION_NOT_PERMITTED`, which is the JX-2 test. Both outcomes are fail-safe. The claim profile decides which one a user sees. This rule is new in this draft and is listed for ratification in §6.

## 4. Separation of duties (checked by script)

| ID | Rule |
|---|---|
| SOD-1 | Maker != approver. No role holds both `ri.policy.author` and `ri.policy.approve`. The approval call also checks the user (409 `MAKER_CHECKER_VIOLATION`) |
| SOD-2 | `policy_author` cannot approve or activate |
| SOD-3 | `policy_release` cannot author or approve. Activator != maker is checked per user |
| SOD-4 | The auditor is read-only and holds no scope marked `write` in the JSON |
| SOD-5 | No staff role holds a service-only scope: `ri.transactions.enrich`, `ri.consent.write`, `ri.ingest.write`, `ri.ingest.read`, `ri.dsr.write`, `ri.policy.overlay.write` |
| SOD-6 | Only `policy_approver` holds `ri.policy.approve`. No demo user holds that role while DEC-11 is open |
| SOD-7 | Reviewer != requester on review transitions (409 `MAKER_CHECKER_VIOLATION`). `ReviewerRole` in outcome schemas is limited to `analyst` and `relationship_staff` |
| SOD-8 | Nobody sees customer names. The name reveal screen (S12) does not exist in this subsystem |
| SOD-10 | Customer directory (DEC-68). Only `analyst` and `relationship_staff` hold `ri.customers.list`; `auditor` and `service_host` never do. Every holder has at least one registered access purpose (Table 5) and every purpose is registered to a holder; no machine client is registered. `listCustomers` is deprecated and not implemented |
| SOD-9 | Sharia maker-checker (DEC-55). Only `sharia_approver` holds `ri.sharia.approve`, and it holds no `ri.policy.author`, `ri.policy.approve` or `ri.policy.release`. The call also checks Sharia approver != maker per user (409 `MAKER_CHECKER_VIOLATION`). A pack in an `ISLAMIC` deployment needs both the legal and the Sharia approval. Per user, the Sharia approver also differs from the legal approver of the same pack version, and the activator differs from both (validation 2026-11-10; planning/21 §4.3) |

## 5. Negative tests

There is one test per `deny` cell. The full list, with ID, probe and expected result, is `negative_tests` in the JSON. Test IDs follow these patterns:
- `NT-RS-<role>-<scope>`
- `NT-SC-<role>-<screen>`
- `NT-AC-<role>-<action>`
- `NT-JX-<role>-<condition>`
- `NT-PU-<role>-<purpose>` (contract 1.2.0)

In addition, `deprecated_operation_tests` holds one test per role for the deprecated, not implemented `listCustomers` (`NT-DEP-<role>-listCustomers`, expected 404 `NOT_FOUND`, no row): 10 tests, outside the one-per-deny-cell count.

Expected results by family:

| Family | Probe | Expected result |
|---|---|---|
| ROLE_SCOPE | The first operation that uses the scope, called with a token for the role | The issuer refuses to put the scope in the role's token. A forged token that carries it gets 403 `FORBIDDEN_SCOPE`, and an audit record is written |
| ROLE_SCREEN | Navigate to the screen | The screen is absent from navigation, and a direct route shows "access denied". No API call returns data. A service token cannot sign in |
| ROLE_ACTION | The action's operation | The control is not rendered, and the call returns 403 `FORBIDDEN_SCOPE` |
| ROLE_PURPOSE | `searchCustomerDirectory` with that `access_purpose` and a narrowing filter | The role holds `ri.customers.list`: 403 `ACCESS_PURPOSE_NOT_REGISTERED`. The role does not: 403 `FORBIDDEN_SCOPE` (scope checked first). Either way an audit record with outcome `DENIED` and the denial code is written |
| ROLE_JURISDICTION | The condition's request | The role holds a scope of the class: 403 `JURISDICTION_NOT_PERMITTED` (JX-1..JX-4), the row is rejected (JX-5) or the row is omitted (JX-6). The role holds no such scope: 403 `FORBIDDEN_SCOPE`, which proves the scope check runs first |

The counts are in the table at the end of §2. Story 2.7 runs these tests in CI; Story 9.1 runs the screen and action families in the UI.

## 6. Changes and gaps against planning/15 §5.1 and EXPERIENCE (for GATE-3)

| ID | Finding | Resolution in this draft | Needs |
|---|---|---|---|
| C-RS-1 | In planning/15 §5.1 the auditor held `ri.recommendations.generate`. That call creates PROPOSED recommendations and review items and emits `recommendation.proposed`, which contradicts "auditors are read-only" (SEC-19, AD-21) | The cell is now `deny`. The auditor sees recommendations through the decision log, replay and audit export (`ri.audit.export`). In composite evaluate, the auditor's recommendation section returns DENIED `FORBIDDEN_SCOPE`. This file supersedes planning/15 §5.1, which is read-only (AGENTS.md). EXPERIENCE is corrected | SEC/MGR confirm at GATE-3 (DEC-38) |
| C-RS-2 | The jurisdiction claim rule was not written down: when a context outside the claim gets 403, and when it reaches the PDP's restrictive default | §3 classes, conditions JX-1..JX-8 and the claim profiles | SEC/MGR ratify at GATE-3 (DEC-38) |
| G-RS-1 | Portfolio view (S11, Story 9.7, Could): EXPERIENCE gives it to viewer and analyst, but contract v1.0 had no aggregate operation or scope | **DECIDED (DEC-62, developer-delegated 2026-11-10).** Contract 1.1.0 adds `GET /v1/portfolio/summary` (`getPortfolioSummary`) under new scope `ri.portfolio.read` (viewer, analyst; ROW_FILTER): aggregate counts by one dimension, no tokens or personal data, cells below `min_cell_size` (config, default 20 [ASSUME]) returned as suppressed, minors excluded from commercial dimensions, aggregation across cells of one track only. Story 9.7 stays Could | Manager countersignature recommended |
| G-RS-2 | Customer picker (S01): contract v1.0 had no customer-list operation | **DECIDED (DEC-62); S01 part superseded by V-RS-1 (DEC-68, contract 1.2.0).** Contract 1.1.0 adds `GET /v1/customers` (`listCustomers`) under `ri.insights.read` (ROW_FILTER): customer token, residence jurisdiction code, protection status, data-quality status and update time only; no name, contact data, address, ZIP or account number; minors listed only with their protection status | Manager countersignature recommended |
| G-RS-3 | The customer view shows transactions (UJ-1 step 2, Story 9.2), but no staff role could read stored transactions | **DECIDED (DEC-62).** Contract 1.1.0 adds `GET /v1/customers/{customer_token}/transactions` (`getStoredCustomerTransactions`) under a new **purpose-bound** scope `ri.transactions.read` (analyst, relationship_staff; REQUEST_CONTEXT; purpose `INSIGHT` only): enriched view (date, amount, currency, kind, category code, flags), no raw description, counterparty or merchant name; sensitive or prohibited categories only as neutral codes; minors 403 `MINOR_PROTECTED`. Chosen over reusing `ri.insights.read` so the transaction view can be granted or withdrawn separately (action A13) | Manager countersignature recommended |
| T-RS-1 | Two-track scope (DEC-50, DEC-55): Sharia approval for packs, catalogue items and templates in `ISLAMIC` deployments | New role `sharia_approver`, scope `ri.sharia.approve` (PACK_SCOPE), operation `recordPackShariaApproval`, screen S08 action A14, SOD-9. EXPERIENCE.md now lists `sharia_approver` (ISLAMIC only) on the Policy pack viewer and Policy approval rows, and Story 9.9 carries the A14 acceptance criteria (readiness re-run fix, 2026-11-10) | COMP / Sharia function review at GATE-3A |
| V-RS-1 | V-8 (independent validation 2026-11-10): `listCustomers` used `ri.insights.read`, held by `auditor` and `service_host`, with no purpose binding, so bulk token enumeration was possible | **DECIDED (DEC-68).** Contract 1.2.0 adds `searchCustomerDirectory` under staff-only `ri.customers.list` (ROW_FILTER) with a required access purpose (Table 5), exact lookup or narrowing filter, page <= 25, query total <= 100, per-user quotas and an audit record per call; S01 requires `ri.customers.list`, so the auditor loses S01 (keeps S02 by known token, S04 and S05) and `service_host` has no directory path. `listCustomers` is deprecated and not implemented. Supersedes the S01 part of G-RS-2 | Manager countersignature recommended |
| N-RS-1 | EXPERIENCE called the review-queue actor `retention_reviewer`, which is not a role | Read as `relationship_staff` (the `ReviewerRole` value). EXPERIENCE is corrected | none |

The verification script (`verify_matrix.py`, kept in the session scratch folder; nothing is installed in the project) checks:
- The JSON validates against a JSON Schema 2020-12 for the matrix (Python `jsonschema` 4.26).
- The scope set equals both OAuth2 flows in the OpenAPI file exactly.
- Every authenticated operation is listed under exactly its OpenAPI scopes, and `GET /health` is the only unauthenticated operation.
- Every cell is present, with exactly one negative test per `deny` cell and none for an `allow` cell.
- Screen and action cells are consistent with the scope cells.
- SOD-1..SOD-6, SOD-8, SOD-9 and SOD-10 hold, and `ri.transactions.read` / `ri.portfolio.read` are staff-only (DEC-62).
- Contract 1.2.0 (DEC-68): the purpose list equals the OpenAPI `DirectoryAccessPurpose` enum; `role_purpose` cells follow scope holding and registration; S01 requires `ri.customers.list`; `listCustomers` is the only deprecated operation, is marked not implemented in the OpenAPI file and the JSON, has a test for every role and backs no screen action.
- Every Markdown table cell equals the JSON.

The result is recorded in the change log below.

## Change log

- 2026-11-09: created (PH-3, readiness P-4; planning/10 S12.1 matrix part, SEC-19, DEC-38 draft). C-RS-1 and C-RS-2 applied; G-RS-1..G-RS-3 are open for GATE-3. Check run: `verify_matrix.py` **PASS**. It covered 9 roles and 27 scopes (identical to both OAuth2 flows in the OpenAPI file), 42 operations, 13 screens, 12 actions and 8 jurisdiction conditions: 540 cells in all, of which 393 are `deny`, with 393 negative tests. All 540 Markdown cells equal the JSON.
- 2026-11-10 (two-track scope, planning/21; planning/12 §H.7): version 0.2.0. DEC-62 closes G-RS-1..G-RS-3 (new scopes `ri.transactions.read`, `ri.portfolio.read`; S01 uses `listCustomers`, S11 enabled for viewer and analyst; action A13). DEC-55 adds role `sharia_approver`, scope `ri.sharia.approve`, action A14 and SOD-9 (T-RS-1). DEC-38 now DECIDED (developer-delegated). Check run: `verify_matrix.py` **PASS**. It covered 10 roles and 30 scopes (identical to both OAuth2 flows of contract 1.1.0), 46 operations, 13 screens, 14 actions and 8 jurisdiction conditions: 650 cells, of which 487 are `deny`, with 487 negative tests. All 650 Markdown cells equal the JSON.
- 2026-11-10 (readiness re-run fixes): T-RS-1 note updated (UX pair and Story 9.9 now include `sharia_approver`). Matrix cells unchanged.
- 2026-11-10 (independent two-track validation): SOD-9 text adds the per-user checks (Sharia approver != legal approver of the same pack version; activator != both), mirrored in the JSON rule text; §3 data-cell note (one instance and `deployment_id` per cell, so JX-1 binds a token to one cell); JSON `contract.info_version` 1.1.0 -> 1.1.2. No matrix cell changed; `verify_matrix.py` re-run: PASS.
- 2026-11-10 (contract 1.2.0, V-8, planning/12 DEC-68): version 0.3.0. New scope `ri.customers.list` (analyst, relationship_staff; ROW_FILTER) for `searchCustomerDirectory`; S01 now requires it (auditor S01 -> deny); new Table 5 (roles x access purposes `CUSTOMER_SERVICING`, `INSIGHT_REVIEW`) and family ROLE_PURPOSE; SOD-10; rule 6; V-RS-1; `listCustomers` deprecated and not implemented, with 10 `deprecated_operation_tests`. Generator and verifier extended (scratch `v89/gen_matrix.py`, `v89/verify_matrix.py`). Check run: `verify_matrix.py` **PASS**: 10 roles, 31 scopes (identical to both OAuth2 flows of contract 1.2.0), 47 operations, 13 screens, 14 actions, 8 jurisdiction conditions, 2 access purposes: 680 cells, of which 514 are `deny`, with 514 negative tests (+10 deprecated-operation tests). All 680 Markdown cells equal the JSON.
