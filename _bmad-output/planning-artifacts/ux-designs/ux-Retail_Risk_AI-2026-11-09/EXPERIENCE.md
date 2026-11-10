---
title: EXPERIENCE.md - Retail Intelligence Staff Dashboard
status: final
created: 2026-11-09
updated: 2026-11-10
sources: [planning/21 §8 §10 §11, prds/prd-Retail_Risk_AI-2026-11-09/prd.md, planning/04 §5.1, planning/07 §2.1, planning/09 §5 §7, planning/10 E10 E17, planning/14 §10 §13, planning/15 §2 §4 §5.1]
design_ref: ./DESIGN.md
---

# EXPERIENCE.md - Retail Intelligence Staff Dashboard

This document is the behavioural contract. It wins over any future mock. Visual tokens are open (see `DESIGN.md`). Journeys mirror the PRD UJ IDs.

## Foundation
- **Form factor:** desktop web, staff only. There is no customer-facing or mobile surface in the MVP. *Source: planning/02 §2; planning/09 §1.*
- **Stack:** React + TypeScript + Vite. The dashboard is just another client of the public `/v1` API, with no private back-door endpoints (DEC-05 DECIDED-DEFAULT, GATE-2). *Source: planning/04 §4, §5.1.*
- **UI system:** not decided (UX-OQ-3).
- **Mode:** the dashboard exists in standalone mode only; in plugin mode the host owns the UI. *Source: planning/06 §3.*
- **Auth:** local demo issuer with demo roles. Screen visibility follows the role-to-scope matrix `contracts/role-scope-matrix.md` / `.json` (screens S00-S12, actions A01-A12; DEC-38 draft for GATE-3), which supersedes planning/15 §5.1.

## Information Architecture
Screens are taken from planning/08 SEC-19 and planning/15 §5.1. Every screen is deny-by-default per role.

| Screen | Purpose | Roles (R read, W act) | Journeys | PRD FRs |
|---|---|---|---|---|
| Customer list / picker | Choose a synthetic customer from the purpose-bound `GET /v1/customer-directory` (contract 1.2.0, DEC-68; tokens and coarse codes only: token, jurisdiction code, age-status class, data-quality flag; no names, DEC-62). The access purpose is the user's registered one (relationship staff `CUSTOMER_SERVICING`, analyst `INSIGHT_REVIEW`), shown on the screen; the user enters a token or picks a narrowing filter (segment or data-quality status) before any list loads; at most 25 rows a page and 100 per query, with a "narrow your filter" message when capped; quota and purpose denials show their error state. Test-jurisdiction selector limited to the serving cell's in-scope codes plus out-of-scope (`US-CA`), cross-cell, cross-track and unknown | analyst, relationship_staff (the auditor has no directory access; it opens a customer from a token in an audit record) | UJ-1, UJ-2, UJ-3 | FR-48, FR-53 |
| Customer view | Transactions, profile, segments, needs, churn, recommendations, retention; each output with `decision_id`, jurisdiction context and versions | analyst, relationship_staff, auditor R (the auditor has no `ri.recommendations.generate`, so its recommendation section shows DENIED `FORBIDDEN_SCOPE`; it inspects recommendations through the audit and decision log; matrix C-RS-1). Picker reads `GET /v1/customer-directory` (tokens only, `ri.customers.list`, purpose-bound, DEC-68); the auditor reaches this view only with a token taken from an audit or decision record; the transactions panel reads `GET /v1/customers/{customer_token}/transactions` (enriched view only, `ri.transactions.read`, purpose INSIGHT). The auditor holds no `ri.transactions.read`, so its transactions panel shows DENIED `FORBIDDEN_SCOPE` (DEC-62; matrix G-RS-2, G-RS-3 decided) | UJ-1, UJ-3, UJ-8, UJ-9, UJ-10 | FR-48, FR-80 |
| Sharia disclosure on suggestion cards (Track B only) | Each product suggestion and credit referral card shows Sharia basis, certificate and KFS links in the active language (Arabic or English), from `sharia_disclosure` (contract 1.2.0, planning/20 SDC-11); in the demo the card adds "illustrative - not a Sharia ruling" and shows no approval reference; a card without a complete disclosure is never shown. Track A cards have no such block. Link labels and wording come from approved templates [SHARIA-CONFIRM] | analyst, relationship_staff | UJ-10 | FR-72, FR-76 |
| Retention outcome capture (within customer view) | Record NOT_ACTIONED / CONTACTED / RETAINED / CLOSED / UNKNOWN with a fixed reason | relationship_staff W | UJ-1 | FR-25 |
| Human review queue | QUEUED, IN_REVIEW, CLEARED_TO_SHOW, REJECTED, ESCALATED; SLA indicator | relationship_staff R/W (`ReviewerRole`); analyst, auditor R | UJ-5 | FR-24 |
| Audit and decision log | Audit trail, gate results, consent state, recommendation replay | auditor R, policy_approver R | UJ-2, UJ-8 | FR-49, FR-31 |
| Policy pack viewer | Packs, versions, `applicability_status`, `review_due`; PENDING_LEGAL_REVIEW shown openly | analyst, model_admin, auditor, policy roles R; `sharia_approver` R (ISLAMIC deployments only) | UJ-2, UJ-6, UJ-10 | FR-53, FR-73 |
| Policy simulator | Run golden cases and a synthetic population against a candidate pack; diff against the current runtime bundle | policy_author, policy_approver (run) | UJ-2, UJ-6 | FR-40 |
| Policy approval | Submit (pack state IN_LEGAL_REVIEW), approve or request changes (maker != checker), activation by release | policy_author W, policy_approver W, policy_release W; `sharia_approver` W for Sharia approval only (action A14, ISLAMIC deployments only, unavailable while DEC-63 names no one; SOD-9) | UJ-6, UJ-10 | FR-35, FR-36, FR-73 |
| Model registry | Registry entries: RULESET (DRAFT / ACTIVE / RETIRED) and ML_MODEL (DRAFT / SHADOW / APPROVED / RETIRED; SHADOW shows aggregate metrics only; APPROVED unavailable while DEC-19 is open) | model_admin W, analyst and auditor R | - | FR-16 |
| Assistant panel | Read-only Q&A over tool results | analyst, relationship_staff W | UJ-4 | FR-50 |
| Portfolio view (Could) | Aggregate segment and churn distributions only, from `GET /v1/portfolio/summary`; small cells suppressed and marked; counts shown rounded ("about"); one fixed dimension at a time, no free filters; never tokens or row-level data; Track A may combine both cells, never tracks (planning/21 §10A) | viewer, analyst R (`ri.portfolio.read`, DEC-62 closes G-RS-1) | UJ-9 | FR-51, FR-80 |

Surface closure: every PRD journey (UJ-1..UJ-6, UJ-8, UJ-9, UJ-10) lands on a screen above. UJ-7 is plugin-only and has no UI. The name "reveal" never appears in this UI; it is a host function only.

## Voice and Tone
- Plain language, from fixed templates in the MVP (DEC-06). Every number shown must match the tool result exactly. *Source: planning/08 SEC-14.*
- Never state a legal conclusion. Policy text cites policy IDs and statuses only, for example "Blocked: policy POL-US-NY-NYC-0007 is PENDING_LEGAL_REVIEW". It never says "permitted by law X". *Source: planning/14 §10.*
- Never imply credit approval. Credit appears only as "Refer to the credit platform for assessment". *Source: planning/14 §13.1.*
- Protected customers: "Protected customer - no commercial profiling". *Source: planning/09 §5 step 10.*
- Products: always "SYNTHETIC - DEMO, not real terms". *Source: BQ-9.*
- Refusals (assistant): a short refusal plus the handoff template. *Source: planning/15 §3.3.*
- Customer-facing explanation languages are decided per approved policy (English/Spanish candidates, CMP-18). The staff dashboard is en-US in `A-US`, en-GB in `A-UK`, and Arabic + English in Track B (DEC-59).
- **Track B wording:** only approved terms from the Sharia term list (`CTL-TERMINOLOGY`), e.g. "finance" and "profit rate", never "loan" or "interest"; prohibited spending categories are labelled neutrally, never judged; nothing implies the customer's religion. Products also read "SYNTHETIC - DEMO, not real terms". The UI never states that something is Sharia-compliant on its own authority; it shows the recorded Sharia approval state. *Source: planning/21 §4; DEC-54, DEC-55.*

## Component Patterns
These components are behavioural; visual specs are pending (DESIGN.md).

| Component | Behaviour |
|---|---|
| Decision badge | Shows the outcome (PERMIT / PERMIT_WITH_OBLIGATIONS / DENY), `decision_id` and the controls applied. Opens the gate trace |
| Gate trace panel | Lists each gate (consent, jurisdiction PDP, minor protection, suppression, channel, human review) with its result and reason code; suppressed candidates stay visible with their reason |
| Pending-legal-review banner | Persistent while any active pack or policy is PENDING_LEGAL_REVIEW (in the MVP, always) |
| Illustrative watermark banner | Persistent on every screen while the policy bundle is `ILLUSTRATIVE` (standalone demo). Text: "illustrative - not legal advice - not for real customers". In Track B it adds "illustrative - not a Sharia ruling". Also printed on exports and explanations (DEC-44, DEC-55) |
| Track and cell indicator | Persistent in the app shell: track (A Conventional / B Islamic), `institution_type` and `cell_id` from `GET /v1/capabilities` `deployment_profile`; read-only, no switcher (a different track is a different deployment and URL) (DEC-61) |
| Sharia approval chip (Track B) | Next to `pack_state` and on catalogue items and templates: NOT_REVIEWED / IN_SHARIA_REVIEW / SHARIA_APPROVED / SHARIA_REJECTED; a blocked suggestion shows reason `SHARIA_REVIEW_PENDING` (DEC-55) |
| Locale switch (Track B) | Arabic / English; Arabic renders right-to-left with mirrored navigation and layout; numbers and codes stay verified (DEC-59) |
| Protected-customer notice | Replaces every blocked section for MINOR_PROTECTED / AGE_UNKNOWN; states which tier blocked it |
| Synthetic label | Shown on every dataset, product and persona surface |
| Version strip | Rule, model, config and pack versions plus `as_of` on each output |
| Evidence table | Shows a need or churn driver with the transactions or features behind it (last 6 or 12 months) |
| Review item row | State, SLA countdown, required role, actions (start review, clear, reject or escalate with mandatory reason) |
| Outcome recorder | Fixed outcome and reason pick-lists; no free text (BQ-7) |
| Jurisdiction selector | Limited to the DEC-16 test values; never lists unsupported states as selectable live markets |
| Simulator diff | Shows decisions changed by activity and jurisdiction between the candidate and active pack |

## State Patterns
| State | Trigger | UI behaviour |
|---|---|---|
| DENIED - MINOR_PROTECTED | `age_status` MINOR_PROTECTED or AGE_UNKNOWN | Protected-customer notice; no partial data in blocked sections |
| DENIED - UNKNOWN / UNSUPPORTED jurisdiction | Missing context, or `US-CA` | Restrictive default shown and labelled; insight only where consent is valid |
| BLOCKED - PENDING_LEGAL_REVIEW | Policy pending or past `review_due` | Output blocked, flag shown, legal-queue event visible in the audit log |
| DATA_QUALITY_INSUFFICIENT | DQ rule breach | Module section shows "insufficient data" with the rule ID; nothing is guessed |
| Consent withdrawn | Withdrawal event | Recommendations removed; recomputed state and audit entry shown |
| Awaiting review | `requires_human_review` | Item hidden from the customer view until CLEARED_TO_SHOW; EXPIRED items never shown |
| Template fallback | LLM disabled or validation failure | Template explanation; no error shown to the user |
| Not ready | Policy bundle unverified at start-up | Whole app shows not-ready; no customer data shown |
| Empty | No candidates after gating | "No suggestion" plus gate trace, so the reason is clear |

## Interaction Primitives
- Every output can be expanded to its evidence and decision.
- Destructive or governance actions (reject, escalate, approve pack, activate pack) need an explicit confirmation and a mandatory reason where the PRD requires one.
- The jurisdiction selector and simulator act only on synthetic data.
- No action in the UI contacts a customer or changes an account (planning/02 §3).

## Accessibility Floor
- Meets WCAG 2.2 AA (DEC-41, DECIDED).
- Full keyboard navigation, visible focus, and screen-reader labels for every status. *Source: planning/10 S10.1 AC.*
- Status is never conveyed by colour alone (see DESIGN.md Colors).
- Explanation templates are written in plain language (planning/05 §9).
- Track B: AA holds in both right-to-left (Arabic) and left-to-right (English) layouts; `lang` and `dir` are set per locale so screen readers switch voice; focus order follows the reading direction (DEC-59).

## Key Flows
Staff protagonists are roles in the pack; no names are invented. Alex is the pack's synthetic customer.

1. **UJ-1 Demo walk-through (manager).**
   1. Pick Alex and see jurisdiction `US-NY-NYC` and the runtime-bundle packs (ILLUSTRATIVE in the demo).
   2. Transactions (duplicate and reversal handled).
   3. Segments with reasons.
   4. Emergency-buffer need with evidence.
   5. Churn band, drivers and model version.
   6. **Climax:** two SYNTHETIC catalogue suggestions, each with a decision badge and gate trace, under the illustrative watermark (permitted only by the `ILLUSTRATIVE` demo bundle, DEC-44). The churn version shown is the rule baseline; the shadow ML model is never shown (DEC-47).
   7. Record CONTACTED on a human check-in.
   - *Source: planning/09 §5, §7.*
2. **UJ-2 Fail-safe (manager).**
   1. Switch to the `US-CA` customer and then the missing-jurisdiction customer: restrictive default is shown.
   2. In the simulator, remove the illustrative assumption from the relevant policy, so it is plain PENDING_LEGAL_REVIEW.
   3. **Climax:** the commercial suggestion is blocked and flagged, and a legal-queue event appears.
3. **UJ-3 Protected minor.** Open the 16-year-old. **Climax:** protected notice, with no Tier-1 or Tier-2 output.
4. **UJ-4 Assistant.** "Why this product?" gets a grounded template answer. "Approve a loan" gets a refusal and the handoff.
5. **UJ-5 Review (retention reviewer).**
   1. Open the queue and start a review.
   2. **Climax:** clear the item to show it, or reject it with a reason.
   3. The SLA lapse path leads to EXPIRED.
6. **UJ-6 Maker-checker.**
   1. The author drafts a change, runs the simulator and submits.
   2. The approver (a different user) reviews.
   3. **In the MVP the flow stops with the pack in `IN_LEGAL_REVIEW`, because no approver is named (DEC-11)**, and the UI shows this.
7. **UJ-8 Consent withdrawal.** Withdraw marketing consent; suggestions disappear and the audit entry is shown.
8. **UJ-9 Track A two cells.** After UJ-1 in `A-US`, open a London customer in `A-UK` (en-GB, GBP; the cell indicator changes); show that the London token is not found in `A-US`, that a NYC-resident London-booked customer is denied commercial activity, and the aggregate-only portfolio across both cells. *Source: planning/21 §11.*
9. **UJ-10 Track B (separate deployment).** Open the Track B URL; the indicator shows ISLAMIC; switch Arabic/English. **Climax:** Islamic-catalogue suggestions only, financing shown only as "Refer to the credit platform for assessment", a terminology fallback and a `SHARIA_REVIEW_PENDING` block, all under both watermarks. *Source: planning/21 §11.*

## Open Questions
- **UX-OQ-1:** Is there a bank brand or design system to inherit? No source in the pack. Owner: MGR.
- **UX-OQ-2:** Colour palette and typography. These need a decision before Epic 9 dashboard stories start (no source; do not invent).
- **UX-OQ-3:** UI component system (e.g. an internal system or an open-source React library). Ratify with the stack at GATE-2 (DEC-05).
- **UX-OQ-4:** Key-screen mocks were not rendered. The bmad-ux finalize step offers them, but they were skipped because UX-OQ-1..3 are open. Candidate screens: customer view, review queue, simulator diff.
- **UX-OQ-5:** Staff persona names for journeys are not in the pack. Role names are used instead.
- **UX-OQ-6:** Customer-facing explanation languages (CMP-18) are deferred; there is no customer UI in the MVP. Staff locales are decided (DEC-59).
- **UX-OQ-7:** Arabic typography and an RTL-capable component system are part of UX-OQ-2/3; translation reviewer and Sharia terminology reviewer for Track B templates are bank roles (no source; do not invent).

## Change log
- 2026-11-09 (readiness fixes): illustrative watermark banner component (DEC-44); UJ-1 climax under the watermark with the rule-baseline churn version (DEC-47); UJ-2 step 2 removes the illustrative assumption; UJ-6 ends at `IN_LEGAL_REVIEW` (DEC-45).
- 2026-11-10 (PH-3 role-scope matrix): Auth points to `contracts/role-scope-matrix.*`; auditor recommendation section denied (C-RS-1); `retention_reviewer` replaced by `relationship_staff` (N-RS-1); picker, transactions panel and portfolio gaps noted (G-RS-1..3).
- 2026-11-10 (two-track scope, planning/21): picker reads tokenised `GET /v1/customers`; portfolio view enabled as aggregate-only (`ri.portfolio.read`, DEC-62); voice rules for Track B terminology and Sharia approval; components track/cell indicator, Sharia approval chip, Track B locale switch, Sharia watermark; RTL accessibility; flows UJ-9, UJ-10; UX-OQ-7.
- 2026-11-10 (readiness re-run fixes): Customer view row uses the DEC-62 read paths (auditor denied `ri.transactions.read`); `sharia_approver` added to Policy pack viewer and Policy approval (ISLAMIC only, A14); surface closure includes UJ-9 and UJ-10.
- 2026-11-10 (independent two-track validation): Portfolio view row follows aggregate disclosure control (planning/21 §10A): rounded counts, one fixed dimension, no free filters.
- 2026-11-10 (contract 1.2.0, V-8 and V-9): picker uses the purpose-bound customer directory (access purpose, token or narrowing filter, bounded pages; auditor removed from the picker, keeps the customer view by known token); new row for the Track B Sharia disclosure on suggestion cards.
