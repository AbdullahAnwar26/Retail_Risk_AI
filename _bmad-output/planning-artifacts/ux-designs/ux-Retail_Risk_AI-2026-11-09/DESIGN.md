---
name: Retail Intelligence Staff Dashboard
description: Internal staff dashboard for the standalone demo of the Retail Banking AI subsystem. The visual identity is not yet decided; this spine records the constraints and leaves the tokens open.
status: final
created: 2026-11-09
updated: 2026-11-09
sources: [planning/05 §9, planning/09 §1 §5 §7, planning/10 E10, planning/12 §H.1 DEC-05 DEC-41]
colors: {}
typography: {}
rounded: {}
spacing: {}
components: {}
---

# DESIGN.md - Retail Intelligence Staff Dashboard

> **Status of visual identity: OPEN.** The planning pack defines no brand, palette, typography or UI system. Following the bmad-ux rule "capture, do not author", the token maps above are left empty on purpose, not invented. See open items UX-OQ-1..UX-OQ-4 in `EXPERIENCE.md` §Open Questions. The behavioural contract (`EXPERIENCE.md`) is complete enough to build against. The visual identity must be decided before dashboard stories start (Epic 9 in `epics.md`).

## Brand & Style

This is an internal, staff-only tool for a regulated bank. It must read as sober and evidential: every number has a source, every decision has an ID. It is not marketing. Every product shown is SYNTHETIC, and the UI must never look like a live bank offer. *Source: planning/09 §1 (synthetic catalogue, BQ-9); planning/02 §3.*
Brand assets and the bank's own design system are unknown (UX-OQ-1).

## Colors

No palette is decided (UX-OQ-2). These constraints bind any future palette:
- Text and UI contrast meet WCAG 2.2 AA (DEC-41, DECIDED; planning/05 §9 NFR).
- Colour is never the only signal for policy outcomes (PERMIT, DENY, PENDING_LEGAL_REVIEW, protected customer). A text label or icon is always present, as required by EXPERIENCE.md Accessibility Floor.
- Status colours must not imply approval of credit. There is no "approved" green for any customer outcome, in line with planning/14 §13.1.

## Typography

Not decided (UX-OQ-2). Constraint: numbers (amounts, scores, versions, IDs) must use a face that keeps digits distinguishable. Tabular figures are preferred so evidence tables align.

Constraint (Track B, UX-OQ-7; DEC-59): the chosen typeface and component system must support Arabic script and right-to-left layout (mirrored navigation and alignment, bidirectional text in evidence tables, a documented choice between Arabic-Indic and Western digits per locale, digits still distinguishable). No tokens are set here; they follow UX-OQ-2/3.

## Components

Visual specs are pending a UI-system choice (UX-OQ-3). The behavioural component inventory lives in `EXPERIENCE.md` §Component Patterns, and visual specs will be added per component once tokens exist.

## Do's and Don'ts

- **Do** show the `SYNTHETIC` / "DEMO - not real terms" label on every product and dataset surface (BQ-9).
- **Do** show `PENDING_LEGAL_REVIEW` openly and prominently (DEC-15).
- **Don't** use approval or celebration styling (ticks, "approved" badges) on recommendations, review items or credit referrals.
- **Don't** display customer names. The subsystem holds tokens only, and "reveal" is a host function (planning/15 §5.1).

## Change log
- 2026-11-10 (readiness re-run fixes): Arabic/RTL typography constraint added under Typography (UX-OQ-7); no tokens.
