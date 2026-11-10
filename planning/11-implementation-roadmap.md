# 11 - Implementation Roadmap

Labels: [DOC p.N] [REC] [ASSUME] [UNKNOWN] [MANAGER] [LEGAL-CONFIRM]. NYC scope update: initial market New York City (federal + NY State + NYC); legal-review gates added (section 7). **Two-track scope update (2026-11-10; file 21; file 12 DEC-50..DEC-64): Track A "Conventional" = NYC + London, Track B "Islamic" = Islamic bank, UAE onshore [ASSUME]; per-track phases and gates in section 7A. GATE-1, GATE-2 and GATE-3 ratification items are DECIDED (developer-delegated, 2026-11-10), manager countersignature recommended (file 12 §H.7.2); starting application code still needs the developer's explicit implementation instruction.** No calendar dates; sequencing is relative (PH-1 before PH-2, etc.). Effort sizing is not estimated here [UNKNOWN]. Work proceeds one phase at a time with an approval gate after each [MANAGER]. BMAD not yet installed; phases will be re-homed into BMAD structure after approval (file 17).

**ID scheme (this file is the source):** phases `PH-1`..`PH-12`, with NYC-scope phases `PH-2A`, `PH-3A`, `PH-4A`, `PH-8A`; gates `GATE-1`..`GATE-12`, `GATE-2A`, `GATE-3A`, `GATE-4A`, `GATE-8A`. Earlier drafts used `P1`/`G1`; those labels are retired because they collided with architecture principles (file 05) and policy-layer goals (file 14). Decisions are cited as `DEC-xx` (file 12 §A).

Note: the PDF's own plan (4 releases in ~3 months with 14-18 people, flagged high-risk, DOC p.45-46) belongs to the credit platform and is not inherited.

## 1. Phase table
| Phase | Name | Entry criteria | Deliverables | Exit criteria | Gate |
|---|---|---|---|---|---|
| PH-1 | Discovery and business requirements | Planning docs 00-15 drafted | Approved scope, definitions (churn, blue-collar, Gen Z; DEC-04), requirements baseline, shared-requirement register, question log | Manager approves scope and definitions; prd.md obtained or gap accepted | GATE-1 Scope approval |
| PH-2 | Technology compatibility and architecture | GATE-1 passed | Tech compatibility report, standalone/plugin architecture, decision records | Architecture approved; stack decision (conditional if host unknown; DEC-05) | GATE-2 Architecture approval |
| PH-3 | Database and integration contracts | GATE-2 passed | ERD incl. policy tables, contract v1 (file 15), adapter ports, jurisdiction/market reference-data model, draft role-to-scope matrix (file 08 SEC-19) | Contracts reviewed; contract tests defined | GATE-3 Contract review |
| PH-4 | Synthetic data | GATE-3 passed | Seeded generators, edge cases, catalogues, consent data, data dictionary, data-quality rules (file 07 §4.1; DEC-39), synthetic-data validation report (distribution sanity, edge-case coverage, label-leakage audit) | Reproducible dataset; edge-case coverage report; validation report reviewed; synthetic-limits statement | GATE-4 Data sign-off (light) |
| PH-5 | Transaction intelligence | Synthetic transactions available | Dedupe, transfers, refunds, reversals, FX, categorisation, income/expense/savings | All fixture tests pass; categorisation metrics on synthetic labels | GATE-5 Demo of slice |
| PH-6 | Customer intelligence and segmentation | PH-5 aggregates | Profiles, features, multi-membership segments, minors guard (two-tier block list, file 14 §13.3) | Minors guard test passes; overlap cases correct; fairness view available (method per DEC-21) | GATE-6 Compliance preview |
| PH-7 | Prediction models | PH-6 features | Needs rules, churn baseline + ML, evaluation, model registry with approval status | ML only if it beats baseline on time-split; leakage checks; explanations stable | GATE-7 Model review |
| PH-8 | Product recommendation | PH-7 outputs, catalogues | Gated recommender, reason codes, retention suggestions with outcome capture, human review queue (DEC-37) | Every output grounded in catalogue; all gates tested; flagged items withheld until reviewed | GATE-8 Recommendation review (incl. legal/compliance placeholders) |
| PH-9 | AI orchestration | PH-5-PH-8 modules callable | Tool layer, grounded explanations, assistant, refusals | Safety/red-team set passes; number checks; template fallback | GATE-9 AI safety review |
| PH-10 | Standalone dashboard and workflows | PH-9, PH-8A | Dashboard (accessibility target, DEC-41), audit view, demo pack | Scripted demo runs from clean start | GATE-10 Manager demo |
| PH-11 | Plugin integration | PH-3 contracts, PH-8 outputs (can overlap PH-9-PH-10) | Published contract v1, reference adapters, mock host | Parity with standalone; host identity/consent via ports | GATE-11 Integration review (needs host info if available) |
| PH-12 | Security, compliance, testing, evaluation | Runs throughout; final after PH-11 | Security baseline, test suite, compliance open-items pack, technical evaluation, backup/restore design and restore test (file 08 SEC-20; DEC-36), operator runbook (pack activation/rollback, signing-key rotation, fail-closed recovery), reason-code library (versioned, owner COMP; DEC-40), user guide | Items to be confirmed by legal/compliance listed with owners; evaluation report issued | GATE-12 Release-candidate / pilot decision |

## 2. Sequencing and parallelism
- Strict: PH-1 -> PH-2 -> PH-3 -> PH-4 -> PH-5 -> PH-6 -> PH-7 -> PH-8.
- Overlap allowed: PH-9 tool layer can start once PH-5/PH-6 APIs exist; PH-10 UI scaffolding from PH-6; PH-11 contract work from PH-3 with parity tests after PH-8; PH-12 security/test work continuous.
- [REC] Build a thin vertical slice after PH-4 (one customer through every module with stubs) to expose contract problems early, before broadening.

## 3. Approval gates (who decides)
| Gate | Decision-maker | Decision |
|---|---|---|
| GATE-1 | Manager | Scope, boundaries, definitions |
| GATE-2 | Manager + team tech leads (if any) | Architecture and stack |
| GATE-3 | Manager / integration owner (2026-11-10: the developer, interim, until a host platform owner exists; DEC-63) | Contract v1 |
| GATE-3A | Manager + compliance engineer (legal approver informed) | Policy layer core: fail-safe tests, no jurisdiction literal in core |
| GATE-4 | Manager (data owner reviews validation report) | Synthetic data and its validation report |
| GATE-5 | Manager | Transaction slice demo |
| GATE-6, GATE-8, GATE-12 | Legal/compliance (advisory) | Minors guard, gates and human review, open legal items |
| GATE-2A, GATE-4A, GATE-8A | Legal/compliance approver (named by manager, DEC-11) [MANAGER]. 2026-11-10 (DEC-63) interim roles per track: Track A Head of Compliance (role); Track B Head of Compliance + Sharia Compliance function / ISSC (roles, `sharia_approver`). No pack is APPROVED or ACTIVE until a named person is recorded | Applicability decisions, pack approval (NYC, GB, AE; Track B also Sharia approval), pack activation [LEGAL-CONFIRM] [SHARIA-CONFIRM] |
| GATE-7 | Model-risk contact [UNKNOWN] | Model approach (DEC-19) |
| GATE-9 | Manager + security (model-risk contact consulted for the LLM governance track, DEC-19) | AI safety |
| GATE-10 | Manager | Demo acceptance |
| GATE-11 | Platform team [UNKNOWN] | Host compatibility |

## 4. Key dependencies
| Dependency | Needed by | Status |
|---|---|---|
| prd.md (FR-39, FR-50/51, FR-79..86) | PH-1 | [MANAGER] (file 12 MS1) |
| Business definitions (churn, blue-collar, Gen Z) | PH-1/PH-4 | [MANAGER] DEC-04 |
| Host stack plan/contracts | PH-2/PH-3/PH-11 | [UNKNOWN]; none exists |
| Ownership of next-best offers, early-warning, readiness record | PH-1/PH-3/PH-8 | [MANAGER] DEC-01, DEC-02 |
| Legal/compliance feedback on minors handling, consent for analytics/marketing, fairness requirements, retention periods | PH-3, PH-6, PH-8, PH-12 | To be confirmed by legal/compliance (DEC-18, DEC-20, DEC-21, DEC-22) |
| Approved LLM runtime | PH-9 | [UNKNOWN] DEC-06 |
| Named legal approver | PH-3A, GATE-2A | [MANAGER] DEC-11 (interim role per track, DEC-63) |
| Named Sharia approver (Track B) | PH-4A-B, GATE-4A-B | [MANAGER] DEC-63 [SHARIA-CONFIRM] |
| UK applicability matrix (file 19) and UAE + Sharia matrix (file 20), both written 2026-11-10 | PH-2A-UK, PH-2A-AE | [LEGAL-CONFIRM] |
| UK operating entity type and UK data controller (DEC-65, BLOCKED-EXTERNAL; file 19 UQ1) | GB pack scope (PH-2A-UK, PH-4A-UK) | [MANAGER] [LEGAL-CONFIRM] |

## 5. Risks
| Risk | Likelihood/Impact [ASSUME] | Mitigation |
|---|---|---|
| Scope drift into credit decisions | M/H | Boundary doc, refusal tests, `not_for_credit_decisioning` output label (DEC-42), review at GATE-1 |
| Host stack emerges incompatible later | H/M | Ports/adapters, stack-neutral contracts, conditional recommendations |
| Synthetic data produces misleading model results | H/M | State limits; validation report at GATE-4; evaluate with time split; no performance claims for real banks |
| Definition gaps (churn, blue-collar) stall models | M/H | Resolve at GATE-1; provisional definitions marked [ASSUME] |
| Minors safeguard breached via side effects (logs, features, training) | L/H | Early guard at ingest; two-tier block list; guard tests; audit |
| Sensitive-trait inference via merchant categories/proxies | M/H | Excluded category list, proxy review, fairness view |
| LLM hallucination in explanations | M/H | Tool-grounded, number check, template fallback |
| Over-engineering (microservices, MLOps) | M/M | Modular monolith default; justify any split |
| Overlap/duplication with credit early-warning | M/M | Decide ownership at GATE-1 (DEC-02); consume rather than duplicate |
| Unrealistic expectations from the PDF's 3-month plan | M/M | Communicate separate plan; relative phases only |
| Regulatory changes not captured | M/M | Policy-as-data, versioned configs, legal confirmation, review_due expiry fail-safe |
| Policy pack built before applicability decided | M/H | GATE-2A must pass before PH-4A; packs stay PENDING_LEGAL_REVIEW and enforced restrictively |
| No named legal approver or charter unknown | H/H | Escalate at GATE-1/GATE-2A; no activation without approver |
| Jurisdiction literals leak into core | M/M | CI lint, add-a-jurisdiction rehearsal (story S15.5, file 10) |
| DSR erasure breaks hash-chained audit/decision log | M/M | Tokenised subject refs + crypto-shredding (file 07 §8; DEC-35) |
| Two tracks and a second Track A cell raise effort (packs, catalogues, Arabic/RTL) (2026-11-10) | H/M | Build the core once; Track A first, Track B as configuration; Track B items need no core change (DEC-51) |
| Data or models leak across cells or tracks (2026-11-10) | L/H | Separate databases per cell, separate Track B deployment, bootstrap track-tag checks, isolation tests (DEC-52, DEC-53, DEC-57) |

## 6. Post-approval working mode
- After GATE-1 approval, proceed phase by phase; each phase starts with a small plan, ends with its gate.
- Keep tasks small and verifiable; do not re-audit completed work without a specific reason.
- Install/inspect BMAD only after approval; re-home epics (10) and requirements into its structure.

## 7. NYC scope update: new and changed phases (no calendar dates)
| Phase | Name | Entry | Deliverables | Exit | Gate |
|---|---|---|---|---|---|
| PH-2A | Regulatory applicability assessment (E13) | GATE-1 passed; institution profile requested | Charter/status/product/activity profile (DEC-10), row-by-row decisions on file 13 (rows F01-F18, S01-S17, C01-C07), owners for top legal questions (file 12 §G.2) | Legal approver records decision per row needed for the NYC pack; undecided rows stay PENDING_LEGAL_REVIEW | GATE-2A Applicability gate [LEGAL-CONFIRM] |
| PH-3 (changed) | Database and integration contracts | GATE-2 passed | Adds policy tables, contract v1 with capabilities manifest and policy decision query (E19) | Contracts reviewed | GATE-3 |
| PH-3A | Jurisdiction policy layer core and authoring workflow (E14, E15) | GATE-3 passed (can overlap PH-4) | Hierarchy, policy model, control catalogue, PDP, PEP stubs, fail-safe rules, maker-checker workflow | Fail-safe tests pass; no jurisdiction literal in core | GATE-3A Policy layer review |
| PH-4A | NYC policy pack authoring (E16) | GATE-2A and GATE-3A passed | Illustrative L0, US, US-NY, US-NY-NYC packs citing file 13 row IDs; exclusion and proxy lists (DEC-23) | Packs built and signed, status per legal; unapproved remain pending | GATE-4A Pack legal review |
| PH-6 (changed) | Adds minors safeguards (E18) | as before | Protected journey, two-tier block list (file 14 §13.3); approval record can relax only the non-commercial tier | Guard tests pass | GATE-6 |
| PH-8A | Policy simulator, golden tests, pack approval (E17) | PH-8, PH-4A | Simulator, golden cases, invariant tests, legal-pending demo case | All pass in CI; approval records stored; activation rules enforced | GATE-8A Pack approval and activation [LEGAL-CONFIRM] |
| PH-10 (changed) | Demo shows jurisdiction/policy decisions and pending-review fail-safe (file 09 §7) | PH-9, PH-8A | as PH-10 | as PH-10 | GATE-10 |

Sequencing: PH-1 -> PH-2A (legal, may run alongside PH-2 and PH-3) -> GATE-2A. PH-4A must not start for a given law until its row is decided at GATE-2A. Policy layer core (PH-3A) can be built with stub packs before approvals. PH-5-PH-8 modules call PEPs from the start (stubbed to restrictive defaults).
Legal-review gates summary: GATE-2A (applicability before pack), GATE-3A (policy layer core, compliance engineer), GATE-4A (pack content), GATE-8A (activation); GATE-6/GATE-8/GATE-12 remain as before. Without GATE-8A approval the demo runs with packs in PENDING_LEGAL_REVIEW state, enforced restrictively.
Deferred phases (not scheduled): other US states and ~~UK or~~ other international packs via the add-a-jurisdiction procedure (file 14 §12) (2026-11-10: UK London and UAE are in scope, DEC-50; DEC-13 superseded).

## 7A. Two-track scope: per-track phases and gates (2026-11-10; file 21)
| Phase | Name | Entry | Deliverables | Exit | Gate |
|---|---|---|---|---|---|
| PH-2A-US | Applicability assessment, US/NY/NYC (= PH-2A above) | GATE-1 | file 13 rows decided | as PH-2A | GATE-2A-US |
| PH-2A-UK | Applicability assessment, UK / England / London | GATE-1; file 19 (written 2026-11-10) | Row decisions for `GB`, `GB-ENG`, `GB-ENG-LDN`; Track A UK bank authorisation profile (DEC-10 per track) | Undecided rows stay PENDING_LEGAL_REVIEW | GATE-2A-UK [LEGAL-CONFIRM] |
| PH-2A-AE | Applicability assessment, UAE onshore + Sharia governance | GATE-1; file 20 (written 2026-11-10) | Row decisions for `AE`, `AE-<emirate>`; Track B licence and Sharia governance profile; Sharia lists for the overlay | As above | GATE-2A-AE [LEGAL-CONFIRM] [SHARIA-CONFIRM] |
| PH-3 (changed) | Contracts | GATE-2 | Adds read-only `deployment_profile` in capabilities, profile/track/cell on decision records, customer list, stored-transactions read, aggregate portfolio (DEC-61, DEC-62) | Contracts reviewed (integration owner: developer, interim) | GATE-3 |
| PH-3A (changed) | Policy layer core | GATE-3 | Adds deployment-profile loader, data cells, institution-overlay layer (restrictive-only invariant), 4 new controls, Sharia approval state, `sharia_approver` role (DEC-51, DEC-54, DEC-55) | Monotonicity test with overlay passes; no track literal in core | GATE-3A |
| PH-4 (changed) | Synthetic data | GATE-3 | Seeds and outputs per cell (`A-US`, `A-UK`, `B-AE`); three synthetic catalogues (DEC-56) | Validation report per cell | GATE-4 |
| PH-4A-US / PH-4A-UK | Track A pack authoring | GATE-2A-US / GATE-2A-UK, GATE-3A | Illustrative US, US-NY, US-NY-NYC / GB, GB-ENG, GB-ENG-LDN packs | Pending until named approver | GATE-4A-US / GATE-4A-UK |
| PH-4A-AE | Track B pack and overlay authoring | GATE-2A-AE, GATE-3A | Illustrative AE packs + `PACK-INST-ISLAMIC` (terminology, catalogue eligibility, prohibited categories) | Pending legal **and** Sharia approval | GATE-4A-AE [SHARIA-CONFIRM] |
| PH-7 (changed) | Models | PH-6 | Per-track registries; per-cell evaluation (DEC-57) | as PH-7 | GATE-7 |
| PH-9/PH-10 (changed) | Explanations and dashboard | as before | Arabic + English templates (Track B), RTL, per-cell currency (DEC-59); two demos (DEC-64) | Both scripted demos run from clean start | GATE-10 |
| PH-8A (changed) | Pack approval/activation | per track | Track B activation needs legal + Sharia approval | as PH-8A | GATE-8A-US/UK/AE |

Sequencing [REC]: build the core once; deliver Track A (`A-US` first, then `A-UK` as a second cell of the same profile), then Track B as configuration (profile, overlay, catalogue, templates, locale). PH-2A-UK and PH-2A-AE run in parallel with build and do not block market-neutral work. GATE-1, GATE-2 and GATE-3 ratification items are developer-ratified (file 12 §H.7.2; DEC-63 covers only the interim approver roles and integration owner); manager countersignature recommended.

## Change log (NYC scope update)
- Added section 7: phases P2A, P3A, P4A, P8A and changed P3, P6, P10; new legal-review gates G2A, G4A, G8A.
- Replaced US/UK config model in P3; added gate decision-makers and three risks.
- Other states and UK deferred; no calendar dates added.

## Change log (validation pass 2026-11-09)
- D-06, R-03, R-04: phases renamed P1..P12 -> PH-1..PH-12 (PH-2A, PH-3A, PH-4A, PH-8A) and gates G1.. -> GATE-1.. (GATE-2A etc.); this file is the source for the ID scheme. Decisions cited as DEC-xx (file 12).
- D-25, R-01, R-02: §3 adds decision-makers for GATE-3A, GATE-4, GATE-5, GATE-9; GATE-3A added to the legal-review gates summary. PH-1 entry "Planning docs 00-15"; PH-10 entry "PH-9, PH-8A".
- D-40 (G-10): PH-12 deliverables add operator runbook, reason-code library (owner COMP), user guide. G-3 (D-31): backup/restore design and restore test in PH-12. G-8 (D-39): synthetic-data validation report reviewed at GATE-4. G-6 (D-34): data-quality rules in PH-4. G-1 (D-29): human review queue in PH-8. G-7 (D-35): accessibility target in PH-10. D-09: retention outcome capture in PH-8.
- D-01: PH-6 minors guard refers to the two-tier block list (file 14 §13.3). R-14: "story S15.5, file 10". New risk: DSR erasure vs hash-chained logs (G-4). Sweep 11:12 "market reference-data" kept (defined concept).

## Change log (two-track scope 2026-11-10)
- Header two-track note; §3 GATE-3 integration owner = developer (interim, DEC-63); GATE-2A/4A/8A interim approver roles per track incl. Sharia approver; §4 new dependencies (Sharia approver, files 19 and 20 in progress); §5 two new risks; §7 deferred line updated (DEC-13 superseded); new §7A per-track phases and gates (PH-2A-UK, PH-2A-AE, PH-4A-UK, PH-4A-AE; changed PH-3, PH-3A, PH-4, PH-7, PH-9/10, PH-8A) and sequencing (core once, Track A first, Track B as configuration). GATE-1/2/3 noted as developer-ratified (manager countersignature recommended).
- Files 19/20 recorded as written 2026-11-10; dependency row for DEC-65 (UK entity type) added.
- 2026-11-10 (independent two-track validation): §4 sequencing note cites file 12 §H.7.2 for the GATE-1/2/3 developer ratification (DEC-63 was cited in error; it covers interim approver roles and the integration owner). GATE-1/2/3 remain developer-ratified, manager countersignature recommended; not manager-approved.
