# Retail Banking AI Subsystem: Planning Summary for Approval

Prepared 2026-11-09 for the project manager. Planning only: no application code, no infrastructure, BMAD not installed. Legal content is technical research, not legal advice; applicability must be confirmed by the bank's legal and compliance teams.

## 1. Where we are

The planning pack (files 01-15) is drafted, independently reviewed, corrected and ready for approval. Scope: a Retail Banking AI subsystem (customer and transaction intelligence, segmentation, financial needs, churn, product recommendation, retention), now in two tracks (section 1B: NYC + London conventional; Islamic bank), running standalone or as a plugin on one shared core. It never approves or declines credit.

## 1A. Approval pass outcome (2026-11-09)

The developer, as owner of the Retail Banking AI subsystem, decided every open item. Full table: file 12 §H.

| Status | DEC (43) | BQ (11) | Total |
|---|---|---|---|
| DECIDED: developer authority, binding now | 14 | 1 | 15 |
| DECIDED-DEFAULT: work proceeds on this default; you ratify at the gate shown | 17 | 10 | 27 |
| BLOCKED-EXTERNAL: needs bank, host or legal input; build uses a safe assumption | 12 | 0 | 12 |

**Blocked items:** DEC-10 charter (N1), DEC-11 named approver (N2), DEC-07 audit scope, DEC-18 consent purposes, DEC-19 model-risk scope, DEC-20 retention periods, DEC-21 fairness proxies, DEC-22 minors (NY Child Data Protection Act), DEC-23 exclusion/proxy lists, DEC-25 jurisdiction attributes, DEC-30 host identity, DEC-33 host events. Until they are resolved: synthetic data only, all policy packs PENDING_LEGAL_REVIEW and enforced as BLOCK, no pack activation, no hosted LLM. None of them blocks the next phase. The latest gates are GATE-2A (N1), GATE-3A (N2), and GATE-4A to GATE-12 for the rest.

**For your ratification:** at GATE-1, the working definitions of churn (90-day window: closure, salary diversion or dormancy), blue-collar (consented self-declared occupation only; proxy risk flagged), Gen Z 18-29 (config), the needs taxonomy, retention outcomes and demo KPIs, plus ownership (DEC-01, DEC-02), adults-only NYC MVP and UK deferral. **Update 2026-11-10:** UK deferral is superseded by the two-track scope, and these items are now decided on the developer's delegation with your countersignature recommended (section 1B). At GATE-2, the stack. At GATE-3, contract defaults. Each default is config or a document, so a change is cheap.

**Verdict: GO, with conditions,** to BMAD setup (v6.12.4, backup and trial install first), technology compatibility (PH-2) and database/contracts design (PH-3). GATE-1 counts as provisionally passed, not formally approved. Conditions: synthetic data only; no pack activation; the charter request goes to legal/compliance now; the GATE-1 defaults are ratified before any application code is written.

## 1B. Two-track scope (2026-11-10)

**What changed.** The developer decided the market scope on 2026-11-10: *"target NYC, London and Islamic bank, separately: NYC and London in one, and on the other side the Islamic bank."* This resolves DEC-50 (target market) and replaces the NYC-only scope and the earlier "UK deferred" default (DEC-13). Design: file 21.

- **Track A, Conventional:** New York City (US federal, NY State, NYC) and London (UK, England, London) in one product. US and UK customer data are kept in separate databases in their own regions, so no personal data crosses between the US and the UK; only anonymous totals can be combined.
- **Track B, Islamic bank:** assumed to be in the UAE (onshore) until you confirm. A separate deployment on the bank's own systems, with its own database, models, catalogue and keys. It never shares data or models with Track A. An extra "Sharia layer" can only make rules stricter: only Sharia-approved products are suggested, wording uses "finance" and "profit" rather than "loan" and "interest", excluded merchant categories are never promoted or used to profile anyone, and nothing customer-facing goes out without Sharia sign-off. Arabic and English.
- **Unchanged:** one shared core and one contract for both tracks; no credit decisions (credit only as "refer for assessment"); under-18 protections; synthetic data only; every policy pack pending legal review and blocked until approved.

**Decided on the developer's delegation** (file 12 §H.7; your countersignature is recommended but planning does not wait for it): the GATE-1 batch (definitions, ownership, adults-only, demo posture), the GATE-2 batch (stack, deployment profiles) and the GATE-3 batch (no live credit feed, approval-workflow owner, contract defaults, role matrix), plus 14 new design decisions DEC-51 to DEC-64 (deployment profiles, data separation, Sharia layer and approval, synthetic catalogues per track, separate models per track, LLM on the bank's own systems only for Track B, languages and currency, age threshold per country, contract additions, the three open contract gaps, interim approver roles, two demos). Application code still needs an explicit go-ahead to start.

**Still needs you (or the bank):**
1. Countersign the delegated decisions above.
2. Name the legal/compliance approver and a backup for each track, and a named Sharia approver for Track B. Until then only roles exist (Head of Compliance; Sharia Compliance / Sharia committee) and no policy pack can be approved or switched on.
3. Bank facts per track (DEC-10): for Track A the bank's US charter and UK authorisation; for Track B the bank's identity, UAE licence and emirate, and its Sharia governance set-up.
4. Legal and Sharia review: the UK matrix (file 19) and the UAE/Sharia matrix (file 20) were written on 2026-11-10; their content must be confirmed by the bank's legal, compliance and Sharia teams before any UK or UAE pack is approved.
5. UK operating entity (DEC-65, file 19 UQ1): is the London business a UK-incorporated bank, a UK branch of the NYC bank, or something else, and which entity is the UK data controller? Until known, every UK pack stays blocked.
6. Two restrictive interim rules for legal to confirm: Track A keeps a person in the loop for offer suppression, retention-offer allocation and vulnerability routing (DEC-66, file 19 U02/UQ4); Track B treats in-app and push messages as telemarketing with contact caps (DEC-67, file 20 E08/IQ4).

## 2. How it was validated

| Check | Result |
|---|---|
| Full review of files 00-15 | 44 defects found (2 High, 15 Medium, 27 Low) and 10 missing requirements; all addressed (exceptions in section 4) |
| Scope check | All nine modules are retail intelligence; no credit scoring, grading, limits or adverse action in data, contracts, MVP or epics. One leak (affordability) removed |
| Shared core | Standalone and plugin use the same core in every document |
| NYC-first sweep | Every US/UK/market reference classified; 17 accidental leftovers corrected; history and future expansion kept |
| Legal re-check against primary sources | 2 errors corrected; matrix now 27 verified, 6 partial, 9 unverified rows |
| Diagrams | 13 of 13 Mermaid diagrams rendered successfully (layout not visually inspected) |
| Cross-references | One ID scheme across files; single decision register (file 12) |

## 3. Main corrections

- **Under-18s:** one two-tier block list now used everywhere; no profiling, prediction or model training on minors' data without recorded legal approval.
- **Affordability:** removed as an input or gate of ours; it belongs to the credit platform.
- **Credit-use restriction:** every output carries a "not for credit decisioning" label; host terms to prohibit credit use (wording needs legal).
- **Legal wording:** SR 26-2 / OCC 2026-13 described as supervisory guidance; a separate LLM governance track is a recommended control, not a legal requirement. Reg B amendment (91 FR 21620, effective 21 Jul 2026) under court challenge. NYC Human Rights Law 8-107(5)(d) covers housing/commercial-space financing only; other sections' applicability is unresolved. TCPA citation corrected to 91 FR 67214.
- **Added requirements:** human review workflow, role permissions, backup/restore, data-quality rules, fairness testing method, erasure despite tamper-evident logs, retention outcome tracking, accessibility.
- **Housekeeping:** decision IDs unified (DEC-01 to DEC-43); phases, gates and other ID schemes de-duplicated; safety-check naming clash resolved.

## 4. Remaining inconsistencies and limits

- File 13's legal question numbers (Q1-Q24) were kept, so "Q" labels are cited as "file 13 §6 Qn" to avoid confusion.
- File 13 row IDs (S01-S17) and the PDF's income segments (S0-S5) look similar to story IDs (S1.1); kept and cited with their file.
- Unverified legal points: Reg B court docket, full FCC rule text, parts of the NY Child Data Protection Act, NYC law preemption for a national bank, whether the NYCHRL text used is fully current.
- Research PDF issues stand: garbled tables in extraction, page-order repeat, figures that disagree (15 vs 12 steps, $150 vs $45, GBP750 vs GBP1,000).
- prd.md (FR-39, FR-50/51, FR-79..86) and the presentation were never provided.
- Rendered diagram layouts and the final edited files have not been read line by line by a person.

## 5. Decisions needed from you (N1-N9)

| # | Decision | Recommendation | Needed |
|---|---|---|---|
| N1 | Confirm the bank's charter, regulator, products and channels | Obtain from legal/compliance; all legal rows stay pending until then | Before legal applicability gate |
| N2 | Name the legal/compliance policy approver and a backup | Name a person or role. **2026-11-10:** interim roles set per track (DEC-63); a named person is still needed, plus a named Sharia approver for Track B | Before policy layer phase ends |
| N3 | Owner of policy approval and release | Compliance owns; operations activates with sign-off | Before policy layer phase |
| N4 | ~~Defer UK and other countries~~ | **Superseded 2026-11-10:** London in Track A; Islamic bank in Track B (DEC-50, section 1B) | Done |
| N5 | Replace fixed US/UK installs with policy-based deployment scope | Confirm | Tech phase |
| N6 | Behaviour after a policy's review date | Restrictive default, no grace period | Policy layer phase |
| N7 | Demo while policies are illustrative | Demo shows "pending legal review" openly | Before demo |
| N8 | Synthetic jurisdictions in tests | Approve as planned | Data phase |
| N9 | Host platform may tighten our rules, never loosen | Approve | Contracts phase |

The full register (43 decisions) is in file 12.

## 6. Open business and legal questions

Business (file 12): churn definition; blue-collar definition using lawful signals; Gen Z band 18-29 (ours) vs 18-25 (credit tag); who owns next-best offers, early-warning signals and the under-18 readiness record; retention outcome values; human review role and SLA; whether credit-platform signals are in the MVP; demo KPIs.

Legal (file 13 §6, top five): charter and NY licence status; NY Child Data Protection Act for 16-17 year olds; whether recommending or hiding credit products counts as discouragement or lending under Reg B and NY/NYC law; data use and service-provider status of any LLM or cloud host; model governance framework for ML and for the LLM.

## 7. Approval checklist

- [ ] Scope: retail intelligence only, two tracks (NYC + London conventional; Islamic bank, UAE assumed), credit decisions excluded (files 02, 06, 21) (decided by the developer 2026-11-10; countersignature recommended)
- [x] Architecture: one core, standalone and plugin adapters, versioned contracts (files 04, 05, 15) (developer decision, file 12 §H)
- [ ] Policy layer: US / NY / NYC and UK / England / London packs, UAE packs plus the Sharia layer for Track B, fail-safe default, legal and Sharia sign-off workflow (files 14, 21) (fail-safe rules and workflow owner decided; named approvers wait for you, DEC-11, DEC-63)
- [x] Data design: synthetic data, decision log, retention by policy (file 07) (developer decision; retention values wait for legal, DEC-20)
- [ ] Security, privacy and safeguards: minors, sensitive attributes, no credit use (file 08)
- [ ] MVP: both tracks, adults only (age threshold per country), rules and baseline ML, template explanations by default, one demo per track (files 09, 21) (decided on delegation 2026-11-10; countersignature recommended)
- [x] Roadmap and gates; epics and 91 stories (files 10, 11) (developer decision)
- [ ] Technology stack proposal (file 03), with platform-wide items noted (subsystem proceeds; platform-wide items ratify at GATE-2)
- [ ] Decisions N1-N9 (section 5) (N3, N5-N9 decided; N4 superseded; N1, N2 named persons and bank facts blocked on you/legal)
- [x] BMAD installation (section 8) (developer decision: v6.12.4, backup and trial install first, build workflow off until GATE-1 ratified)

## 8. BMAD recommendation

Install later, after your approval. Use the stable version v6.12.4 (MIT licence, Node 20.12+), not the v7 preview. Back up the workspace, trial the install in a copy, and compare which files it adds or changes, because the official docs do not fully state this. Then install only the Claude Code integration and the core method module. Load files 01-15 as read-only input and run the BMAD analyst, product, architecture and epics steps, with a written sign-off after each. Block its code-building workflow until implementation is approved. Details: file 17.

## 9. Files

Added 2026-11-10: 21 (two-track deployment design); 19 (UK/London regulatory matrix) and 20 (Islamic bank UAE regulatory and Sharia matrix) were written on 2026-11-10.

New in this pass: 16 (review report), 16b (diagram and link validation), 17 (BMAD recommendation), 18 (this summary). Corrected: 00-15. All in `planning/`.

## Change log
- 2026-11-09 approval pass: added section 1A (counts by status, blocked items, conditional GO). Ticked checklist items decided by the developer (architecture, data design, roadmap, BMAD); items needing you stay unticked. Details in file 12 §H.

## Change log (two-track scope 2026-11-10)
- Added section 1B (two-track scope decision, delegated decisions, what still needs the manager). Updated section 1 scope line, section 1A ratification note, N2 and N4 rows, checklist lines for scope, policy layer, MVP and decisions, and section 9 file list. See file 12 §H.7 and file 21.
- Files 19 and 20 now recorded as written 2026-11-10; §1B adds DEC-65 (UK entity type, blocked on the bank) and DEC-66/DEC-67 (interim human-review and contact-cap rules).

