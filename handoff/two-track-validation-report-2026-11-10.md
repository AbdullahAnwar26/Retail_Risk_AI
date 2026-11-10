# Two-Track Independent Validation Report

Date: 2026-11-10. Project: `G:\Retail_Risk_AI`. Validator: an independent Opus agent, separate from the session that wrote the two-track update. **Planning only.** No application code, migrations, infrastructure or datasets were written. PH-4 was not started. `.claude/`, `CLAUDE.md` and `handoff/manager-approval-pack-2026-11-10.md` were not touched. Nothing here is a legal or Sharia approval, and no licence, approver, manager decision or new DEC number was invented.

## 1. Verdict in brief

- **The architecture is internally consistent after this pass.** Track A cells, Track B isolation, the tighten-only Sharia overlay, the signed profile and the contract now agree across planning/12, 14, 15, 20, 21, the contract, role matrix, ERD, PRD, spine and epics.
- **Readiness: CONCERNS, no Blocker.** The earlier findings T-1..T-11 are confirmed fixed. This pass found 9 new issues (V-1..V-9). V-1..V-7 are fixed as design tightenings. V-8 and V-9 (Medium) were closed afterwards in contract 1.2.0 (§10).
- **The most serious gap (V-1, High) is fixed in the design.** The portfolio summary relied only on a minimum group size of 20, which could be lowered to 1 and could be bypassed by differencing. It now follows aggregate disclosure control AG-1..AG-10 (planning/21 §10A).
- **GATE-1, GATE-2 and GATE-3 are developer-ratified on delegated authority, with a manager countersignature recommended.** They are not manager-approved. The earlier word "signed" was corrected.

## 2. Files inspected

- **Repository rules and handoff:** AGENTS.md, CLAUDE.md, .claude/settings.json (read only), handoff/status-report-2026-11-10.md.
- **planning/:** 00, 07, 08, 09, 11, 12 (§A, §H.1-§H.7), 13 (by reference), 14, 15, 18, 19, 20, 21.
- **contracts/:**
  - openapi/retail-intelligence-v1.yaml (1.1.1, now 1.1.2)
  - events/ (envelope and 15 type schemas)
  - data-model/erd.md, ports.md
  - role-scope-matrix.md and .json
  - README.md, redocly.yaml
- **_bmad-output/planning-artifacts/:**
  - README.md
  - prd.md
  - ARCHITECTURE-SPINE.md
  - epics.md
  - EXPERIENCE.md, DESIGN.md
  - implementation-readiness.md
  - brief.md (by grep)
- **Readiness method:** .claude/skills/bmad-sprint-planning/references/readiness-gate.md.

## 3. Files changed, with reasons

Every edited file has a change-log line.

| File | Change | Reason |
|---|---|---|
| planning/21 | §2.1 cell isolation checklist; §2.2 customer token scheme; §3.1 profile integrity; §4.3 SoD aligned with SOD-9 plus per-user checks; §6 ML per cell in the synthetic MVP; §10A aggregate disclosure control AG-1..AG-10; §11A suppression review cannot release an offer, and DEC-66 policies carry no not-applicable illustrative assumption | V-1..V-7 |
| planning/12 | New §H.7.3 records the tightenings of DEC-51/61, 52/53, 55, 57, 62 and 66, with no new DEC and no status or count change. It adds an authority note on DEC-01, DEC-02, DEC-12 and DEC-63 item 1 | Record every decision change. Flag items beyond developer authority |
| planning/14 | §9 Track B SoD aligned (it said only "never also `policy_author`") | V-6 |
| planning/15 | Tracks-and-cells note: token scheme, portfolio control, profile integrity | V-1, V-3, V-4 |
| planning/11 | §4 sequencing note cited DEC-63 for the GATE ratification; it now cites file 12 §H.7.2 | Wrong citation |
| planning/20 | SH01 cited SDC-13 instead of SDC-15. SH03 cited SDC-06 instead of SDC-09. No Sharia content changed | Wrong cross-references |
| planning/00 | §5B row and change-log line | Work log |
| contracts/openapi (1.1.1 → 1.1.2) | Descriptions on `Token`, `DeploymentProfile`, `getPortfolioSummary`, `PortfolioCell`, `transitionReviewItem` and `recordPackShariaApproval`. `min_cell_size` minimum changed from 1 to 20 (a tighten-only floor). Missing 1.1.1 change-log entry added. No new field, operation or scope | V-1, V-3..V-6 |
| contracts/role-scope-matrix.md and .json | SOD-9 per-user text (in md and JSON); §3 data-cell note; JSON `info_version` changed from 1.1.0 to 1.1.2. No matrix cell changed | V-2, V-6, version drift |
| contracts/README.md | Version, last check run, change log | Contract 1.1.2 |
| PRD | FR-51, FR-69, FR-70, FR-73, FR-75, FR-81 tightened | Carry V-1..V-7 forward |
| Spine | AD-21, AD-23, AD-26 | Same (lint 0) |
| epics.md | Acceptance criteria tightened in Stories 2.11, 2.12, 6.10, 7.9, 7.10, 8.5 (SDC-10 handoff) and 9.7; still 86 stories | Same. Story 2.12 said an aggregate path "may read both cells" |
| EXPERIENCE.md | Portfolio view row: rounded counts, one fixed dimension | V-1 |
| implementation-readiness.md | New re-run section, actual verdict, GATE-3 wording corrected | Task 7 |
| handoff/status-report-2026-11-10.md | ERD 66 entities (was 63), contract 1.1.2, GATE-3 wording, readiness result, next steps | Reflect actual results |

## 4. Checks run (exact results, after the fixes)

| Check | Command or source | Result |
|---|---|---|
| Redocly lint | `npx @redocly/cli@1.34.5 lint --config contracts/redocly.yaml` from the scratch folder, `npm_config_cache` set to scratch | Baseline (1.1.1): valid. After (1.1.2): "Your API description is valid", exit 0. That is 0 errors and 0 warnings |
| Python contract and jsonschema check | `check_contract.py` (scratch; jsonschema 2020-12) | `operations: 46; component schemas: 162; customer-outcome schemas scanned: 43; event schema files: 16; event examples validated: 15; OK: no errors`. It also confirmed: every response has the `X-Not-For-Credit-Decisioning` header; every JSON body requires `not_for_credit_decisioning`; no forbidden credit term; every event rejects data without the label |
| Role matrix vs OpenAPI scopes | `ph3b/verify_matrix.py` | `roles=10 scopes=30 operations=46 screens=13 actions=14 jurisdiction_conditions=8 cells=650 deny_cells=487 negative_tests=487 md_cells_checked=650 PASS` |
| Cross-check of ERD, ports and contract | `ph3b/crosscheck.py` | `erd_entities=66 diagrams_entities=66 driven_ports=18 operations=46 mapped=46 events=15 PASS` |
| Spine lint | `uv run .claude/skills/bmad-architecture/scripts/lint_spine.py --workspace <spine run folder>` | `ok: true, total_findings: 0` (before and after the edits) |
| Trace probe (readiness) | Python over the PRD and epics | 82/82 FRs in the epics inventory and cited; 86 stories, no duplicates; 20 UX-DRs; no forward dependency |
| T-fix probes | grep | No `ILLUSTRATIVE_ASSUMED_COMPLIANT` residue; `ritrack`/`ricell` in the envelope; RTL in DESIGN.md; A14 in Story 9.9; Stories 9.10 and 9.11 present |
| DEC register | Manual review plus counts | 67 DECs. §H.7 count confirmed: 47 DECIDED, 5 DECIDED-DEFAULT, 13 BLOCKED-EXTERNAL, 2 SUPERSEDED. BQ: 10 DECIDED, 1 DECIDED-DEFAULT. No duplicate with a conflicting status |
| Mermaid | not run | No diagram was changed |

## 5. Findings by validation area

**1. Track A cells.**
- **Cell separation:** database and region separation was already designed. Telemetry, outbox, backups, support/admin access, training inputs and per-cell sessions were not covered explicitly (V-2). Fixed in planning/21 §2.1.
  - Each cell runs its own instance with its own `deployment_id`, so JX-1 binds a token to one cell.
  - Telemetry is token-free.
  - Audit, replay, outbox and backups are per cell.
  - There is no Track-wide administrator with access to personal data.
- **Portfolio controls (V-1, High) were insufficient.** Now fixed:
  - one fixed dimension per call;
  - the jurisdiction filter accepts only a cell root;
  - `as_of` accepts only published snapshot dates;
  - 20 is a floor that config can raise but never lower;
  - counts are rounded to base 5 [ASSUME];
  - complementary suppression covers totals derived from another dimension;
  - each cell suppresses its own counts before any combination, and a code suppressed in any cell stays suppressed;
  - every call is audited, and a per-user quota of 200 per day applies [ASSUME].
- **Fairness group size 100 vs disclosure floor 20:** these are coherent. 100 is the statistical floor for a fairness rate; 20 is the disclosure floor for any published count. The stricter rule applies where both do (AG-10).
- **DEC-66** is consistent in 08, 09, 12, 14, 18, 21, the PRD, the spine and the epics. The gap V-5 is fixed: a reviewer could clear a suppressed offer, and an illustrative assumption could switch the review off in the demo.

**2. Track B.**
- Separate deployment, data, models, keys, logs, identity and backups; no external LLM; Arabic and English with RTL; tighten-only overlay; Sharia-approved products only (`sharia_status`, never an illustrative value). All consistent.
- The four controls (`CTL-CATALOGUE-ELIGIBILITY`, `CTL-TERMINOLOGY`, `CTL-PROHIBITED-CATEGORY`, `CTL-SHARIA-APPROVAL`) are consistent across 14, 20, 21, the PRD, the spine and the epics.
- Separation of duties was inconsistent (V-6) and is now fixed. The strictest form is used everywhere, matching SOD-9.
- Contact caps (DEC-67) are consistent.
- SDC-10 (no Sharia rulings) is present in the PRD and file 21, but no story had a test for the Sharia-question handoff. An AC was added to Story 8.5.
- The SDC-11 disclosure fields were missing from the contract and ERD (V-9); closed in contract 1.2.0 (§10).

**3. Signed profile (V-4).** Now specified in planning/21 §3.1:
- Ed25519 detached signature over the canonical form (ECDSA P-256 if the bank requires FIPS modules [ASSUME]).
- One signing key per track, held in KMS/HSM. Track B keys are bank-held in the UAE.
- Maker-checker signing by the bank's release function, never by the vendor for `demo: false`.
- Track-bound trust anchors carried in the image.
- Fail-closed on a missing, invalid, altered, incompatible or rolled-back profile, or a key bound to another track.
- No environment variable, flag, config file, endpoint or request can switch tracks. There is no hot reload.

**4. Contract.**
- **Customer list:** tokens only. The token scheme was unspecified (V-3); it is now defined as not reversible by the subsystem, keyed per cell and track, with no cross-context linkage.
- **Transactions:** separate scope `ri.transactions.read` (analyst and relationship staff only); purpose `INSIGHT`; jurisdiction check on the request context; no raw description, counterparty or merchant name. This is correct.
- **Portfolio:** see V-1.
- **Sharia approval call:**
  - `ri.sharia.approve`, held only by `sharia_approver`;
  - `sharia_approval_ref` required for SHARIA_APPROVED;
  - reason required for a rejection;
  - maker-checker, now also checked against the legal approver;
  - audited;
  - 422 outside ISLAMIC deployments;
  - 403 for everyone while no person is named.
- **Excessive exposure:** `listCustomers` was reachable by `service_host` and `auditor` through `ri.insights.read` with no purpose binding (V-8); closed in contract 1.2.0 (§10).

**5. Safeguards.** All present and consistent:
- no credit decision; credit appears only as `CREDIT_REFERRAL`;
- `not_for_credit_decisioning` is required on every response and event (checked by script);
- synthetic data only;
- the two-tier minors block list is unchanged, Tier 1 is never relaxed, and the AE age of majority is 21 [ASSUME];
- no sensitive or proxy targeting, and religion is never inferred;
- packs stay `PENDING_LEGAL_REVIEW` = BLOCK until a named approver is recorded;
- Sharia approval is separate human governance;
- ML stays SHADOW until DEC-19;
- explanations are template-only by default;
- no external LLM in Track B;
- human review where specified.

**6. Decisions.**
- No duplicate DEC with a conflicting status. §A differs from §H only where §H overrides by design.
- DEC-13 and DEC-28 are explicitly SUPERSEDED.
- GATE wording was corrected in planning/11, the readiness document and the status report.
- **Beyond developer authority (flagged, not reversed):**
  - DEC-01 and DEC-02: the ownership split with the credit platform;
  - DEC-12: which bank function owns approval and release;
  - DEC-63 item 1: the interim approver roles.
  - These are now recorded as working positions pending manager countersignature (12 §H.7.3). No safeguard depends on them.

**7. Readiness.** Re-run, with the verdict recorded in implementation-readiness.md (CONCERNS, no Blocker).

## 6. Remaining issues and contradictions

| ID | Severity | Issue | Recommended action |
|---|---|---|---|
| V-8 | Medium | **CLOSED (design, contract 1.2.0, §10).** `GET /v1/customers` used `ri.insights.read`, so `service_host` and `auditor` can enumerate the tokens and protection status in their claim; the call has no `purpose` | In contract 1.2.0 (planning), add a staff-only picker scope and a `purpose` parameter, regenerate the matrix, and re-run `verify_matrix.py`. Decide before Stories 6.10 and 9.1 |
| V-9 | Medium | **CLOSED (design, contract 1.2.0, §10; content [SHARIA-CONFIRM]).** SDC-11 fields (`sharia_basis_ref`, `issc_approval_ref`, `sharia_certificate_ref`, `kfs_ref`) are in no contract or ERD schema | Add them in contract 1.2.0 and the ERD `product` entity before Track B recommendation stories (already declared in planning/21 §4.4) |
| L-1 | Low | Stored transactions: no maximum date window; `txn_ref` is a free string; the `INTEREST` kind code needs a terminology-checked UI label in Track B | Cap the window at the feature lookback, make `txn_ref` a token, and take UI labels from reference text that passes `CTL-TERMINOLOGY` |
| L-2 | Low | A suppression-confirmation review item is recognised only through its parent recommendation's state | Consider an explicit `item_type` value in 1.2.0 |
| L-3 | Info | Older texts (file 18 §1, the readiness section "GATE-3 verdict: not yet signable", file 12 §H.4 "provisionally passed") are historical and superseded by later sections | Leave as history; the newest sections are authoritative |

## 7. Blockers on legal, Sharia and management (cannot be resolved by the agent)

- **Licence and charter per track:**
  - DEC-10 (US/NY charter; UAE licence, file 20 IQ1);
  - DEC-65 (UK bank or branch, and the UK data controller).
- **Named people:**
  - DEC-11 and DEC-63: a named legal approver per track and a named Sharia approver (ISSC/SCF) for Track B;
  - until then no pack reaches APPROVED or ACTIVE.
- **Manager:**
  - countersignature of the developer-ratified GATE-1, GATE-2 and GATE-3 batches, including the DEC-01, DEC-02, DEC-12 and DEC-63 working positions;
  - GATE-2 version pins and the UI design system.
- **Legal confirmations:**
  - file 13 §6;
  - file 19 UQ1 and UQ4 (DEC-65, DEC-66);
  - file 20 IQ3 (DEC-58) and IQ4 (DEC-67);
  - DEC-60 (AE age of majority);
  - TT-R2 / DEC-52 (any cross-cell transfer; LEG/PRIV before real data);
  - DEC-18, DEC-20, DEC-22, DEC-23, DEC-25, DEC-35 (by GATE-12).
- **Sharia:** term, category and product-structure lists, and the ISSC process (TT-R4; all SDC content [SHARIA-CONFIRM]).
- **Model risk:** DEC-19 (ML stays SHADOW) and DEC-21 (fairness proxies; group size 100 per DEC-48).
- **Host and security:** DEC-07, DEC-30, DEC-33.

## 8. Is the architecture internally consistent?

Yes, at planning level, after this pass. The two items left open by this pass (V-8, V-9) were closed in contract 1.2.0 (§10); no finding remains open. No safeguard was weakened: every change in this pass tightens a rule or corrects a reference.

## 9. Recommended next action

1. Done 2026-11-10: V-8 and V-9 closed in the planning-only contract 1.2.0, role matrix regenerated, ERD extended, all checks re-run (§10).
2. Present the manager pack, which another agent is preparing, for countersignature: GATE-1, GATE-2 and GATE-3, the four working positions, the licence questions per track (DEC-10, DEC-65, file 20 IQ1), and the named legal and Sharia approvers (DEC-11, DEC-63).
3. Start PH-4 (the synthetic generator, per cell) only on the developer's explicit implementation instruction, recorded in file 12, after step 1.

## 10. Addendum: V-8 and V-9 closed in contract 1.2.0 (2026-11-10)

Planning and contract only (Opus). No application code, migration, infrastructure or dataset; PH-4 not started; `.claude/` and `CLAUDE.md` untouched; no BMAD build skill used. Nothing here is a legal or Sharia approval.

**V-8 design (DEC-68, new; file 12 §A and §H.7.4; file 21 §2.3).**
- Inventory: `listCustomers` (analyst, relationship_staff, auditor, service_host via `ri.insights.read`) was the only directory path. Other token-returning lists are the review queue (`ri.review.read`), the audit export (`ri.audit.export`) and events to the host (file 12 §H.7.4 table).
- New `GET /v1/customer-directory` (`searchCustomerDirectory`) under new staff-only scope `ri.customers.list` (analyst, relationship_staff). Deny by default for auditor, service_host and every machine client; the host holds the token mapping itself.
- Required `access_purpose`, closed list `CUSTOMER_SERVICING` (relationship_staff) and `INSIGHT_REVIEW` (analyst), registered per role (matrix Table 5) and per client (issuer registry, none registered).
- Exact token lookup = operational access; anything else = enumeration, which needs `segment_code` or `data_quality_status`. Page <= 25, query total <= 100, per-user daily quotas 200 calls / 500 rows [ASSUME], cursor bound to user, purpose and filters. Per-cell and per-track boundaries unchanged (ROW_FILTER, file 21 §2.1, §2.2).
- Audit: `AccessAuditDetail` on `AuditExportRecord` and ERD `audit_record` columns for every call, allowed or denied (account, client, role, scope, purpose, access kind, filters, result count, outcome, denial code; timestamp, cell, track). Review-queue and export calls are also audited.
- Denials (extensible `ErrorCode`): `ACCESS_PURPOSE_INVALID` 400, `ACCESS_PURPOSE_NOT_REGISTERED` 403, `NARROWING_FILTER_REQUIRED` 400, `ACCESS_QUOTA_EXCEEDED` 429, `FORBIDDEN_SCOPE` 403.
- Auditor: read-only; reads access records through the export, opens a customer only by a token from a record; S01 now denied.
- `listCustomers` deprecated and not implemented (404 for every role; 10 matrix tests).

**V-9 design (implements DEC-55 and file 20 SDC-11; no new DEC).**
- `Recommendation.sharia_disclosure` (GOVERNANCE schema) is exactly one of two shapes:
  - approved: `sharia_status` const `SHARIA_APPROVED`, `sharia_approval_ref`, `issc_approval_ref`, `contract_type` (no `CONVENTIONAL_*`), `sharia_basis_ref` and `kfs_ref` per profile locale (min 2, ar + en), `sharia_certificate_ref`, optional `contract_consequences_ref`, `education_ref`, `operator_sharia_approval_ref`;
  - illustrative (demo): `sharia_status` `NOT_REVIEWED` / `IN_SHARIA_REVIEW`, `illustrative_sharia_assumption: ASSUME_COMPLIANT`, notice "illustrative - not a Sharia ruling", no approval reference.
- Required on `PRODUCT_SUGGESTION` and `CREDIT_REFERRAL` when the response's `institution_type` is `ISLAMIC`; forbidden otherwise (Track A). Outside an `ILLUSTRATIVE` bundle only the approved shape validates (also in the composite). An incomplete disclosure is suppressed, never shown. References are opaque refs to bank-approved documents, never free text.
- ERD: `product` disclosure columns with load validation; new `message_template` entity (Sharia state, language pairs, typed slots); `recommendation.sharia_disclosure` snapshot (SDC-15 evidence). Event `recommendation.proposed`: optional `sharia_disclosure_required` and a Track B example; the block itself stays API-only.

**Version decision: 1.2.0 (minor).** Every change is additive under contracts/README.md: a new operation, a new scope, new optional or response-only fields, and new values of an extensible list. Tightening `listCustomers` in place would have been breaking (a newly required parameter and tighter validation). It is therefore deprecated and marked not implemented. No 1.1.x implementation, tag or consumer exists, so no served consumer loses it. The strict alternative (serve it six months under DEC-31) is recorded in file 12 §H.7.4 and not recommended.

**Files changed (each with a change-log line).**
- Contracts: OpenAPI (1.2.0), `events/com.retailintel.recommendation.proposed.v1.schema.json`, `data-model/erd.md`, `ports.md`, `role-scope-matrix.md` and `.json` (0.3.0), `README.md`.
- Planning: 12 (DEC-68, §H.7.4, DEC-62 item 1 note), 21 (§2.3, §4.4, §10), 15 (customer-list row), 20 (change-log line only), 00 (§5B row).
- BMAD: PRD (FR-72, FR-80), spine (AD-21, AD-25), epics (FR80, UX-DR20, Stories 6.1, 6.10, 9.2, 9.11), EXPERIENCE, implementation-readiness.
- Handoff: this report, the manager pack ("Validation results"), the status report.

**Checks (exact).**

| Check | Result |
|---|---|
| Redocly CLI 1.34.5 (scratch, `npm_config_cache` in scratch) | "Your API description is valid", exit 0, 0 errors, 0 warnings |
| `check_contract.py` (jsonschema 4.26, 2020-12) | operations 47; component schemas 174; customer-outcome schemas scanned 44; event schema files 16; event examples validated 16; OK, no errors |
| `v89/probe.py` (new behavioural probes) | probes=30 failures=0 |
| `v89/verify_matrix.py` (extended) | roles=10 scopes=31 operations=47 screens=13 actions=14 jurisdiction_conditions=8 purposes=2 deprecated_operation_tests=10 cells=680 deny_cells=514 negative_tests=514 md_cells_checked=680 PASS |
| `v89/crosscheck.py` (extended) and original `ph3b/crosscheck.py` | erd_entities=67 diagrams_entities=67 driven_ports=18 operations=47 mapped=47 events=15 PASS (both) |
| `lint_spine.py` | ok: true, total_findings: 0 |
| Readiness trace probe (limited re-run) | 82/82 FRs in inventory and cited; 86 stories, no duplicates; 20 UX-DRs; no forward prerequisite |
| Mermaid (ERD, mermaid-cli 11.16.0, scratch) | 6/6 rendered (exit 0), D4 includes `message_template` |

**Remaining concerns.**
- Low, residual: `ri.review.read` (work queue) and `ri.audit.export` (audit export) still return tokens to `auditor` and `service_host`. They are bounded, audited and unchanged so 1.2.0 stays compatible; SEC to confirm at GATE-12.
- The quota and size defaults are [ASSUME].
- DEC-68 needs a manager countersignature.
- Legal and Sharia content is still pending.

**[SHARIA-CONFIRM] items.**
- Whether `issc_approval_ref` may be the same reference as `sharia_approval_ref`.
- Whether the Sharia certificate needs one reference per language.
- Whose ISSC approves Takaful item text (file 20 SH11).
- The ISSC-approved `contract_type` list.
- All disclosure wording and link labels, and the education text.

**[LEGAL-CONFIRM] items.** Whether a `CREDIT_REFERRAL` must link the KFS, and whether an in-app suggestion is "advertising" (file 20 row E03).

## Change log

- 2026-11-10: created (independent two-track validation).
- 2026-11-10 (contract 1.2.0): V-8 and V-9 marked CLOSED (design) in §1, §5, §6, §8 and §9; §10 addendum added with design, version decision, files, exact check results, remaining concerns and [SHARIA-CONFIRM] items.
