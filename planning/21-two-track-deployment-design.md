# 21 - Two-Track Deployment Design (Conventional NYC + London; Islamic bank)

**Label key:** [REC] recommendation | [ASSUME] assumption | [UNKNOWN] | [MANAGER] manager countersignature recommended | [LEGAL-CONFIRM] legal/compliance must confirm | [SHARIA-CONFIRM] the bank's Sharia Compliance function / Internal Sharia Supervisory Committee (ISSC) must confirm.
**Status:** planning only, 2026-11-10. No application code. Contract and schema sketches are illustrative, not code.
**Authority:** developer scope decision of 2026-11-10 (verbatim): *"target NYC, London and Islamic bank, separately: NYC and London in one, and on the other side the Islamic bank."* All other choices here were made by the developer's delegate on the developer's delegated authority (*"rest of the questions you answer on your own; I need the best results"*) and are recorded in file 12 §H.7 as **DECIDED (developer-delegated, 2026-11-10)**, DEC-50 to DEC-67. A manager countersignature is recommended where marked [MANAGER].

> **This document does not state that any law applies.** UK and UAE legal content belongs to `19-uk-london-regulatory-applicability-matrix.md` (rows U01-U25, questions UQ1..) and `20-islamic-bank-uae-regulatory-and-sharia-matrix.md` (rows E01-E21, SH01-SH14, Sharia design constraints SDC-01..SDC-18, questions IQ1-IQ21). Both were written in parallel on 2026-11-10; this file is aligned with them (§4.4, §7, §12) and cites rows by ID only. NYC legal content stays in file 13. Sharia content (which products are compliant, which terms are prohibited, which merchant categories are excluded) is decided by the bank's Sharia function, not by us; every list below is a **SYNTHETIC / ILLUSTRATIVE placeholder** marked [SHARIA-CONFIRM]. No fact about any bank's charter, licence, regulator status or named staff is assumed.

## 0. What this supersedes

| Before (files 02-18, BMAD artefacts up to 2026-11-10) | Now | Decision |
|---|---|---|
| NYC-first; federal + NY State + NYC only | **Two tracks.** Track A "Conventional" = US / US-NY / US-NY-NYC **and** UK / England / London. Track B "Islamic" = an Islamic bank, assumed UAE onshore [ASSUME] | DEC-50 (resolved) |
| DEC-13: UK, other countries and other US states deferred | **UK (London) returns to scope** in Track A; other US states, other UK nations and other countries stay deferred; UAE enters as Track B | DEC-13 SUPERSEDED by DEC-50, DEC-51 |
| DEC-28: international data residency deferred | Addressed now: per-jurisdiction data cells in Track A (DEC-52) and a fully separate Track B (DEC-53) | DEC-28 SUPERSEDED by DEC-52, DEC-53 |
| DEC-50 BLOCKED-EXTERNAL: target market NYC vs UAE vs both | **DECIDED** by the developer: both, as two separate tracks | DEC-50 |
| One deployment scope (`deployment_scope`) per install | A **deployment profile** (§3) per track, and a **data cell** per legally separate data partition inside Track A | DEC-51, DEC-52 |
| Conventional products only | Institution type `CONVENTIONAL` or `ISLAMIC`; the Islamic type adds a tighten-only **Sharia overlay** (§4) | DEC-54, DEC-55 |

What does **not** change: one shared intelligence core and one contract (no duplicated logic, no track literal in `core/`); the minors two-tier block list (file 14 §13.3; Tier 1 never relaxed); no credit decision of any kind, credit only as `CREDIT_REFERRAL`, `not_for_credit_decisioning` on every output (DEC-42); synthetic data and synthetic catalogues only (12 §H.4 condition 1); every jurisdiction pack `PENDING_LEGAL_REVIEW` enforced as BLOCK until a named approver signs (DEC-11, DEC-63); template-only explanations in MVP, no hosted LLM (DEC-06).

## 1. Design goals

| # | Goal | Label |
|---|---|---|
| TT-1 | **Same core, same contract, both tracks.** A track is configuration (deployment profile, packs, catalogue, reference data, models), never a code branch. CI lint forbids `track`, `ISLAMIC`, `CONVENTIONAL`, `GB`, `AE`, `US` literals in `core/` outside the policy and profile loader (extends file 14 PG-1) | DEC-51 |
| TT-2 | **Track B never shares data, models, keys, identities or logs with Track A.** Only released code images, the contract and the market-neutral control catalogue are shared | DEC-53 |
| TT-3 | **Inside Track A, personal data does not cross the US/UK boundary.** US and UK personal data live in separate databases in their own regions; only aggregate, non-personal metrics may be combined | DEC-52 [LEGAL-CONFIRM] |
| TT-4 | **Every overlay tightens, none loosens.** Institution-type overlay, jurisdiction packs and host overlays combine by "most restrictive wins" (file 14 §4.2; DEC-26) | DEC-54 |
| TT-5 | **Fail safe on unknowns.** A customer whose jurisdiction is not in the deployment's (cell's) scope is `UNSUPPORTED_JURISDICTION`; missing or conflicting attributes are `UNKNOWN`; both deny every commercial activity (DEC-24, DEC-25) | DEC-51 |
| TT-6 | **Two demos from one build:** a Track A demo and a Track B demo, each standalone, synthetic, watermarked | DEC-64 |

## 2. Tracks, cells and jurisdictions

| Track | Institution type | Jurisdiction tree (codes; file 14 §2) | Data cells | Hosting | Status of legal content |
|---|---|---|---|---|---|
| **A - Conventional** | `CONVENTIONAL` | `*` > `US` > `US-NY` > `US-NY-NYC`; `*` > `GB` > `GB-ENG` > `GB-ENG-LDN` (London as a deployment-defined local level, same convention as `US-NY-NYC` [ASSUME no ISO code is used for Greater London as a whole]) | `A-US` (US personal data, US region) and `A-UK` (UK personal data, UK region) | Bank-controlled environment per cell [REC]; region set by `CTL-DATA-RESIDENCY` | US: file 13 (PENDING_LEGAL_REVIEW). UK: file 19 **in progress** |
| **B - Islamic** | `ISLAMIC` | `*` > `AE` > `AE-<emirate>` [ASSUME onshore UAE; the emirate is unknown, e.g. `AE-DU` or `AE-AZ` as ISO 3166-2 examples only; financial free zones such as DIFC/ADGM are out of scope] | `B-AE` (one cell) | **Bank-hosted / on-premises only** (UAE manual ch. 33.2 rule that customer data stays on the bank's own systems; file 00 §5C) | File 20 **in progress**; Sharia content [SHARIA-CONFIRM] |

Notes:
- **Cell = unit of personal-data isolation.** One database (PostgreSQL per cell, file 03), one subject-key store (crypto-shredding, DEC-35), one audit/decision log, one object store, one secrets/KMS scope, one region. A deployment profile may own one or more cells; Track A owns two, Track B one.
- **Jurisdiction-to-cell routing** is data: `cell.jurisdiction_roots = [US]` for `A-US`, `[GB]` for `A-UK`, `[AE]` for `B-AE`. A request is served only by the cell whose roots contain the resolved residence and booking jurisdictions. A customer resident in one cell's tree and booked in another's (e.g. NYC resident booked in London) resolves to `UNKNOWN` (DEC-25 safe assumption) and is denied every commercial activity; no cross-cell join exists to "help". [REC] [LEGAL-CONFIRM]
- **Out of scope in every track:** other US states (test fixture `US-CA` stays as the out-of-scope case), Scotland/Wales/Northern Ireland, EU, Qatar, other GCC states. They can be added later as packs and cells with no core change (file 14 §12).

### 2.1 Cell isolation checklist (validation 2026-11-10)

Every path that could carry personal data, or give a person access to it, stays inside one cell. This tightens DEC-52/DEC-53 (file 12 §H.7.3); it is design, not code. [REC] [LEGAL-CONFIRM for anything that would ever relax it, TT-R2]

| Path | Rule |
|---|---|
| Runtime instance and sessions | Each cell runs its own instance of the shared release with its own `deployment_id` (ERD `deployment.cell_id`). Tokens carry `ri_deployment` = that id (role matrix JX-1), so a session reaches one cell only. The Track A IdP realm is shared, but `ri_jurisdictions` claims for personal-data scopes are issued from one cell's roots only. The one dashboard build connects to one cell instance per session; a person who works in both cells has a separate session per cell. Remote access from one region to the other cell's personal data is itself a transfer question and is not designed in [LEGAL-CONFIRM] (TT-R2) |
| Application logs, traces, metrics (telemetry) | PII-free and also token-free: no customer, account or consent token, no `subject_ref`, no free text; correlation IDs only. Stored in the cell's region. A Track-level operations view receives only metrics with no customer-level label |
| Audit and decision logs, replay snapshots | In the cell's database, hash-chained; replay payloads encrypted with the cell's subject keys (DEC-35). Audit export is scoped to one cell (as SDC-15 does for `B-AE`); auditor sessions are per cell |
| Outbox and events | One outbox and one transport adapter per cell; events carry `ricell`; a host consumer subscribes per cell, in-region (host obligation, DEC-33). No topic or fan-in mixes customer-level events from two cells |
| Backups and restores | Per cell, in-region, encrypted with the cell's keys; restored only into the same cell (file 08 TT-01); DSR erasures re-applied after restore (ERD E-DEL) |
| Analytics and reporting | The only cross-cell output is the portfolio summary under §10A. No export, BI extract or warehouse feed of customer-level data leaves a cell |
| Model training inputs | Features, labels, training sets, shadow logs and evaluation data per cell, synthetic data included (§6). Registry metrics are aggregates only |
| Support and admin access | No Track-wide administrator with personal-data access. Operator/DBA access per cell, in-region, named, time-limited (break-glass) and audited (`audit_record`). Support tools receive correlation IDs only, never tokens or payloads. The vendor/developer has no access to a non-demo cell's personal data |
| Keys and secrets | Per cell: KMS scope, subject-key store (DEC-35) and tokenisation key (§2.2). Track B keys are bank-held in the UAE (DEC-53) |
| Synthetic generator | One seed and one output per cell, never mixed (§5) |

### 2.2 Customer token scheme (validation 2026-11-10)

- Customer, account and consent tokens are opaque at the boundary (file 15 IC5). The subsystem never derives a token from a name, national identifier, account number or date of birth with an unkeyed hash, and it holds no mapping back to a person: tokens are **not reversible by the subsystem or by any consumer of its outputs**.
- **Plugin:** the host tokenisation service in the same cell and region issues tokens (file 15 CI-4, host obligation). Required properties: a random surrogate, or a keyed hash (e.g. HMAC-SHA-256) whose key is specific to the cell and track, so **the same person gets unrelated tokens in `A-US`, `A-UK` and `B-AE`**. Only the host holds the mapping, in-region.
- **Standalone / demo:** the synthetic generator issues random tokens per cell seed; no token value repeats across cells.
- **No stable cross-context linkage:** decision, audit and event records use only the serving cell's tokens; the portfolio summary carries none; Track A and Track B never share a token namespace. A token presented to another cell is "not found", with no lookup or hint (Story 2.12). Staff identifiers (`reviewer_ref`, `actor`) are per IdP realm.
- Re-tokenisation is a host action; crypto-shredding (DEC-35) is unaffected.

### 2.3 Purpose-bound customer directory (contract 1.2.0, V-8, DEC-68)

The only path that lists customer tokens for people is the staff picker. It follows least privilege and purpose limitation; it does not change §2.1 or §2.2. Design, not code. [REC]

| Control | Rule |
|---|---|
| Scope | `ri.customers.list`, separate from generic `ri.insights.read`; held by `analyst` and `relationship_staff` only. `auditor`, `service_host` and every other role or machine client are denied by default. The host never needs it: it holds the token mapping itself (§2.2) |
| Approved purpose | Required `access_purpose` from a closed list: `CUSTOMER_SERVICING` (relationship staff), `INSIGHT_REVIEW` (analyst). Registered per role (role matrix Table 5) and per client (issuer registry, none registered). Unknown or missing: 400; not registered: 403. A new purpose is a contract decision |
| Minimum necessary | Exact token lookup = operational access (one row or an empty page, same answer for unknown, other-cell or out-of-claim tokens). Anything else = enumeration: needs `segment_code` or `data_quality_status`; page <= 25; at most 100 rows per query [ASSUME, may only be lowered]; cursor bound to user, purpose and filters; per-user daily quotas 200 calls and 500 enumerated rows [ASSUME] -> 429. Repeated filter variations raise a security alert for review |
| Data | Token, residence code, protection status, data-quality status, update time; no personal data (IC5). Minors only with their protection status (file 14 §13.3). Rows only from the serving cell and inside the caller's claim (ROW_FILTER) |
| Audit | Every call after token validation, allowed or denied, writes an `audit_record` with an access block: account, client, role, scope, purpose, access kind, filters, result count, outcome and denial code, with timestamp, cell and track. Records stay in the cell (§2.1) |
| Auditor | Read-only: reads the access records through the audit export and opens a customer only by a token taken from a record; holds no directory scope |
| Old picker call | `listCustomers` is deprecated and not implemented by any release (contract 1.2.0) |

## 3. Deployment profile (configuration, not code)

Each deployment loads exactly one signed **deployment profile** at bootstrap. The profile is read-only at runtime, versioned, and its hash is written to every decision record. A request cannot override any profile field (the host cannot "switch off" the Sharia overlay by sending a different institution type). [REC]

```yaml
# ILLUSTRATIVE - planning sketch, not code
deployment_profile:
  profile_id: PROFILE-TRACK-A            # PROFILE-TRACK-B for the Islamic track
  profile_version: 1.0.0
  track: A                               # A | B (reference data; core reads it only through the profile loader)
  institution_type: CONVENTIONAL         # CONVENTIONAL | ISLAMIC
  mode: STANDALONE                       # STANDALONE | PLUGIN (file 04)
  demo: true                             # demo deployments only (DEC-44 load guard)
  synthetic_only: true                   # AD-19 guard; must be true until 12 §H.4 condition 1 is lifted
  cells:
    - cell_id: A-US
      jurisdiction_roots: [US]
      in_scope: [US, US-NY, US-NY-NYC]
      region: US                          # CTL-DATA-RESIDENCY allowed_deployments
      currency: USD
      locales: [en-US]
      catalogue_id: CAT-A-US-SYNTH
    - cell_id: A-UK
      jurisdiction_roots: [GB]
      in_scope: [GB, GB-ENG, GB-ENG-LDN]
      region: UK
      currency: GBP
      locales: [en-GB]
      catalogue_id: CAT-A-UK-SYNTH
  packs:                                 # L0 + jurisdiction packs + institution overlay; all PENDING_LEGAL_REVIEW
    baseline: PACK-GLOBAL-BASELINE
    jurisdiction: [PACK-US, PACK-US-NY, PACK-US-NY-NYC, PACK-GB, PACK-GB-ENG, PACK-GB-ENG-LDN]
    institution_overlay: null             # Track B: PACK-INST-ISLAMIC
  age_of_majority_by_jurisdiction: {US: 18, GB: 18}   # config; [LEGAL-CONFIRM]; Track B: {AE: 21} [ASSUME, restrictive]
  llm: {enabled: false, hosting: NONE}    # DEC-06, DEC-58
  model_registry_scope: TRACK-A           # DEC-57
  idp_realm: track-a                      # separate realm per track (DEC-53)
```

Track B profile differences: `track: B`, `institution_type: ISLAMIC`, one cell `B-AE` (`jurisdiction_roots: [AE]`, `region: BANK-ON-PREM-AE` (UAE, file 20 IQ3), `currency: AED`, `locales: [ar-AE, en-AE]`, `catalogue_id: CAT-B-AE-ISLAMIC-SYNTH`), `institution_overlay: PACK-INST-ISLAMIC`, `llm.hosting` allowed values `NONE | BANK_ON_PREM` only, `model_registry_scope: TRACK-B`, `idp_realm: track-b`.

Bootstrap refusals (deployment stays not ready, fail-closed): profile signature invalid; `institution_type: ISLAMIC` without an institution overlay pack; any pack, catalogue, model or template whose `track` tag differs from the profile's; a cell region not in `CTL-DATA-RESIDENCY`; `llm.hosting` not allowed for the track; `demo: true` with `mode: PLUGIN` (DEC-44, unchanged). [REC]

### 3.1 Profile integrity: signer, signature, keys, fail-closed (validation 2026-11-10)

Design level only; it tightens DEC-51 and DEC-61 (file 12 §H.7.3). [REC]
- **Signature.** A detached signature over the canonical form of the profile (RFC 8785 JSON canonicalisation of the parsed YAML) plus a SHA-256 `content_sha256`. Algorithm Ed25519 [REC]; ECDSA P-256 with SHA-256 if the bank's cryptography policy requires FIPS-validated modules [ASSUME]. The envelope names `alg` and `key_id`; any other algorithm is refused (no negotiation).
- **Signer and keys.** One profile-signing key pair per track, plus the demo-class key for `demo: true` deployments only (DEC-44). Private keys are non-exportable in the track's KMS/HSM (Track B: bank-held, on-premises, in the UAE; DEC-53). Signing is maker-checker: the author of a profile change is never its signer. For `demo: false` the signer is the bank's release function (the organisation holding `policy_release`, DEC-12), never the vendor or developer; every signing writes an audit record.
- **Trust anchors.** The release image carries the allowed public-key fingerprints, each bound to one track; a key may sign only profiles of its bound track. The environment supplies only the profile file path and the KMS reference, never a trust anchor. New or rotated keys ship in a new release; a revoked `key_id` is on the image's deny list.
- **Fail-closed (not ready, specific reason, no partial start):** profile missing or unreadable; signature missing or invalid; `alg` not allowed; `key_id` unknown, revoked or bound to another track; content hash mismatch (altered profile); profile schema version outside the range the image supports (incompatible); `profile_version` lower than the version recorded for the cell (rollback) without a signed rollback record; a cell instance whose `cell_id` is not in the profile; plus every refusal listed above.
- **No runtime switch.** Track, `institution_type`, cells, packs, institution overlay, `llm.hosting`, `demo`, `synthetic_only`, registry scope and IdP realm come only from the verified profile through `DeploymentProfilePort`. No environment variable, command-line flag, feature flag, config file, admin endpoint, header or request field can set or override them. Other config and feature flags are read after the profile and may only tighten (switch a feature off). There is no hot reload: a changed profile needs a new signed version and a restart. CI lint forbids reading these fields from the environment.

## 4. Institution-type policy overlay (Sharia layer)

### 4.1 Placement and composition

Resolution order (extends file 14 §4.1; each step can only add or tighten):

1. **L0 `PACK-GLOBAL-BASELINE`** (locked controls: no credit decision, credit only as referral, minors two-tier list, LLM guardrail, default deny).
2. **Jurisdiction packs** L1-L3 for the resolved jurisdiction set (file 14), all `PENDING_LEGAL_REVIEW` = BLOCK until approved.
3. **Institution-type overlay `LI`**: `PACK-INST-ISLAMIC` when `institution_type = ISLAMIC`; nothing when `CONVENTIONAL`. It is **restrictive-only by CI invariant** (like L0, DEC-45): no policy in it may grant an activity, widen an allow-list or raise an upper bound.
4. **Host overlay** (plugin only; tighten only, DEC-17).

Combining is per control, "most restrictive wins" (file 14 §4.2; DEC-26). The monotonicity property test (file 14 §14) is extended: adding the institution overlay to any bundle never makes any decision more permissive. The overlay cannot touch locked L0 controls and cannot relax Tier 1 or Tier 2 of the minors list. [REC]

### 4.2 New controls (added once to the market-neutral control catalogue; file 14 §5)

| control_id | Purpose | Params (illustrative) | PEP | Combining |
|---|---|---|---|---|
| `CTL-CATALOGUE-ELIGIBILITY` | A catalogue item may be suggested only if it carries every attribute the bundle requires. Islamic overlay requires `sharia_status = SHARIA_APPROVED` (or, in an `ILLUSTRATIVE` demo bundle only, the separate attribute `illustrative_sharia_assumption: ASSUME_COMPLIANT`; `sharia_status` never takes an illustrative value) | `required_item_attributes{}` | M6 candidate generation (before ranking), M9 (assistant may mention only eligible items) | intersection of required attributes (more requirements = stricter) |
| `CTL-TERMINOLOGY` | Explanations, assistant answers, reason-code texts and marketing templates use only approved terms. Islamic overlay maps prohibited terms to approved ones (illustrative: "loan" -> "finance", "interest" -> "profit" or "profit rate", "interest-free" not used as a claim, "insurance" -> "Takaful" for Takaful items) [SHARIA-CONFIRM] | `term_list_id`, `version`, `on_violation: BLOCK_AND_FALLBACK` | M9 output validator, template CI lint, reason-code library build | union of term lists |
| `CTL-PROHIBITED-CATEGORY` | Merchant categories and product types the institution does not promote. Effect: never suggested, never in offers, rewards or partner content, never a ranking or targeting feature, never used to infer religion or religiosity, never shown with judgemental wording. M2 still categorises spending neutrally for budgeting insight | `category_list_id` (illustrative: gambling, alcohol, conventional interest-bearing products, adult entertainment [SHARIA-CONFIRM]), `uses_blocked[]` | M2 (label only), M3, M6, M7, M9, feature build | union of lists; union of blocked uses |
| `CTL-SHARIA-APPROVAL` | Packs, catalogue items, explanation/marketing templates and reason-code texts used in a customer-facing output need `sharia_approval_status = SHARIA_APPROVED` with an approval reference; otherwise the output is BLOCK with reason `SHARIA_REVIEW_PENDING` | `artefact_kinds[]`, `allow_illustrative` (demo bundles only) | M6, M7, M9, pack load, template load | deny-overrides |

| `CTL-CONTACT-LIMIT` (existing control, new Track B parameters) | Telemarketing contact caps and consent dimensions for the AE pack (file 20 row E08, CBUAE Telemarketing Regulation C 3/2026 [LEGAL-CONFIRM]): purpose x channel x product type x contact method x language consent; frequency caps; DNCR check as a host obligation. **Pending file 20 IQ4**, in-app banners, push and digital messages built from our outputs are treated as telemarketing (restrictive reading) | `max_per_window`, `window`, `quiet_hours`, `channels_counted[]` (default: all customer-facing channels incl. in-app) | M7, M6 output | min / union (file 14 §4.2) |

`EXCL-FORBIDDEN-INFERENCE` (file 14 §13.2) gains an explicit entry in every track: **religion or religious observance must not be inferred** from merchant categories, donations, Islamic product holdings or the institution type. A Track B customer is not assumed to be Muslim; the Sharia overlay governs what the **bank** offers, not who the customer is. [REC]

### 4.3 Sharia approval state (packs, catalogue items, templates)

A second approval dimension, separate from and in addition to legal approval (file 14 §9; DEC-45):

`NOT_REVIEWED` -> `IN_SHARIA_REVIEW` -> `SHARIA_APPROVED` | `SHARIA_REJECTED` -> (`RETIRED`)

- Held on: institution overlay pack and every policy in it, every Track B catalogue item, every Track B explanation/marketing template and reason-code text (en and ar).
- Recorded by role `sharia_approver` (new; maps to the bank's Sharia Compliance function / ISSC secretariat). Separation of duties per file 20 SDC-17, applied at its strictest form (role matrix SOD-9): `sharia_approver` holds no `policy_author`, `policy_approver` or `policy_release` scope; per user, the Sharia approver differs from the maker and from the legal approver of the same pack version, and the activator differs from both (validation 2026-11-10); the vendor/developer never holds it; the Sharia Compliance Function's review is never automated by the subsystem (role matrix SOD-9). Interim approver role per DEC-63; no named person is assumed.
- **Submit transition:** in an ISLAMIC deployment, `:submit` (pack `VALIDATED` -> `IN_LEGAL_REVIEW`) also moves the overlay policies and the released Track B artefacts in the submission from `NOT_REVIEWED` to `IN_SHARIA_REVIEW` in the same audited step (contract 1.1.1); the `sharia_approver` then records the outcome.
- A Track B pack becomes `APPROVED` only when **both** the legal approver and the Sharia approver have signed; activation is unchanged (`policy_release`, DEC-12). Until then: BLOCK.
- **Demo:** inside an `ILLUSTRATIVE` bundle (DEC-44 load guard unchanged) synthetic items and templates may carry `illustrative_sharia_assumption: ASSUME_COMPLIANT`, confirmed by the activator; outputs carry the DEC-44 watermark plus **"illustrative - not a Sharia ruling"**. [REC]

### 4.4 Alignment with file 20 Sharia design constraints (SDC-01..SDC-18)

| SDC | Where it lands in this design |
|---|---|
| SDC-01 approved products only (bank ISSC ref; other boards' approvals rejected) | `CTL-CATALOGUE-ELIGIBILITY` + catalogue load check (§4.2) |
| SDC-02 no conventional products in Track B | Catalogue field `contract_type` from an ISSC-approved enum; bootstrap refuses a Track B catalogue with `CONVENTIONAL_*` items (§3 refusals) |
| SDC-03 bilingual terminology, contract names whitelisted in context | `CTL-TERMINOLOGY` with ar + en lists and a context whitelist |
| SDC-04 Sharia approval of every customer-facing artefact; ar/en approved as a pair | §4.3; `CTL-SHARIA-APPROVAL` |
| SDC-05 no free-text customer content | Typed-slot templates only; `CTL-LLM-GUARDRAIL` `allow_llm: false` |
| SDC-06, SDC-07 prohibited categories; no religion inference | `CTL-PROHIBITED-CATEGORY`; `EXCL-FORBIDDEN-INFERENCE` |
| SDC-08 financing stays a referral; phone channel excluded for financing referrals | `CTL-CREDIT-REFERRAL-ONLY`; `CTL-CHANNEL-ALLOW` in the overlay |
| SDC-09, SDC-16 late-payment charity and settlement/fee arithmetic data-only | §4.5; M6/M9 slot whitelist |
| SDC-10 Sharia questions go to humans | M9 handoff template; the system never states a ruling |
| SDC-11 Sharia basis, ISSC, certificate and KFS refs on every suggestion | Catalogue fields (ERD `product`) carried as `Recommendation.sharia_disclosure` in contract 1.2.0 (V-9): required on product suggestions and credit referrals in ISLAMIC deployments, absent in CONVENTIONAL ones; approved shape only with `sharia_status = SHARIA_APPROVED` and the approval references, demo shape only with `illustrative_sharia_assumption` and the watermark; KFS and Sharia-basis refs per locale (ar + en); an incomplete disclosure is suppressed, never rendered. Field details beyond file 20 (e.g. whether `issc_approval_ref` equals `sharia_approval_ref`, per-language certificates, Takaful approver) are [SHARIA-CONFIRM] |
| SDC-12, SDC-13 no promised return; Takaful never "insurance" | Catalogue schema; `CTL-HUMAN-REVIEW`; `CTL-TERMINOLOGY` |
| SDC-14 re-review on Sharia change | Artefact metadata `aaoifi_basis[]`, `hsa_resolution_refs[]`, `issc_resolution_ref`; a new or amended resolution moves dependants to `IN_SHARIA_REVIEW` (BLOCK) |
| SDC-15 Sharia audit evidence, never leaves `B-AE` | `CTL-AUDIT-LOG` FULL; audit export scoped to the cell |
| SDC-17 separation of duties | §4.3 |
| SDC-18 Islamic-window guard | Bootstrap refuses `institution_type: ISLAMIC` when the bank is flagged as an Islamic window until per-product segregation is designed (file 20 Q-L2) |

### 4.5 Late-payment charity semantics (data only)

Track B financing items may carry descriptive attributes such as `late_payment_treatment: CHARITY_UNDERTAKING` with an approved text reference. The subsystem **never** calculates, charges, waives, forecasts or decides any late-payment amount, never uses it as a feature, and shows it only as approved catalogue text in an explanation of a `CREDIT_REFERRAL` or an educational item. [REC] [SHARIA-CONFIRM]

## 5. Per-track synthetic data and catalogues

All items are **SYNTHETIC / ILLUSTRATIVE**, labelled as such, with no real product names, rates, fees, profit rates or terms (BQ-9 extended by DEC-56). Every credit-type item, conventional or Islamic, is reachable only as `CREDIT_REFERRAL` (no eligibility, limit, rate, profit rate, price or likelihood).

| Catalogue | Item (synthetic name) | Recommendation kind | Notes |
|---|---|---|---|
| `CAT-A-US-SYNTH` | Synthetic Everyday Checking, Synthetic Savings Pocket, Budgeting Tool, Bill Reminder Service, Financial Education Module | PRODUCT_SUGGESTION / SERVICE_SUGGESTION / EDUCATIONAL_CONTENT | USD; en-US |
| `CAT-A-US-SYNTH` | Synthetic Credit Card, Synthetic Personal Loan | CREDIT_REFERRAL only | No terms shown |
| `CAT-A-UK-SYNTH` | Synthetic Current Account, Synthetic Easy-Access Saver, Budgeting Tool, Savings Goals, Money Guidance Module | PRODUCT_SUGGESTION / SERVICE_SUGGESTION / EDUCATIONAL_CONTENT | GBP; en-GB |
| `CAT-A-UK-SYNTH` | Synthetic Credit Card, Synthetic Personal Loan | CREDIT_REFERRAL only | No terms shown |
| `CAT-B-AE-ISLAMIC-SYNTH` | Synthetic Wakala Savings Account; Synthetic Qard-based Current Account [ASSUME structure] | PRODUCT_SUGGESTION | AED; ar + en; `sharia_status` per §4.3 |
| `CAT-B-AE-ISLAMIC-SYNTH` | Synthetic Mudarabah Investment Deposit | PRODUCT_SUGGESTION with `CTL-HUMAN-REVIEW` | Profit-sharing, capital not guaranteed: always human review [REC] |
| `CAT-B-AE-ISLAMIC-SYNTH` | Synthetic Family Takaful Plan | PRODUCT_SUGGESTION with `CTL-HUMAN-REVIEW` | Takaful distribution rules [LEGAL-CONFIRM] (file 20) |
| `CAT-B-AE-ISLAMIC-SYNTH` | Synthetic Murabaha Personal Finance; Synthetic Tawarruq Personal Finance; Synthetic Ijarah Home Finance; Synthetic Islamic Card | **CREDIT_REFERRAL only** | Financing = credit; never ranked as a product suggestion; optional data-only `late_payment_treatment` (§4.5) |
| `CAT-B-AE-ISLAMIC-SYNTH` | Budgeting Tool, Savings Goals, Islamic Finance Education Module (terminology-checked) | SERVICE_SUGGESTION / EDUCATIONAL_CONTENT | Templates need Sharia approval in non-demo bundles |

Synthetic generators: one generator codebase, one seed per cell; outputs never mixed across cells. Personas per cell (e.g. Alex, NYC, 24, shift worker in `A-US`; a London salaried renter in `A-UK`; a Dubai or Abu Dhabi salaried customer with a Wakala account in `B-AE`) plus the shared control personas in each cell: minor (protected), age unknown, consent withdrawn, unknown jurisdiction, out-of-scope jurisdiction (`US-CA` in A, `GB` customer presented to Track B and `AE` customer presented to Track A), pending-review block, credit-request refusal. Merchant taxonomy per cell includes the `CTL-PROHIBITED-CATEGORY` categories so the Track B exclusion path is tested. Currency per cell; no FX conversion between cells. [REC]

## 6. Per-track models

- A model registry entry carries `track` and `cell_scope`. Track A and Track B models are **trained, evaluated, validated and registered separately**; no artefact, feature store, embedding or training set crosses tracks, synthetic ones included (keeps the rule simple and avoids a later "but it was synthetic" exception). [REC]
- Inside Track A, with real data (after 12 §H.4 condition 1 is lifted), models train per cell on that cell's data; a pooled US+UK model needs a separate transfer assessment [LEGAL-CONFIRM]. In the synthetic MVP one Track A rule baseline (`RULESET`: rules and thresholds only, no data) serves both cells; ML is trained, evaluated and fairness-tested per cell on that cell's synthetic data, the same pipeline as for real data, with no pooled training set (validation 2026-11-10; consistent with §2.1 and §5 "never mixed across cells").
- DEC-47 applies per track: rule baselines serve; ML stays `SHADOW`; `APPROVED` is refused until DEC-19 is resolved for that bank's model-risk policy (Track B's model-risk policy is a separate unknown, file 20 in progress).
- Fairness testing (DEC-21, DEC-48) runs per cell; minimum group size 100 per cell; synthetic group labels only.

## 7. LLM hosting per track

MVP stays template-only in both tracks (DEC-06; the LLM adapter port is disabled). If an LLM is ever enabled, by a separate MGR/SEC decision before PH-9:
- **Track B:** **no hosted or external LLM**; bank-hosted / on-premises model only (`llm.hosting: BANK_ON_PREM`), and customer data, prompts, logs and backups stay in the UAE on the bank's systems (UAE manual ch. 33.2; file 20 IQ3). Whether any third-party hosting or a vendor foundation model, even on-prem, is acceptable is open (file 20 IQ3) and is treated as **not allowed** until answered. Arabic and English outputs pass `CTL-TERMINOLOGY` and the number check; template fallback on any failure.
- **Track A:** per cell, in-region; `A-UK` prompts never leave the UK region and `A-US` prompts never leave the US region unless a transfer assessment exists [LEGAL-CONFIRM]. A hosted provider needs the same MGR/SEC decision as DEC-06.
- Any enablement remains a 12 §H.4 no-go trigger until its blocking item is resolved. [REC]

## 8. Languages, currency, accessibility

- Track A: en-US (`A-US`), en-GB (`A-UK`); USD and GBP formatting per cell.
- Track B: Arabic and English (`ar-AE`, `en-AE`); AED; the dashboard supports right-to-left layout and Arabic numerals as a locale option; explanation templates exist in both languages and both need translation review and Sharia terminology review (`CTL-TERMINOLOGY`). `CTL-EXPLANATION.languages` for Track B = `[ar, en]` [REC]. WCAG 2.2 AA (DEC-41) applies in both directions.

## 9. Shared vs separate

| Item | Shared across tracks | Track A (A-US / A-UK) | Track B (B-AE) |
|---|---|---|---|
| Core code (M1-M9, PDP/PEP, resolver, assembler) | **Shared** (same release images) | - | - |
| Contract v1 (OpenAPI, events), error model, scopes | **Shared** | - | - |
| Control catalogue (incl. 4 new controls), DSL, pack schema | **Shared** (market-neutral) | - | - |
| Deployment profile | - | `PROFILE-TRACK-A` | `PROFILE-TRACK-B` |
| Deployment / runtime | - | One profile, one release, two cells | Separate deployment, bank on-prem |
| Database, subject keys, audit and decision logs, object store | - | **Separate per cell** | Separate |
| Jurisdiction packs | - | US, US-NY, US-NY-NYC; GB, GB-ENG, GB-ENG-LDN | AE, AE-<emirate> |
| Institution overlay | - | none | `PACK-INST-ISLAMIC` |
| Catalogue, templates, reason-code texts | Reason-code IDs shared; texts per track | `CAT-A-US-SYNTH`, `CAT-A-UK-SYNTH`; en | `CAT-B-AE-ISLAMIC-SYNTH`; ar + en |
| Models, features, training sets, registry | - | Track A registry (per-cell training for real data) | Track B registry |
| Synthetic generator | Code shared | Seeds and outputs per cell | Seed and output per cell |
| IdP realm, signing keys (deployment, demo-class), secrets | - | Track A realm and keys | Track B realm and keys |
| Approvers | Role names shared | Legal approver (interim role, DEC-63) | Legal approver + Sharia approver |
| Cross-cell analytics | - | Aggregate-only, non-personal (portfolio, §10) | Never combined with Track A |
| LLM | Adapter port shared (disabled) | In-region per cell, if ever enabled | Bank on-prem only, if ever enabled |

## 10. Contract impact (market-neutral; applied in `contracts/`)

- `GET /v1/capabilities` adds a read-only `deployment_profile` object: `profile_id`, `profile_version`, `track` (reference code), `institution_type` (`CONVENTIONAL | ISLAMIC`), `cell_id`, `currencies[]`, `locales[]`, `catalogue_id`, `institution_overlay` (pack ref or null). No request field can set or override any of these (DEC-61).
- Decision records (`/v1/policy/decisions/{id}`, audit records) add `profile_id`, `track`, `institution_type` and `cell_id` so replay is exact.
- `JurisdictionContext` is unchanged; a jurisdiction outside the serving cell's scope resolves to `UNSUPPORTED_JURISDICTION` (already in the contract).
- The three GATE-3 gaps are closed by DEC-62 (customer list, stored transactions, portfolio): tokenised identifiers only, purpose-bound scopes, aggregate-only portfolio with small-cell suppression; see file 12 §H.7 and `contracts/role-scope-matrix.md` §6.
- Contract 1.2.0 (validation V-8, V-9): the customer list becomes the purpose-bound `searchCustomerDirectory` (§2.3, DEC-68; `listCustomers` deprecated, not implemented), and recommendations carry the SDC-11 `sharia_disclosure` block in ISLAMIC deployments only (§4.4).
- Reason codes added as reference data: `SHARIA_REVIEW_PENDING`, `CATALOGUE_ITEM_NOT_ELIGIBLE`, `TERMINOLOGY_VIOLATION_FALLBACK`, `PROHIBITED_CATEGORY_EXCLUDED`, `CELL_SCOPE_MISMATCH`.

### 10A. Aggregate disclosure control for the portfolio summary (validation 2026-11-10)

A minimum group size alone does not stop re-identification: overlapping filters, successive snapshots and totals from another dimension can recover a hidden cell. `GET /v1/portfolio/summary` therefore follows these rules (tightens DEC-62; file 12 §H.7.3; contract 1.1.2 descriptions). Design, not code; every value is config. [REC]

| # | Control | Rule |
|---|---|---|
| AG-1 | Fixed shapes only | One dimension per call from the fixed list (segment, churn band, need, retention action, jurisdiction). No cross-tabulation, no free filter, no customer attribute filter, no sort or top-N by customer |
| AG-2 | Restricted jurisdiction filter | The optional `jurisdiction` filter accepts only a cell root (`US`, `GB`; `AE` in Track B) from reference data, never a nested subtree, so `US` minus `US-NY` style differencing is impossible. The `JURISDICTION` dimension reports one fixed level per cell |
| AG-3 | Published snapshots only | `as_of` must equal a published batch snapshot date (monthly, BQ-1); any other value is 400. This limits differencing over time to real snapshot changes, which AG-5 then blurs |
| AG-4 | Threshold floor | `min_cell_size` default 20 [ASSUME] is a floor: config may raise it, never lower it (tighten-only). A count below it is `null` with `suppressed: true` |
| AG-5 | Rounding | Every published count is rounded to a multiple of `rounding_base` (config, default 5 [ASSUME]) after suppression; each count is rounded on its own and no total is published. No random noise in v1, so outputs stay deterministic and replayable (BQ-11 KPI 8) |
| AG-6 | Complementary suppression | In any dimension where every customer falls in exactly one cell (churn band, jurisdiction), either no cell or at least two cells are suppressed, and the suppressed cells together hold at least `min_cell_size`; otherwise the next-smallest cell is also suppressed. Because the population total is derivable from such a dimension, the same rule protects multi-membership dimensions (segment, need) against subtraction from that total |
| AG-7 | Suppress in the cell, then combine | Each cell computes and suppresses its own aggregates; only that post-suppression, rounded output leaves the cell, through the same contract call. No component reads another cell's database or result store. In a combined Track A view a code is suppressed if it is suppressed in any contributing cell (no partial sums), and the combined value is re-checked against AG-4 and re-rounded. Never across tracks |
| AG-8 | Population | Adults only for commercial dimensions; minors, age-unknown and consent-withdrawn customers excluded; the population follows the same purpose and consent rules as `SEGMENTATION_INSIGHT` |
| AG-9 | Query audit and rate | Every call writes an `audit_record` (actor, dimension, filter, `as_of`). A per-user daily quota (config, default 200 calls [ASSUME]) returns 429 when exceeded, on top of the client rate limit (DEC-31). Repeated filter or `as_of` variations by one user raise a security alert for review (SEC); the alert never relaxes a rule |
| AG-10 | Fairness group size vs disclosure floor | They answer different questions and are coherent: 20 is the disclosure floor for any published count; 100 per cell (DEC-48) is the statistical floor for a fairness rate. Fairness reports show groups under 100 as "insufficient" with no rate, and never print a group count under 20; the stricter rule applies wherever both do |

## 11. Demo plan (both tracks)

Both demos: `mode=STANDALONE`, `demo=true`, synthetic only, `ILLUSTRATIVE` bundles (DEC-44), watermark on every output, no hosted LLM.

**Track A demo (about 12 minutes; extends file 09 §5 and §7):**
1. Framing slide: two tracks, one core; credit stays with the credit platform.
2. `A-US` cell: Alex, NYC, 24, shift worker; the file 09 §5 flow (transactions, segments, needs, churn rule baseline, watermarked recommendations, retention outcome, assistant refusal).
3. `A-UK` cell: a London customer; same flow, en-GB, GBP; capabilities show `cell_id: A-UK` and GB packs `PENDING_LEGAL_REVIEW` (file 19 in progress) with illustrative assumptions.
4. Cell isolation: the London customer token is not found in the `A-US` cell; a NYC-resident, London-booked customer resolves to UNKNOWN and is denied commercial activity.
5. Portfolio view: aggregate-only counts across both cells (no tokens, small cells suppressed).
6. Shared controls: minor (16) protected, `US-CA` out of scope, consent withdrawal, pending-review fail-safe in the simulator.

**Track B demo (about 10 minutes; separate deployment, separate URL):**
1. Capabilities: `institution_type: ISLAMIC`, `PACK-INST-ISLAMIC` loaded, Arabic + English, AED, no LLM.
2. A synthetic Dubai/Abu Dhabi customer: the same core flow; dashboard switched to Arabic (RTL) and back.
3. Recommendations come only from the synthetic Islamic catalogue (Wakala savings, budgeting tool); Mudarabah and Takaful items go to the human-review queue; Murabaha/Ijarah items appear only as `CREDIT_REFERRAL`.
4. Terminology: the explanation says "finance" and "profit rate"; a test template containing "loan interest" is rejected by `CTL-TERMINOLOGY` and the template fallback is shown.
5. Prohibited categories: spending in an excluded category appears in budgeting insight with neutral wording, but no offer or feature uses it; no religion inference.
6. Sharia approval: flip a catalogue item to `IN_SHARIA_REVIEW` (new version) and the suggestion becomes BLOCK with `SHARIA_REVIEW_PENDING`; watermark "illustrative - not a Sharia ruling".
7. Isolation: Track A customer token and Track A models are unknown to Track B; a `GB` jurisdiction presented to Track B is `UNSUPPORTED_JURISDICTION`.
8. Shared controls: minor protected (age threshold per config, [ASSUME 21] for AE), credit-request refusal ("approve my financing" -> refusal + handoff template).

### 11A. Human review on Track A outcomes that may be significant decisions

Pending file 19 UQ4 (rows U02, U07: UK GDPR Arts 22A-22D [LEGAL-CONFIRM]), Track A keeps meaningful human involvement on three outcome types, in both cells (one rule for the track; simplest and restrictive):
- **Offer suppression:** suppression still applies immediately (fail-safe), and a `review_item` lets `relationship_staff` confirm or escalate it; the customer is never told a product is unavailable because of us. **The review can never release the suppressed offer** (validation 2026-11-10): for a review item whose parent recommendation is suppressed, `CLEARED_TO_SHOW` is refused; `REJECTED` records "suppression confirmed" and `ESCALATED` routes the case to COMP. A wrong input is corrected at its source and the evaluation re-runs under the same policies.
- **Retention-offer allocation:** every retention action needs `CTL-HUMAN-REVIEW` before it is shown or acted on (already true for `RA_HUMAN_CHECKIN`; now all `RA_*` except `RA_NO_ACTION`).
- **Vulnerability routing:** `CTL-VULNERABILITY-ROUTE` always ends with a person (already the design; now explicit for Track A).
Encoded as `CTL-HUMAN-REVIEW` policies in `PACK-GB` and mirrored in `PACK-US` as an internal-policy tightening (DEC-66). These policies never carry `illustrative_assumption: ASSUME_NOT_APPLICABLE`, so an `ILLUSTRATIVE` demo bundle keeps the review step (validation 2026-11-10).

## 12. Risks and open items

| # | Item | Owner | Blocks | Label |
|---|---|---|---|---|
| TT-R1 | UK applicability (file 19) and UAE regulatory + Sharia applicability (file 20) are in progress; all GB and AE packs stay PENDING_LEGAL_REVIEW = BLOCK | LEG / Sharia | GB and AE pack content (Story 7.4 equivalents), not the mechanism | [LEGAL-CONFIRM] [SHARIA-CONFIRM] |
| TT-R2 | Whether the US and UK cells may ever share processing (international transfer mechanism) | LEG / PRIV | Pooled models, cross-cell personal views (none planned) | [LEGAL-CONFIRM] |
| TT-R3 | Track B bank identity, licence and onshore/free-zone status; emirate | MGR / bank | AE pack scope; DEC-10 equivalent for Track B | [UNKNOWN] |
| TT-R4 | Sharia lists (prohibited terms, categories, product structures) and the ISSC approval process | Sharia | Track B non-demo bundles | [SHARIA-CONFIRM] |
| TT-R5 | Age of majority per jurisdiction (US, GB 18; AE [ASSUME 21] as the restrictive default) | LEG | Nothing (restrictive default applies) | [LEGAL-CONFIRM] |
| TT-R6 | Two demos and one more cell raise build effort (more catalogues, packs, templates, Arabic RTL) | DEV | Schedule | [REC]: build the core once, Track A first, then Track B as configuration |
| TT-R8 | UK operating entity type: UK-incorporated bank vs UK branch of the NYC bank vs other (file 19 UQ1, Q-U2); decides which UK rows apply (e.g. U16, U20) and who is the UK data controller | MGR / LEG | GB pack scope (not the mechanism) | DEC-65 BLOCKED-EXTERNAL |
| TT-R9 | Whether in-app/push outputs are telemarketing (file 20 IQ4) | COMP / LEG | Nothing (restrictive reading applies) | DEC-67 |
| TT-R7 | Named legal approver (both tracks) and named Sharia approver (Track B) | MGR | Any `APPROVED`/`ACTIVE` pack (12 §H.4 condition 2) | DEC-11, DEC-63 |

## Change log
- 2026-11-10: created. Records the developer's two-track scope decision (DEC-50 resolved) and the delegated design decisions DEC-51..DEC-64 (file 12 §H.7). Supersedes DEC-13 and DEC-28. Files 19 and 20 referenced as in progress. No application code.
- 2026-11-10 (alignment): aligned with files 19 and 20 now written: SDC-01..SDC-18 mapping (§4.4), SoD per SDC-17, Track B contact caps (E08, pending IQ4; DEC-67), Track B no hosted/external LLM and data stays in the UAE (IQ3), Track A human review on suppression / retention allocation / vulnerability routing (U02, UQ4; DEC-66), UK entity type open (UQ1; DEC-65).
- 2026-11-10 (readiness re-run fixes): §4.3 submit transition to `IN_SHARIA_REVIEW` defined; illustrative Sharia assumption uses only the attribute form `illustrative_sharia_assumption: ASSUME_COMPLIANT` (§4.2).
- 2026-11-10 (independent two-track validation, handoff/two-track-validation-report-2026-11-10.md): new §2.1 cell isolation checklist (logs, telemetry, audit/replay, outbox, backups, analytics, training inputs, support/admin access, keys), §2.2 customer token scheme (not reversible by the subsystem, per-cell/per-track keyed, no cross-context linkage), §3.1 profile integrity (Ed25519/ECDSA signature, per-track keys in KMS/HSM, maker-checker signing, track-bound trust anchors, fail-closed list incl. rollback, no runtime/env/flag override), §10A aggregate disclosure control AG-1..AG-10; §4.3 SoD aligned with SOD-9 plus per-user checks; §6 ML per cell in the synthetic MVP too; §11A suppression review cannot release an offer and DEC-66 policies carry no not-applicable illustrative assumption. Recorded as tightenings of DEC-51, DEC-52, DEC-55, DEC-57, DEC-62, DEC-66 in file 12 §H.7.3. No new DEC, no code.
- 2026-11-10 (contract 1.2.0, V-8 and V-9): new §2.3 purpose-bound customer directory (DEC-68); §4.4 SDC-11 row states the contract 1.2.0 disclosure block and its approval-state rules; §10 bullet. No Sharia content added; nothing here is a Sharia ruling or a legal conclusion.
