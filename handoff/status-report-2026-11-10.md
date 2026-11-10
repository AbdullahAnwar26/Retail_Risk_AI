# Status Report for the Prompt-Generator Agent

As of 2026-11-10. Project folder: `G:\Retail_Risk_AI`. Use this to write the next prompt for the working agent (Claude Code, Opus). Facts are labelled; nothing here is approved by the bank's legal team.

## 1. What the project is

A **Retail Banking AI Intelligence subsystem**: customer intelligence, transaction intelligence, segmentation, financial-needs detection, churn prediction, product recommendations and retention. It is one part of a larger banking platform that has no code yet. It runs **standalone** (own API, dashboard, synthetic data) or as a **plugin** to the future host platform, on **one shared core** with versioned contracts.

Hard rules (never relax):
- Never makes or emits a credit decision (approve, decline, limit, pricing, affordability, adverse action). Credit appears only as `CREDIT_REFERRAL`; every output carries `not_for_credit_decisioning: true`.
- Synthetic data only; no real bank systems.
- Under-18s: two-tier block list (file 14 §13.3); no commercial profiling or marketing to minors.
- No sensitive or proxy attributes for targeting; never infer religion, health, ethnicity.
- Laws are configuration (versioned policy packs), never hard-coded; every pack stays `PENDING_LEGAL_REVIEW` (enforced as BLOCK) until a named legal approver signs.
- LLM only explains or orchestrates; template-only by default.

## 2. Scope decision (user, 2026-11-10)

Two separate deployment tracks on the same core:
- **Track A, conventional bank:** New York City (US federal + NY State + NYC) and **London (UK)** in one deployment.
- **Track B, Islamic bank:** separate deployment, data, models and product catalogue. **Assumed UAE onshore (CBUAE-licensed)**, based on a UAE Islamic Bank credit manual the user shared; not yet confirmed.

The user delegated all other open questions to the agent ("answer on your own, I need the best results"). The agent must still never invent external facts (the bank's licence or charter, a named person).

## 3. What exists (all in `G:\Retail_Risk_AI`)

| Area | Location | Status |
|---|---|---|
| Planning pack | `planning/00`–`18` | Done, validated, corrected |
| Work log for the manager | `planning/00-work-log-and-methodology.md` | Kept current |
| Decision register | `planning/12-open-questions-and-decisions.md` (§H authoritative; DEC-01..DEC-68; 48 decided, 5 decided-default, 13 blocked-external, 2 superseded; DEC-68 purpose-bound customer directory, §H.7.4) | Updated 2026-11-10 |
| NYC legal matrix | `planning/13` | Done; re-verified against primary sources |
| Policy layer design | `planning/14` | Done |
| Integration contracts (prose) | `planning/15` | Done; normative YAML wins |
| Manager summary | `planning/18` | Updated for two tracks |
| UK/London legal matrix | `planning/19` (rows U01–U25; 9 verified, 8 partial, 8 unverified) | Done 2026-11-10 |
| Islamic bank (UAE) legal + Sharia matrix | `planning/20` (rows E01–E21, SH01–SH14; Sharia design constraints SDC-01..18; questions IQ1–IQ21) | Done 2026-11-10 |
| Two-track deployment design | `planning/21` (Track A cells `A-US` and `A-UK`, separate databases and regions; Track B separate bank-hosted deployment; Sharia overlay controls mapped to file 20 SDC-01..18; validation additions: cell isolation checklist §2.1, token scheme §2.2, profile integrity §3.1, portfolio disclosure control §10A) | Done 2026-11-10; independently validated |
| BMAD v6.12.4 (core + bmm) | `_bmad/`, `.claude/skills/` | Installed; reads `planning/` via `_bmad/custom/config.toml` |
| BMAD artefacts | `_bmad-output/planning-artifacts/` (brief, PRD FR-1..FR-82, UX incl. Arabic RTL, architecture spine AD-1..AD-27, epics 86 stories, readiness, README) | Updated for two tracks and contract 1.2.0; readiness CONCERNS, no Blocker, no open finding (V-8, V-9 closed as design 2026-11-10) |
| API contract | `contracts/openapi/retail-intelligence-v1.yaml` (v1.2.0, OpenAPI 3.1, 47 operations incl. the purpose-bound customer directory `GET /v1/customer-directory` (scope `ri.customers.list`, closed access purpose, narrowing filter, bounded pages, quotas, audit of every call; `listCustomers` deprecated, not implemented), the Sharia disclosure block on Track B suggestions (file 20 SDC-11), `ri.transactions.read`, aggregate-only portfolio summary with disclosure control (floor 20, rounding, complementary suppression, no differencing filters), Sharia approval call; Redocly lint 0/0) | Done |
| Events | `contracts/events/` (15 CloudEvents schemas, 16 validated examples) | Done |
| Data model | `contracts/data-model/erd.md` (67 entities incl. `message_template`; Sharia disclosure columns on `product`; access columns on `audit_record`) | Done |
| Ports | `contracts/ports.md` | Done |
| Role/scope matrix | `contracts/role-scope-matrix.md` and `.json` (0.3.0: 10 roles incl. `sharia_approver`, 31 scopes, 2 access purposes, 680 cells, 514 negative tests + 10 deprecated-operation tests) | Done; checks pass |
| Agent rules | `AGENTS.md` (imported by `CLAUDE.md`) | Updated for two tracks; `planning/` edits allowed with change logs |
| Safeguard | `.claude/settings.json` switches off `bmad-build`, `bmad-build-auto`, `bmad-agent-dev`, `bmad-qa-generate-e2e-tests` | Active |

No application code, database migrations or infrastructure exist.

## 4. Key decisions already made

- Stack (proposed, developer-decided): Python + FastAPI, PostgreSQL, React dashboard, OpenAPI contract-first, scikit-learn baselines; ML runs in SHADOW mode only until model governance is approved; churn/needs served by rules in v1.
- Policy layer: jurisdiction packs (federal / state / city / country) + institution overlay (Sharia for Track B); tighten-only, most restrictive wins; ILLUSTRATIVE pack state for the standalone demo only (watermarked, refused in plugin/production).
- Fairness: minimum group size 100.
- "Credit-building need" removed from v1.
- GATE-1, GATE-2 and GATE-3 batches: recorded as **DECIDED on developer-delegated authority** (manager countersignature recommended). GATE-2 version pins and the UI design system are still open. Starting code still needs the developer's explicit go-ahead.
- Two tracks: Track A one profile with US and UK data cells (no personal data crosses; only aggregates combine); Track B fully separate (data, models, keys, logs, identity), bank-hosted, no external LLM, Arabic + English. Track is a signed deployment profile; mismatch stops startup. AE adult age set to 21 [ASSUME].
- DEC-66: human review kept in Track A for offer suppression, retention allocation and vulnerability routing (UK Art 22A). DEC-67: Track B contact caps pending file 20 IQ4.
- Interim legal approver: a role, not a person (Head of Compliance; plus Sharia Compliance/ISSC for Track B). Packs remain blocked until a person is named.
- Interim integration owner: the developer.

## 5. Key legal/Sharia findings to respect

- US: SR 26-2 / OCC 2026-13 is supervisory guidance and excludes generative AI (a separate LLM governance track is a recommended control, not law). Reg B amendment effective 21 Jul 2026, under challenge.
- UK: new automated-decision rules (UK GDPR Art 22A–22D, from 5 Feb 2026) may cover offer suppression, retention-offer allocation and vulnerability routing, so keep human review there. London entity type (UK bank vs branch) changes which PRA rules apply. Recommendation cards may be financial promotions.
- Track B (UAE Islamic): customer data must stay in the UAE; no hosted or external LLM. Higher Sharia Authority resolutions are binding. Recommend only ISSC-approved Sharia-compliant products; "finance/profit", never "loan/interest", in Arabic and English; Takaful never called insurance; ISSC approves customer-facing templates; the system never gives a Sharia ruling. Telemarketing Regulation C 3/2026 may cap contacts (1 a day, 2 a week).

## 6. Blocked on people (cannot be decided by the agent)

1. Bank licence/charter per track (DEC-10): US/NY charter; UK bank vs branch (DEC-65); Track B onshore CBUAE bank vs Islamic window vs DIFC/ADGM (file 20 IQ1).
2. A **named** legal/compliance approver (DEC-11), plus Sharia approver for Track B.
3. Manager countersignature of the developer-ratified GATE-1/2/3 batches (recommended); GATE-2 version pins and UI design system.
4. Legal confirmations listed in files 13 §6, 19 and 20 (IQ1–IQ21).
5. Model-risk scope (DEC-19): ML stays SHADOW until resolved.

## 7. Where the work stands in the roadmap

- PH-1 discovery, PH-2 technology, PH-3 data and contracts: complete for both tracks; GATE-3 developer-ratified on delegated authority, manager countersignature recommended (interim integration owner: the developer).
- Nothing is running. Independent validation re-run (2026-11-10, `handoff/two-track-validation-report-2026-11-10.md`): earlier 11 findings confirmed fixed; 9 new findings, 7 fixed as design tightenings (file 12 §H.7.3); V-8 and V-9 closed in contract 1.2.0 (planning only; DEC-68, file 12 §H.7.4; validation report addendum). Readiness CONCERNS, no Blocker, no open finding; declared gates and [SHARIA-CONFIRM]/[LEGAL-CONFIRM] content items remain.
- Not started: PH-4 onward (synthetic data generator, transaction intelligence, segmentation, models, recommendations, orchestration, dashboard, plugin, security and testing). Code is still off.

## 8. Suggested next prompts (in order)

1. Done 2026-11-10: independent readiness re-check (validation report above). Done 2026-11-10: V-8 and V-9 closed in contract 1.2.0 (planning only). Next: the manager pack for countersignature (now also DEC-68) (GATE-1/2/3 incl. the DEC-01/02/12/63 working positions, licence questions per track, named legal and Sharia approvers).
2. Decide whether to start PH-4 code. If yes: re-enable the code skills by removing the four entries in `.claude/settings.json` and `CLAUDE.md` note, record the approval in file 12, then start with the **synthetic data generator** (both tracks, with edge cases: refunds, reversals, duplicates, pending, transfers, multi-currency), contract-first per the OpenAPI file.
3. Draft the illustrative policy packs per track (US/NY/NYC, GB, AE + Sharia overlay) as data, marked ILLUSTRATIVE and PENDING_LEGAL_REVIEW.

## 9. Working conventions for the next prompt

- Use Opus for every task and subagent; let an Opus agent decide anything within developer authority; record decisions in file 12 and the work log (file 00).
- Cite decisions as DEC-xx / BQ-x, legal rows as "file 13 row S12", "file 19 row U07", "file 20 row E05 / SH03".
- Keep `planning/` edits paired with change-log lines.
- Never fabricate legal facts; mark legal points [LEGAL-CONFIRM] and Sharia points [SHARIA-CONFIRM].

## Change log
- 2026-11-10 (independent two-track validation): rows for file 21, BMAD artefacts, contract (1.1.2) and ERD (66 entities; was 63) corrected; §7 GATE-3 wording "signed" -> "developer-ratified, manager countersignature recommended"; readiness result and open items V-8, V-9; §8 item 1 marked done.
- 2026-11-10 (contract 1.2.0, V-8/V-9): decision register DEC-68 and counts 48/5/13/2; BMAD, contract (1.2.0, 47 operations), events (16 examples), ERD (67 entities), role matrix (31 scopes, 514 negative tests) rows; §7 and §8 item 1 updated.
